---
layout: post
title: "Logstash + Opscode Omnibus"
date: 2013-05-06 19:03
comments: true
categories: logstash omnibus devops opscode
---

At [DevOps Days Austin](http://devopsdays.org/events/2013-austin/) [@mattray](http://twitter.com/mattray) did an Openspace session on [Omnibus](https://github.com/opscode/omnibus-ruby) which is a toolset based around the concept of installing an app and all of it's prerequisites from source into a directory and then building a package ( either .deb or .rpm ) of that using [fpm](https://github.com/jordansissel/fpm).

Having battled many times with OS Packages trying to get newer versions of Ruby, or Redis or other software installed and having to hunt down some random package repo or manually build from source this seems like an excellent idea.

To learn the basics I decided to build an [omnibus package for fpm](https://github.com/paulczar/omnibus-fpm) which helped me work out the kinks and learn the basics.   

<!--more-->

From there I moved onto something a little more ambitious... [logstash](http://logstash.net/), which is an awesome opensource project for log aggregation and searching.     

Using Omnibus I took the Logstash .jar file and bundled in Redis, Kibana, Kibana3(+NodeJS), RabbitMQ, Elasticsearch along with all of their depedencies into a big fat package which installs to /opt/logstash and includes init scripts and default configs for each.

The Logstash Omnibus project can be found [here](https://github.com/paulczar/omnibus-logstash).  I also uploaded the resultant packages for [Ubuntu 12.04](https://s3-us-west-2.amazonaws.com/paulcz-packages/logstash-omnibus-1.1.10_amd64.deb) and [RHEL 6](https://s3-us-west-2.amazonaws.com/paulcz-packages/logstash-omnibus-1.1.10.el6.x86_64.rpm).
 
This gives us a really powerful platform to deploy logstash and all of its prequisites in a completely repeatable manner and not have to worry about the existing versions of Ruby, Java, etc.    It also gives a super simple testing platform where a new user to logstash can install logstash with a single `dpkg` or `rpm` command and immediately be able to push logs to it via syslog or redis.

Read more about using and building the [Logstash Omnibus package here](https://github.com/paulczar/omnibus-logstash/blob/master/README.md)


