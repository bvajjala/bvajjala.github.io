---
layout:     post
title:      "Book review: Erlang and OTP in Action"
date:       2010-11-06 12:00:00
categories: [Book Review,Erlang]
tags:       [book review,erlang]
published:  true
---

{% img left /images/posts//otp-action-book-cover.png %}

Title: Erlang and OTP in Action<br />
Author: Martin Logan, Eric Merritt, and Richard Carlsson<br />
Paperback: 432 pages<br />
Publisher: [Manning Publications](http://www.manning.com/logan/); November 2010<br />
Language: English<br />
ISBN-10: 1933988789<br />
ISBN-13: 978-1933988788<br />
$38.99 ([amazon.com](http://www.amazon.com/Erlang-OTP-Action-Martin-Logan/dp/1933988789))

### Overview

Even though this book has Erlang in its title, it's only about 15% of the content dedicated to Erlang language itself — the biggest portion of the book is about OTP. Nowadays, when more and more developers get familiar with Erlang, they need a new book that can boost them to the next level of proficiency, where they can produce industry standard code leveraging all the power of Erlang platform. This book is supposed to fill this gap!

<!-- more -->

## Part One — The OTP basics

### Chapter 1 — The Erlang/OTP platform

This chapter gives an overview of the important *concepts and features* of Erlang/OTP: concurrency, fault-tolerance, distribution. It discusses four inter-process communication paradigms — shared memory, STM, futures, message passing — and shows how the latter makes the distribution trivial to implement in Erlang. You will see how the linked processes and supervision trees build the foundation of Erlang famous fault-tolerance, and how three aspects of Erlang runtime system — sophisticated scheduler, non-blocking IO, and per-process garbage collection — complete the picture.

### Chapter 2 — Erlang language essentials

Take a deep breath — this long chapter is going to be an Erlang Crash Course. If you already worked with the language, most of it won't be new for you, but it's still worthy to read it because there are many small things that you are probably not aware of or don't use very often. For example,

- are you familiar with all available shell functions, and break menu options?
- do you know how to work with multiple shells in one window?
- how lists are implemented internally, and how to use ++ operator efficiently?
- what's the difference between arithmetic and exact equality operators?
- do you know that all operators are actually functions, and `1+2` is the same as `erlang:'+'(1,2)`?
- that assignment operator is a form of pattern matching?
- that you can use pattern matching instead of regex: `"http://" ++ Rest = "http://www.erlang.org"`?
- what's the difference between case- and if-expressions, and between pattern matching and guards?
- that besides list comprehensions there are also bitstring comprehensions: `<< <<X:3>> || X <- [1,2,3,4,5,6,7] >>`?
- which steps Erlang preprocessor performs?
- what's the difference between linked and monitored processes?
- what's the relationship between messages and signals?

There is also nice introduction to algorithms in this chapter with excellent examples of how to use tail-recursion and accumulators to improve performance.

Some important topics are covered briefly, like selective receive mechanism, for example. But at the end of the chapter authors give a list of useful Erlang resources, including books and web sites, so you should be able to find there the answers to all your language related questions.

### Chapter 3 — Writing a TCP based RPC service

This chapter is about OTP *behaviours*. It describes what behaviour is, what are the benefits of it comparing to pure Erlang implementation, and which parts the behaviour consists of. As a 'Hello, World' example authors implemented TCP server!

Over the course of the chapter you will learn how to model client-server communication using *gen_server* behaviour, and how to implement active socket connection using *get_tcp* module.

It also shows the industry conventions and best practices of how to implement and layout behaviour module. You can use this chapter as a reference every time you need to implement a behaviour.

Code snippet: [TCP server](http://github.com/erlware/Erlang-and-OTP-in-Action-Source/blob/master/chapter_03/tr_server.erl).

### Chapter 4 — OTP applications and supervision

An OTP *application* is what ties your modules into a single unit. A *supervisor* is what makes your application fault-tolerant. From this chapter you will learn how to implement both behaviours properly: how to layout application directory, how to structure application descriptor, how to write child specifications and restart strategies, and how to generate application documentation. As before, all examples are accompanied by standard conventions and best practices.

Sample code: [application directory layout](http://github.com/erlware/Erlang-and-OTP-in-Action-Source/tree/master/chapter_04/tcp_rpc/).

### Chapter 5 — Using the main graphical introspection tools

This chapter demonstrates how to use some of the Erlang graphical tools: appmon, webtool, pman, debugger, and table viewer. It's good to know that those tools exist, so when you encounter a problem in your code, you would be able to find the root cause and resolve it quickly.

{% img center /images/posts/otp-action-book-tools.png %}

## Part Two — Building a production system

In the second part of the book you are going to apply all the knowledge you obtained in the first part to build a real world production system: *distributed cache*.

### Chapter 6 — Implementing a caching system

How do you implement a cache? I guess there are many ways to do it, but I would never come up with the idea the authors of the book came up with. They use a *separate process to store each value*, and they map each key to its corresponding process. How cool is that! This way of thinking is possible only in Erlang.

During the implementation of process management you will learn a new strategy when the supervisor creates multiple child-processes in runtime based on preconfigured template. It's different from what you saw in Chapter 4 where single child process was created on the application startup. One interesting twist here is an inversion of control — the worker process will call the supervisor to start it.

The rest of the chapter is dedicated to ETS tables (which are used here to store the mapping). You will see how to create tables and how to perform CRUD operations.

### Chapter 7 — Logging and event handling

Logging is very important part of any system. In OTP there are two logging utilities: error_logger and SASL. *error_logger* is similar to log4x libraries in other languages. It provides basic functions (info, warning and error) that you can call from your code to print messages to the standard output. SASL is more sophisticated. It's an OTP application that logs life cycle events, including crash reports, from other applications. Both methods are thoroughly described in the first part of this chapter.

The second part explains how to implement *custom event handler*. An OTP event handler is just another behaviour that models observer pattern. You can use it for example to implement your own log appender which you can plug in to the error_logger.

The final section of the chapter provides a step-by-step guide of how to build a *custom event stream*, and how to integrate it with the cache application. This technique was totally new to me. I never worked with event handlers on such advanced level.

Code snippets: [event manager](http://github.com/erlware/Erlang-and-OTP-in-Action-Source/blob/master/chapter_07/simple_cache/src/sc_event.erl) and [event handler](http://github.com/erlware/Erlang-and-OTP-in-Action-Source/blob/master/chapter_07/simple_cache/src/sc_event_logger.erl).

### Chapter 8 — Distributed Erlang/OTP

Distribution is one of the famous features of Erlang. It's very easy to build distributed applications, and more important, it's such a fun to play with Erlang *clusters*.

This chapter will guide you through all the methods and techniques you need to know to make your application distributed. You will learn how to start Erlang nodes in different modes, how to combine them into the clusters, how to define topology and isolate clusters from each other, and how to send messages between nodes in the same cluster.

One of the cool things I learned from this chapter is a *remote shell*. It's very similar to SSH but more powerful. Unlike SSH, Erlang remote shell is not a session — it's a real shell of the remote node where you can start any application including graphical tools!

The second half of the chapter discusses the problem of *resource discovery*: What's the best way to add a new node to the cluster and synchronize its state with existing nodes? The authors come up with a simple and elegant algorithm. You will use this algorithm in the next chapter to build a distributed cache.

Code snippet: resource discovery [algorithm](http://github.com/erlware/Erlang-and-OTP-in-Action-Source/blob/master/chapter_08/resource_discovery.erl).

### Chapter 9 — Adding distribution to the cache with Mnesia

When you design a distributed system you have to make a choice which inter-node communication strategy you are going to use: *synchronous* or *asynchronous*. Chapter 9 starts with the comparison of these two approaches, their advantages and drawbacks.

The next step towards the distributed cache is obvious: making the cache storage distributed. As you remember from the chapter 6 the storage was implemented as ETS table. The easiest way to make it distributed is to replace it with Mnesia database. Why and how? You will find it in the next section of this chapter.

You will learn what *Mnesia* is, how to configure it properly, and how to manipulate the data. At the end you will meet beauty and the beast of read operations - query list comprehensions and match specifications. Equipped with all these knowledge you will easily replace ETS table with Mnesia, and make your cache distributed.

The most amazing part of the last section is an algorithm of *dynamic table replication*. You will definitely appreciate it after you learn it — it's the heart of true scalability.

Code snippet: working with [Mnesia](http://github.com/erlware/Erlang-and-OTP-in-Action-Source/blob/master/chapter_09/simple_cache/src/sc_store.erl).

### Chapter 10 — Packaging, services, deployment

At this moment you should be able to write non-trivial OTP applications. It's time to think now how to make your application easy to install and start. So far you have started it manually from the shell, and if your app had many dependencies, it was a tedious process. This chapter describes how to *automate* it.

In OTP a deployment unit is called *release*. In this chapter you will learn how to build it properly, i.e. how to create release metadata and configuration, resolve dependencies, and generate boot scripts. You will see different ways to start your application: locally in shell, as a daemon or in embedded mode.

After you release the application, you might want to share it with other people. That's what the next section is about. It shows you how to make a *package* — standard or customized, universal or OS-dependent — and how to install it on a different machine.

## Part Three — Integrating and refining

In the previous part you built a distributed cache in OTP, and you can use it now from any Erlang application. This is already a big achievement, and you must be proud of it, but you can make it even bigger if you expose this wonderful functionality to other platforms. Erlang is known for its robustness and scalability, and it would be very beneficial for non-Erlang clients as well to utilize these features.

### Chapter 11 — Text and REST (Communication via TCP and HTTP)

The first non-Erlang interface you are going to implement is TCP. If you remember, you already did it in Chapter 3 when you implemented *TCP server*. That server though had one significant limitation: it handled only one connection. The new implementation in this chapter is more efficient: it supports multiple concurrent connections.

The next interface is HTTP. Although it sounds similar to the previous one, the way you will implement it is totally different. You won't use standard gen_server behaviour. Instead, you will implement a *custom behaviour* which you are going to define yourself. This is a very advanced topic, and if you want to build extensible systems in Erlang, you need to understand all the details of how to do it. Fortunately, this section provides thorough instructions.

Over the course of this chapter you will also learn bunch of other useful things besides server behaviours. You will see how HTTP protocol works and how to design RESTful services on top of it, how to use TCP sockets more effectively, and how to increase stability of your system with well-designed OTP supervisors. You will have lots of fun doing binary pattern matching.

Code snippets: [TCP interface](http://github.com/erlware/Erlang-and-OTP-in-Action-Source/tree/master/chapter_11/tcp_interface/src/), [HTTP server behaviour](http://github.com/erlware/Erlang-and-OTP-in-Action-Source/tree/master/chapter_11/gen_web_server/src/), [REST interface](http://github.com/erlware/Erlang-and-OTP-in-Action-Source/tree/master/chapter_11/http_interface/src/).

### Chapter 12 — Drivers (Communication with C programs)

This chapter is tough — you have to be a C programmer to understand all the details. If you are not involved into C programming, it's still worthy to read it, just to understand the concept, although it might be hard to get through the entire text.

There are two types of drivers in Erlang: *port drivers* and *linked-in drivers*. The chapter starts with an overview of them both. It explains their benefits and drawbacks, how you should design the driver, and where you should handle the driver's state, global vs. instance variables.

The rest of the chapter is a tutorial of how to implement drivers. It describes three components that comprise driver implementation: C side, Erlang side and the protocol between them. It shows the differences in each component for both types of drivers, and it gives you recommendations of how you should approach the problem required communication with C code.

### Chapter 13 — Jinterface (Communication with Java programs)

Unlike the C driver implementation, connecting together Erlang and Java is pretty simple: you instantiate OtpNode class in the JVM thread, and it becomes available as Erlang node to any running Erlang application. You can start sending messages between Java and Erlang, and all Erlang terms will be properly converted to Java classes, and vice versa. All the magic is done in *Jinterface* library, which is a part of OTP distribution, and what you need to know to start using it is perfectly explained in this chapter.

After you learn how to work with Jinterface, you will apply this knowledge to building the bridge between the cache you implemented in the previous chapters and HBase. *HBase* is one of the modern NoSQL databases. If you didn't work with it before, don't worry — the authors will show you how to get started with it, and how to implement HBase connector using Java API. Having this API in place, all you need to do is to link it with the Erlang cache using the technique described above.

By the end of the chapter (and in fact end of the book) you will have a distributed cache written in Erlang backed by NoSQL database via Erlang-Java bridge. I don't know about you, but I was actually very impressed after I finished the coding and saw the entire solution working on my machine. It's really amazing that you can build pretty sophisticate piece of software with such a small amount of code.

Code snippets: [message receiver](http://github.com/erlware/Erlang-and-OTP-in-Action-Source/blob/master/chapter_13/simple_cache/java_src/HBaseNode.java) and [message responder](http://github.com/erlware/Erlang-and-OTP-in-Action-Source/blob/master/chapter_13/simple_cache/java_src/HBaseTask.java).

### Chapter 14 — Optimization and performance

As we all know, premature optimization is the root of all evil. In other words, don't spend time optimizing your solution before you actually measured the performance. That implies you must know what and how to measure. In this chapter authors describe the approach you should take when you prepare performance test, as well as the basic tools available in Erlang for performance testing: *cprof* and *fprof*.

The second part of the chapter explains the Erlang programming language *caveats*. You will see

- how primitive data types stored in memory, and which data structures you should use to fulfil performance requirements;
- how to use some built-in functions and operators properly;
- how to call function in different ways, and how performant those calls are;
- how compiler optimizes pattern matching and tail recursion;
- whether to use OTP behaviours or plain Erlang processes.

That concludes the main content of the book.

There are also two appendices in this book. The first one describes how to install Erlang on the OS of your choice. The second explains what *referential transparency* is and why lists in Erlang are implemented as they are.

### Conclusion

If you are an intermediate Erlang developer, go and buy this book! It will teach you how to build robust production systems following proven design principles and standard conventions. It will make your code easy to read and maintain. You will learn lots of new things and it will be a big step towards Erlang mastery.

Happy OTPing!
