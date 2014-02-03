---
layout:     post
title:      "Integrating RabbitMQ with ejabberd"
date:       2010-03-31 12:00:00
categories: [Ejabberd,RabbitMQ]
tags:       [ejabberd,erlang,rabbitmq,xmpp]
published:  true
---

ejabberd is integrated with RabbitMQ by means of [mod_rabbitmq][1] gateway. If you followed my ejabberd installation [instructions][2], you should already have mod_rabbitmq installed.

## Configure mod_rabbitmq

Open */opt/ejabberd/ejabberd/etc/ejabberd/ejabberd.cfg* file, find *modules* section, and add mod_rabbitmq stanza to the list

{% codeblock /opt/ejabberd/ejabberd/etc/ejabberd/ejabberd.cfg lang:erlang %}
{modules,
 [
  ...
  {mod_rabbitmq,  [{rabbitmq_node, RABBIT_NODE}]},
  ...
 ]}.
{% endcodeblock %}

You need to replace *RABBIT_NODE* with the real value, which you can find from the RabbitMQ process information

    $ ps -ef | grep beam
    rabbitmq  8525  8142  0 16:37 pts/0    ... -sname rabbit@ubuntu ...

You can see my RabbitMQ node name is *rabbit@ubuntu*.

## Set up cookie

To make RabbitMQ and ejabberd work together, they have to run in the same Erlang cluster. That means they have to use the same cookie file. If you [installed][3] RabbitMQ from binary distribution, it uses the user's cookie *~/.erlang.cookie*. ejabberd, on the other hand, uses its own cookie. Let's replace it with the user's

    $ cd /opt/ejabberd/ejabberd/var/lib/ejabberd
    $ rm -f .erlang.cookie
    $ ln -s ~/.erlang.cookie

Restart ejabberd server

    $ /opt/ejabberd/ejabberd/sbin/ejabberdctl restart

## Add rabbit buddy to roster

The rabbit's JID consists of two parts: exchange name and routing domain. The routing domain is a string `rabbitmq.EJABBERD_HOST` where *EJABBERD_HOST* is the host you registered in *ejabberd.cfg*. In my case the routing domain is `rabbitmq.jabber.ndpar.com`.

For the name you can use any exchange name available in the RabbitMQ server. I use `amq.fanout` exchange which exists in every RabbitMQ server. So I go to my IM client (Adium) and add this user to the buddies list `amq.fanout@rabbitmq.jabber.ndpar.com`.

## Rabbit's greetings

To publish a message to RabbitMQ I use the same Groovy script as in the previous [post][3]

    $ ./publisher.groovy

{% img center /images/posts/mod_rabbitmq.png %}

### Resources

- Tony Garnock-Jones' [presentation slides][4] about RabbitMQ and its extensions.


[1]: http://tonyg.github.io/rabbitmq-xmpp/
[2]: /2010/02/23/get-started-with-ejabberd/
[3]: /2010/03/14/get-started-with-rabbitmq/
[4]: http://www.erlang-factory.com/upload/presentations/229/ErlangFactorySFBay2010-TonyGarnock-Jones.pdf
