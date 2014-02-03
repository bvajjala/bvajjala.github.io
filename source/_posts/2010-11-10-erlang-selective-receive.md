---
layout:     post
title:      "Erlang explained: Selective receive"
date:       2010-11-10 12:00:00
categories: [Erlang]
tags:       [erlang]
published:  true
---

If you worked with Erlang you've probably heard about selective receive. In this article I want to demonstrate what selective receive is and how it works. Let me start with an excerpt from Joe Armstrong's book [Programming Erlang][1] ([Section 8.6][2], p.155):

{% codeblock lang:erlang %}
receive
    Pattern1 [when Guard1] -> Expressions1;
    Pattern2 [when Guard2] -> Expressions2;
    ...
after
    Time -> ExpressionTimeout
end
{% endcodeblock %}

1. When we enter a **receive** statement, we start a timer (but only if an **after** section is present in the expression).
1. Take the first message in the mailbox and try to match it against `Pattern1`, `Pattern2`, and so on. If the match succeeds, the message is removed from the mailbox, and the expressions following the pattern are evaluated.
1. If none of the patterns in the **receive** statement matches the first message in the mailbox, then the first message is removed from the mailbox and put into a "save queue." The second message in the mailbox is then tried. This procedure is repeated until a matching message is found or until all the messages in the mailbox have been examined.
1. If none of the messages in the mailbox matches, then the process is suspended and will be rescheduled for execution the next time a new message is put in the mailbox. Note that when a new message arrives, the messages in the save queue are not rematched; only the new message is matched.
1. As soon as a message has been matched, then all messages that have been put into the save queue are reentered into the mailbox in the order in which they arrived at the process. If a timer was set, it is cleared.
1. If the timer elapses when we are waiting for a message, then evaluate the expressions `ExpressionsTimeout` and put any saved messages back into the mailbox in the order in which they arrived at the process.

<!-- more -->

Did you notice the concept of "save queue"? That's what many people are not aware of. Let's run few tests and see the mailbox and the save queue in action.

The first test is simple

{% codeblock lang:erlang %}
1> self() ! a.
a

2> process_info(self()).
 ...
 {message_queue_len,1},
 {messages,[a]},
 ...

3> receive a -> 1; b -> 2 end.
1

4> process_info(self()).
 ...
 {message_queue_len,0},
 {messages,[]},
 ...
{% endcodeblock %}

We send a message to the shell, we see it in the process mailbox, then we receive it by matching, after which the queue is empty. Standard queue behaviour.

For the next test we send three messages, and we match the last one

{% codeblock lang:erlang %}
1> self() ! c, self() ! d, self() ! a.
a

2> process_info(self()).
 ...
 {message_queue_len,3},
 {messages,[c,d,a]},
 ...

3> receive a -> 1; b -> 2 end.
1

4> process_info(self()).
 ...
 {message_queue_len,2},
 {messages,[c,d]},
 ...
{% endcodeblock %}

Again, no surprises. In fact, this test demonstrates what people think when they hear about selective receive.

Unfortunately we didn't see what happened internally between expressions 3 and 4. To find it out we need to modify the test. Let's start the shell in distributed mode so that we can connect to it later from the remote shell.

{% codeblock lang:erlang %}
(foo@bar)1> register(shell, self()).
true

(foo@bar)2> shell ! c, shell ! d.
d

(foo@bar)3> process_info(whereis(shell)).
 ...
 {message_queue_len,2},
 {messages,[c,d]},
 ...

(foo@bar)4> receive a -> 1; b -> 2 end.
{% endcodeblock %}

At this moment the shell is suspended, exactly as described in step 4 of selective receive algorithm. Open remote shell, connect to the initial one, and type the following:

{% codeblock lang:erlang %}
(foo@bar)1> process_info(whereis(shell)).
 ...
 {message_queue_len,0},
 {messages,[]},
 ...
{% endcodeblock %}

That's interesting: no messages in the mailbox! As Joe said, they are in the save queue. Now send a matching message:

{% codeblock lang:erlang %}
(foo@bar)2> shell ! a.
a
{% endcodeblock %}

Go back to initial shell, which should be resumed now, and check the mailbox again:

{% codeblock lang:erlang %}
1
(foo@bar)5> process_info(whereis(shell)).
 ...
 {message_queue_len,2},
 {messages,[c,d]},
 ...
{% endcodeblock %}

That's the same output we saw in the previous test, but now we know what happens behind the scene: *messages are moved from the mailbox to the save queue and then back to the mailbox after the matching message arrives*.

Next time you explore your Erlang process, keep in mind the save queue and disappearing and reappearing messages.

### Resources

- A [screencast](http://ascii.io/a/3477) recorded for this blog entry.
- Joe Armstrong: [A History of Erlang](http://www.search-document.com/pdf/3/3/erlang.html). In this article you can find why the selective receive was implemented this way.

[1]: http://www.pragprog.com/titles/jaerlang/programming-erlang
[2]: http://media.pragprog.com/titles/jaerlang/Concurrent.pdf
