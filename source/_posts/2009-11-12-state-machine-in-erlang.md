---
layout:     post
title:      "State Machine in Erlang"
date:       2009-11-12 10:43
categories: [Erlang]
tags:       [erlang]
published:  true
---

There is some sort of confusion in the object-oriented community about functional languages. How is it possible to implement stateful application if the language has no concept of state? It turns out that it's actually quite possible, although the solution is completely deferent from what we see in the OO realm. In Erlang, for example, state can be implemented by using *process message passing* and *tail recursion*. This approach is so elegant that after you've learned it, the OO way of doing this looks unnatural. The code below is the Erlang implementation of Uncle Bob's [FSM example][1]. Look at it. Isn't that code clean and expressive? It looks almost like DSL but it's actually regular Erlang syntax.

{% codeblock lang:erlang %}
-module(turnstile).
-export([start/0]).
-export([coin/0, pass/0]).
-export([init/0]).

start() -> register(turnstile, spawn(?MODULE, init, [])).

% Initial state

init() -> locked().

% Events

coin() -> turnstile ! coin.
pass() -> turnstile ! pass.

% States and transitions

locked() ->
    receive
        pass ->
            alarm(),
            locked();
        coin ->
            unlock(),
            unlocked()
    end.

unlocked() ->
    receive
        pass ->
            lock(),
            locked();
        coin ->
            thankyou(),
            unlocked()
    end.

% Actions

alarm() -> io:format("You shall not pass!~n").
unlock() -> io:format("Unlocking...~n").
lock() -> io:format("Locking...~n").
thankyou() -> io:format("Thank you for donation~n").
{% endcodeblock %}

The idea behind this code is simple. Every state is implemented as a function that does two things: it listens for messages sent by other processes; when message is received the appropriate action is taken and one of the *state-functions called recursively*. Simple.


[1]: http://www.objectmentor.com/resources/articles/umlfsm.pdf
