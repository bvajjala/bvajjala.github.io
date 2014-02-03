---
layout: post
title: "Managing docker services with this one easy trick"
date: 2013-10-20 15:46
comments: true
categories: docker runit
---

I have been having a lot of internal debate about the idea of running more than one service in a docker container.   A Docker container is built to run a single process in the foreground and to live for only as long as that process is running.  This is great in a utopian world where servers are immutable and sysadmins drink tiki drinks on the beach,  however it doesn't always translate well to the real world.

Examples where you might want to be able to run multiple servers span from the simple use case of running `sshd` as well as your application to running a web app such as `wordpress` where you might want both `apache` and `mysql` running in the same container.

Wrapping your applications in a supervisor daemon such as `runit` seems like a perfect fit for this.  All you need to do is install `runit` as part of your `dockerfile` and then create appropriate service directories for the apps you want to run in the container.    I was doing some testing of this when I realized a quirk of `runit` which I could exploit for evil.   

To start or stop a service with `runit` is simply a matter of creating or deleting a symlink in a service directory,   so in theory if you could expose that directory to the server hosting the container you could exploit that to start and stop services from outside of the container.  `Docker` volume mapping allows exactly this! 

Below you will find examples of running three services (logstash,elasticsearch,kibana) that make up the `logstash` suite.

<!--more-->

## Start by cloning the demo git repository and run demo.sh ##

```
$ git clone https://github.com/paulczar/docker-runit-demo.git
$ cd docker-runit-demo
$ ./demo.sh
```

### demo.sh script ###

#### Step 1:  Build the container ####

The script uses the below `Dockerfile` to build the base container that we'll be running.

```
# Installs runit for service management
#
# Author: Paul Czarkowski
# Date: 10/20/2013

FROM paulczar/jre7
MAINTAINER Paul Czarkowski "paul@paulcz.net"

RUN apt-get update

RUN apt-get -y install curl wget git nginx
RUN apt-get -y install runit || echo

CMD ["/usr/sbin/runsvdir-start"]

```

#### Step 2: Install the applications ####

This will take a few minutes the first time as it needs to download `logstash`, `kibana`, and `elasticsearch` and stage them in a local `./opt`directory.

#### Step 3: Start the Docker container ####

Starts the `Docker` container with the following command:

```
docker run -d -p 8080:80 -p 5014:514 -p 9200:9200 \
  -v $BASE/opt:/opt \
  -v $BASE/sv:/etc/sv \
  -v $BASE/init:/etc/init \
  -v $BASE/service:/etc/service \
  demo/runit
```

The container should be up and running

```
$ docker ps
ID                  IMAGE               COMMAND                CREATED             STATUS              PORTS
eb495ad92ba0        demo/runit:latest   /usr/sbin/runsvdir-s   4 seconds ago       Up 3 seconds        5014->514, 8080->80, 9200->9200   
```

However there aren't any services running!

```
$ curl localhost:8080
curl: (56) Recv failure: Connection reset by peer
$ curl localhost:9200
curl: (56) Recv failure: Connection reset by peer
```

We can start the services with the following commands

```
$ cd service
$ ln -s ../sv/elasticsearch
$ ln -s ../sv/logstash
$ ln -s ../sv/kibana
cd ..
```

We can now see the services are running, test the ports and send some data to logstash.

```
$ curl localhost:8080      
<!DOCTYPE html><!--[if IE 8]><html class="no-js lt-ie9" lang="en"><![endif]--><!--[if gt IE 8]><!--><html class="no-js" lang="en">
...
curl localhost:9200
{
  "ok" : true,
  "status" : 200,
...
$tail -100 /var/log/syslog | nc localhost 5014
```

Stop a service ?

```
$ rm service/elasticsearch
$ rm service/logstash
$ rm service/kibana
```

## Bonus Round: Logs! ##

The beautify of doing this is that we're actually logging the application output to a mounted volume.   This means we now have access to their logs from the host machine.

```
$ tail opt/logstash/logs/current
$ tail opt/elasticsearch-0.90.5/logs/current
$ tail opt/kibana/logs/access.log
```

## Cleanup ##

Unfortunately any files created inside the docker instance are owned by root ( an artifact of docker daemon running as root ).   If you're in The following script will clean out any such files after you've stopped the docker container.

It will delete any files/dirs inside your current directory that are owned by root.  Obviously it can be very dangerous to run ... so be careful where you run it from!

```
$ sudo find . -uid 0   -exec rm -rfv {} \;
```