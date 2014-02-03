---
layout:     post
title:      "Counting modifications in Git repository"
date:       2011-01-29 12:00:00
categories: [Git,Groovy]
tags:       [git,groovy]
published:  true
---

Michael Feathers wrote a [blog][1] about Open-Closed Principle, where he described simple technique that measures the closure of the code. I created a Groovy [script][2] which implements this algorithm for Git repositories. If you run it from the root of your Git project, it produces a CSV file with the statistics of how many times the files have been modified.

As an example, here is the top 10 files from [rabbitmq-server][3] repository

    845  src/rabbit_amqqueue_process.erl
    711  src/rabbit_channel.erl
    650  src/rabbit_tests.erl
    588  src/rabbit_variable_queue.erl
    457  src/rabbit_amqqueue.erl
    448  src/rabbit_mnesia.erl
    405  src/rabbit.erl
    395  src/rabbit_reader.erl
    360  src/rabbit_msg_store.erl
    356  src/rabbit_exchange.erl


[1]: http://michaelfeathers.typepad.com/michael_feathers_blog/2011/01/measuring-the-closure-of-code.html
[2]: https://github.com/ndpar/utilities/blob/master/git-files-modified.groovy
[3]: https://github.com/rabbitmq/rabbitmq-server
