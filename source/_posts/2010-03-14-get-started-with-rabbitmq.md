---
layout:     post
title:      "Get started with RabbitMQ"
date:       2010-03-14 12:00:00
categories: [Groovy,RabbitMQ]
tags:       [groovy,rabbitmq]
published:  true
---

[RabbitMQ][1] is an open-source implementation of [AMQP][2]. If you don't know what AMQP is, I encourage you to read about it on the official web site or the reference [page][3]. What *I* personally like in RabbitMQ/AMQP is

- AMQP is a free alternative to expensive TIBCO Randezvous.
- Functionality-wise AMQP is a superset of JMS.
- RabbitMQ is written in Erlang, which means fault-tolerance and reliability.

In this short blog entry I show how to install RabbitMQ on a \*nix box, and verify that it's working with a simple Groovy client.

<!-- more -->

## Installing RabbitMQ server

Before installing RabbitMQ make sure you have Erlang environment set up. Run the following command and verify the output

    $ erl -version
    Erlang (ASYNC_THREADS,HIPE) (BEAM) emulator version 5.10.1

I used to install RabbitMQ using apt-get command, which is convenient but always several versions behind. That's why I switched to [binary distribution][4] installation. So here are the installation steps

    $ sudo mkdir /opt/rabbitmq
    $ sudo chown -R rabbitmq /opt/rabbitmq
    $ cd /opt/rabbitmq
    $ wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.1.1/rabbitmq-server-generic-unix-3.1.1.tar.gz
    $ tar xf rabbitmq-server-generic-unix-3.1.1.tar.gz
    $ rm rabbitmq-server-generic-unix-3.1.1.tar.gz
    $ ln -s rabbitmq_server-3.1.1 rabbitmq

Append the following lines to *~/.bashrc*

{% codeblock ~/.bashrc lang:bash %}
RABBITMQ_HOME=/opt/rabbitmq/rabbitmq
PATH=$PATH:$RABBITMQ_HOME/sbin
{% endcodeblock %}

Enable web admin plugin

    $ rabbitmq-plugins enable rabbitmq_management

Run the following command to verify server is starting

    $ rabbitmq-server

                  RabbitMQ 3.1.1. Copyright (C) 2007-2013 VMware, Inc.
      ##  ##      Licensed under the MPL.  See http://www.rabbitmq.com/
      ##  ##
      ##########  Logs: /opt/rabbitmq/rabbitmq/sbin/../var/log/rabbitmq/rabbit@beta.log
      ######  ##        /opt/rabbitmq/rabbitmq/sbin/../var/log/rabbitmq/rabbit@beta-sasl.log
      ##########
                  Starting broker... completed with 6 plugins.

Go to [localhost:15672](http://localhost:15672) in the browser. You should be able to login as guest/guest.

## Groovy client

Groovy client is essentially Java client written in Groovy language. Here are the scripts for consumer and publisher in Groovy

{% codeblock consumer.groovy lang:groovy %}
#!/usr/bin/env groovy

import com.rabbitmq.client.*

@Grab(group='com.rabbitmq', module='amqp-client', version='3.1.0')
factory = new ConnectionFactory([
    username: 'guest',
    password: 'guest',
    virtualHost: '/',
    requestedHeartbeat: 0
])
conn = factory.newConnection(new Address('localhost', 5672))
channel = conn.createChannel()

queueName = 'myQueue'

channel.queueDeclare queueName, false, true, true, [:]
channel.queueBind queueName, 'amq.fanout', 'myRoutingKey'

def consumer = new QueueingConsumer(channel)
channel.basicConsume queueName, false, consumer

while (true) {
    delivery = consumer.nextDelivery()
    println "Received message: ${new String(delivery.body)}"
    channel.basicAck delivery.envelope.deliveryTag, false
}
channel.close()
conn.close()
{% endcodeblock %}


{% codeblock publisher.groovy lang:groovy %}
#!/usr/bin/env groovy

import com.rabbitmq.client.*
 
@Grab(group='com.rabbitmq', module='amqp-client', version='3.1.0')
factory = new ConnectionFactory([
    username: 'guest',
    password: 'guest',
    virtualHost: '/',
    requestedHeartbeat: 0
])
conn = factory.newConnection(new Address('localhost', 5672))
channel = conn.createChannel()
 
channel.basicPublish 'amq.fanout', 'myRoutingKey', null, "Hello, world!".bytes
 
channel.close()
conn.close()
{% endcodeblock %}

Run consumer script in one terminal window

    $ chmod 755 consumer.groovy
    $ ./consumer.groovy

and publisher script in another

    $ chmod 755 publisher.groovy
    $ ./publisher.groovy

On the consumer window you should see *Received message: Hello, world!* text.

Now you just need to stop RabbitMQ server, and restart it in detached mode

    $ sudo -u rabbitmq rabbitmq-server -detached


[1]: http://www.rabbitmq.com
[2]: http://en.wikipedia.org/wiki/AMQP
[3]: http://www.rabbitmq.com/how.html
[4]: http://www.rabbitmq.com/install-generic-unix.html
