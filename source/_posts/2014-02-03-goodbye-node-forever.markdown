---
layout: post
title: "Goodbye node-forever"
date: 2014-02-03 14:15:10 -0500
comments: true
categories: 
---

## [Goodbye node-forever, hello PM2](/blog/2013/06/26/goodbye-node-forever-hello-pm2.html)

![pm2 logo](http://apps.hemca.com/pm2/pres/pm22.png)

It's no secret that the devo.ps team has a crush on Javascript; node.js in the backend, AngularJS for our clients, there isn't much of our stack that isn't at least in part built with it. Our approach of building [static clients and RESTful JSON APIs](http://devo.ps/blog/2013/01/31/farewell-to-regular-web-development-approaches.html) means that we run a lot of node.js and I must admit that, despite all of it awesomeness, node.js still is a bit of a
headache when it comes to running in production. Tooling and best practices (think monitoring, logging, error traces…) are still lacking when compared to some of the more established languages.

So far, we had been relying on the pretty nifty [node-forever](https://github.com/nodejitsu/forever). Great tool, but a few things were missing:

  * Limited monitoring and logging abilities,
  * Poor support for process management configuration,
  * No support for clusterization,
  * Aging codebase (which meant frequent failures when upgrading Node).

This is what led us to write [PM2](https://github.com/Unitech/pm2) in the past couple months. We thought we'd give you a quick look at it while we're nearing a production ready release.

### So what's in the box?

First things first, you can install it with `npm`:

    
    npm install -g pm2

Let's open things up with the usual comparison table:

FeatureForeverPM2

Keep Alive

✔

✔

Coffeescript

✔

Log aggregation

✔

API

✔

Terminal monitoring

✔

Clustering

✔

JSON configuration

✔

And now let me geek a tad more about the main features…

### Native clusterization

Node v0.6 introduced the cluster feature, allowing you to share a socket across multiple networked Node applications. Problem is, it doesn't work out of the box and requires some tweaking to handle master and children processes.

PM2 handles this natively, without any extra code: PM2 itself will act as the master process and wrap your code into a special clustered process, as Nodejs does, to add some global variables to your files.

To start a clustered app using all the CPUs you just need to type something like that:

    
    $ pm2 start app.js -i max

Then;

    
    $ pm2 list

Which should display something like (ASCII UI FTW);

![pm2 list](http://apps.hemca.com/pm2/pres/pm2-list.png)

As you can see, your app is now forked into multiple processes depending on the number of CPUs available.

### Monitoring _a la_ termcaps-HTOP

It's nice enough to have an overview of the running processes and their status with the `pm2 list` command. But what about tracking their resources consumption? Fear not:

    
    $ pm2 monit

You should get the CPU usage and memory consumption by process (and cluster).

![pm2 monit](http://apps.hemca.com/pm2/pres/pm2-monit.png)

**Disclaimer**: [node-usage](https://github.com/arunoda/node-usage) doesn't support MacOS for now (feel free to PR). It works just fine on Linux though.

Now, what about checking on our clusters and GC cleaning of the memory stack?
Let's consider you already have an HTTP benchmark tool (if not, you should definitely check [WRK](https://github.com/wg/wrk)):

    
    $ express bufallo     // Create an express app
    $ cd bufallo
    $ npm install
    $ pm2 start app.js -i max
    $ wrk -c 100 -d 100 http://localhost:3000/

In another terminal, launch the monitoring option:

    
    $ pm2 monit

W00t!

### Realtime log aggregation

Now you have to manage multiple clustered processes: one who's crawling data, another who is processing stuff, and so on so forth. That means logs, lots of it. You can still handle it the old fashioned way:

    
    $ tail -f /path/to/log1 /path/to/log2 ...

But we're nice, so we wrote the `logs` feature:

    
    $ pm2 logs

![pm2 monit](http://apps.hemca.com/pm2/pres/pm2-logs.png)

### Resurrection

So things are nice and dandy, your processes are humming and you need to do a hard restart. What now? Well, first, dump things:

    
    $ pm2 dump

From there, you should be able to resurrect things from file:

    
    $ pm2 kill     // let's simulate a pm2 stop
    $ pm2 resurect // All my processes are now up and running 

### API Health point

Let's say you want to monitor all the processes managed by PM2, as well as the status of the machine they run on (and maybe even build a nice Angular app to consume this API…):

    
    $ pm2 web

Point your browser at `http://localhost:9615`, aaaaand… done!

### And there's more…

  * Full tests,
  * Generation of `update-rc.d` (`pm2 startup`), though still very alpha,
  * Development mode with auto restart on file change (`pm2 dev`), still very drafty too,
  * Log flushing,
  * Management of your applications fleet via JSON file,
  * Log uncaught exceptions in error logs,
  * Log of restart count and time,
  * Automated killing of processes exiting too fast.

### What's next?

Well first, you could show your love on Github (we love stars):
[https://github.com/Unitech/pm2](https://github.com/Unitech/pm2).

We developed PM2 to offer an advanced and complete solution for Node process management. We're looking forward to getting more people helping us getting there: pull requests are more than welcome. A few things already on the
roadmap that we'll get right at once we have a stable core:

  * Remote administration/status checking,
  * Built-in inter-processes communication channel (message bus),
  * V8 GC memory leak detection,
  * Web interface,
  * Persistent storage for monitoring data,
  * Email notifications.

Special thanks to [Makara Wang](https://github.com/makara) for concepts/tools and [Alex Kocharin](https://github.com/rlidwka) for advices and pull requests.