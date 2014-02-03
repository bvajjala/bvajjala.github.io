---
layout:     post
title:      "Sleeping Barber in Erlang"
date:       2013-06-11 20:43
categories: [Erlang]
tags:       [erlang,gen_fsm]
published:  true
---

After working on [*Sleeping Barber*][1] problem at [Coding Dojo][2] I decided to implement it in Erlang. Because Erlang is the perfect language for these sorts of problems, many people have alredy solved it in many different ways. I found at least three groups of solutions: 1) direct implementation using message passing between processes, 2) OTP solution using *gen_server*, and 3) something resembling object-oriented approach.

Strangely enough, I didn't see any solution based on [FSM][3]. That is odd because this is the first thing coming to my mind when I hear this problem. To fill the gap, I'm going to solve it using *gen_fsm*, which is a standard OTP behaviour for FSM.

<!-- more -->

## Overview of gen_fsm

In essence gen_fsm behaviour is built the same way as a [naïve][4] implementation of FSM. It just hides the low-level code that deals with messages and provides integration with OTP framework. An implementation of gen_fsm behaviour consists of three parts

- A *client API* that calls gen_fsm public interface
- gen_fsm *callbacks* required by OTP
- State *transition* functions

We'll see examples of all of them in a minute, but first

## Dialogue

*Barber shop. The Barber is sleeping in the chair. Customer I comes in.*

*Customer I.* Good morning.<br/>
*Barber.* Good morning. Please take a seat.

*Barber starts shaving Customer I. Customer II comes in.*

*Customer II.* Good morning.<br/>
*Barber.* I'm busy. You have to wait.<br/>
*Customer II.* OK.

*Customer II goes to waiting room. The same happens with Customers III and IV. Customer V comes in.*

*Customer V.* Good morning.<br/>
*Barber.* Sorry Customer V. Not today.<br/>
*Customer V.* Maybe tomorrow.

*Customer V exits the shop. Meantime Barber finished Customer I's haircut.*

*Barber.* Do you like your haircut?<br/>
*Customer I.* Thank you.<br/>
*Barber (looking at Customer II).* Next please.

*The same happens with Customer II, III and IV. After that Barber takes the chair and sleeps.*

## Modules

From this dialogue we can see we just need to implement two modules: *barber* and *customer*.
I'm not going to show all the functions here in this article, only those that are interesting from gen_fsm perspective.

Let's start with *customer* because it's simpler.

{% codeblock customer.erl lang:erlang %}
-module(customer).
-behaviour(gen_fsm).

%% API
-export([start/0, sit_down/1, done/1, wait/1, sorry/1]).

%% Callback functions
-export([init/1, handle_event/3, handle_sync_event/4, handle_info/3,
         terminate/3, code_change/4]).

%% FSM states
-export([waiting/2, served/2]).

%%====================================================================
%% API
%%====================================================================

start() ->
    {ok,Pid} = gen_fsm:start(?MODULE, [], []),
    Pid.

sit_down(Customer) ->
    gen_fsm:send_event(Customer, sit_down).

done(Customer) ->
    gen_fsm:send_event(Customer, exit).

wait(Customer) ->
    gen_fsm:send_event(Customer, wait).

sorry(Customer) ->
    gen_fsm:send_event(Customer, exit).

%%====================================================================
%% gen_fsm callbacks (empty functions are omitted)
%%====================================================================

init([]) ->
    log("Good morning."),
    {ok, waiting, unow()}.

%%====================================================================
%% FSM states and transitions
%%====================================================================

waiting(sit_down, WaitStart) ->
    log("I've been waiting for ~p sec.", [duration(WaitStart)]),
    {next_state, served, unow()};

waiting(wait, StateData) ->
    log("OK."),
    {next_state, waiting, StateData};

waiting(exit, _StateData) ->
    log("Maybe tomorrow."),
    {stop, normal, undefined}.

served(exit, ServiceStart) ->
    log("Thank you. The haircut took ~p sec.", [duration(ServiceStart)]),
    {stop, normal, undefined}.
{% endcodeblock %}

*barber* module is little bit more complicated, because it not only calls customer's API, it also sends events to itself.

{% codeblock barber.erl lang:erlang %}
-module(barber).
-behaviour(gen_fsm).

%% API
-export([start_link/0, new_customer/1]).

%% Callback functions
-export([init/1, handle_event/3, handle_sync_event/4, handle_info/3,
         terminate/3, code_change/4]).

%% FSM states
-export([busy/2, sleep/2]).

-define(SERVER, ?MODULE).
-define(HAIRCUT_TIME, 5000).
-define(ROOM_SIZE, 3).

-record(state, {chair, room = []}).

%%====================================================================
%% API
%%====================================================================

start_link() ->
    gen_fsm:start_link({local, ?SERVER}, ?MODULE, [], []).

new_customer(Customer) ->
    gen_fsm:send_event(?SERVER, {new, Customer}).

%%====================================================================
%% gen_fsm callbacks (empty functions are omitted)
%%====================================================================

init([]) ->
    log("Shop is open. zzzzZ"),
    {ok, sleep, #state{}}.

handle_info(finish, busy, #state{chair = Customer, room = Room}) ->
    log("Do you like your haircut ~p?", [Customer]),
    customer:done(Customer),
    case Room of
        [C | Rest] ->
            log("Next please ~p.", [C]),
            serving(C),
            NewStateData = #state{chair = C, room = Rest},
            {next_state, busy, NewStateData};
        [] ->
            log("Time for nap. zzzzZ."),
            {next_state, sleep, #state{}}
    end.

serving(Customer) ->
    customer:sit_down(Customer),
    timer:send_after(?HAIRCUT_TIME, finish).

%%====================================================================
%% FSM states and transitions
%%====================================================================

sleep({new, Customer}, StateData) ->
    log("Good morning ~p. Please take a seat.", [Customer]),
    serving(Customer),
    NewStateData = StateData#state{chair = Customer},
    {next_state, busy, NewStateData}.

busy({new, Customer}, #state{room = Room} = StateData)
  when length(Room) == ?ROOM_SIZE ->
    log("Sorry ~p. Not today.", [Customer]),
    customer:sorry(Customer),
    {next_state, busy, StateData};

busy({new, Customer}, #state{room = Room} = StateData) ->
    log("I'm busy. You have to wait."),
    customer:wait(Customer),
    NewStateData = StateData#state{room = Room ++ [Customer]},
    {next_state, busy, NewStateData}.
{% endcodeblock %}


To reproduce the dialogue above we evaluate the following expressions in the REPL


{% codeblock lang:erlang %}
barber:start_link().
barber:new_customer(customer:start()).
barber:new_customer(customer:start()).
barber:new_customer(customer:start()).
barber:new_customer(customer:start()).
barber:new_customer(customer:start()).
{% endcodeblock %}


### Resources

- [Source code][5] for this article.
- [Screencast][6] reproducing the dialogue.
- [gen_fsm][7] documentation.


[1]: http://en.wikipedia.org/wiki/Sleeping_barber_problem
[2]: http://www.meetup.com/Toronto-Coding-Dojo/events/120577762/
[3]: https://en.wikipedia.org/wiki/Finite-state_machine
[4]: /2009/11/12/state-machine-in-erlang/
[5]: https://github.com/ndpar/erlang/tree/master/barber
[6]: http://ascii.io/a/3613
[7]: http://www.erlang.org/doc/man/gen_fsm.html
