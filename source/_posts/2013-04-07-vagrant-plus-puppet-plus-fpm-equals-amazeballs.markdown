---
layout: post
title: "Vagrant+Puppet+FPM=Amazeballs"
date: 2013-04-07 18:23
comments: true
categories: [devops, puppet, vagrant, fpm, packaging]
---

Lately I've been doing a lot of prototyping with [Vagrant](http://www.vagrantup.com/), specifically for a couple of distinct activities:- 

* building puppet modules using [the excellent puppet sandbox](https://github.com/elasticdog/puppet-sandbox) project 
* and building RPM packages with [FPM](https://github.com/jordansissel/fpm).

I realized I was spending a bunch of time flipping back and forth between Vagrant environments and I had no quick way to utilize RPMs built with FPM inside my puppet modules.    

<!--more-->

An idea was born.   I forked off the [puppet sandbox](https://github.com/paulczar/puppet-sandbox) project and added a Yum repo module `repository` to the standalone puppet provisioner that vagrant uses when it first brings up a box.   It adds a Yum repo on the puppet server called `sandbox` and adds a repo file to the client boxes pointing to the repo.   Now I can simply push an RPM to `packages/rpm` and run `vagrant provision puppet` which reruns puppet and rebuilds the yum repo.

Given that I often flip back and forth between Ubuntu and CentOS boxes I also created `Vagrantfile.centos63` and `Vagrantfile.precise64` so I can swiftly destroy the existing environment and bring up another of a different flavour by simply symlinking `Vagrantfile` to the appropriate file.

This worked out pretty well for a while until I realized I was still jumping back and forth between vagrant environments and I realized I had another improvement to make.   So I then went on to create a definition in the puppet sandbox `Vagrantfile` file for a `FPM server` and a new module in the provisioner to install FPM on it.   Given that this module simply adds a few packages this module works for both CentOS and Ubuntu.   

I also created a couple of sample scripts to download source and build RPMs for both Redis and Elasticsearch which get pushed via the provisioner to `/tmp/redis-rpm.sh` and `/tmp/elasticsearch-rpm.sh`

Now ( For CentOS boxes at least ) I can very quickly iterate on puppet modules and create RPM packages on the fly and have them instantly available.   The process is very simple and looks a little something like this : 


```
$ git clone https://github.com/paulczar/puppet-sandbox
$ cd puppet-sandbox
$ vagrant up puppet fpm client1
$ vagrant ssh fpm
[vagrant@fpm ~]$ sudo /tmp/redis-rpm.sh
  ... 
  ... A bunch of scrolling text while files are downloaded and rpm is built
  ...
[vagrant@fpm ~]$ exit
$ vagrant provision puppet
$ vagrant ssh client1
[vagrant@client1 ~]$ sudo yum clean all
[vagrant@client1 ~]$ sudo yum -y install redis
[vagrant@client1 ~]$ sudo service redis-server start
[vagrant@client1 ~]$ redis-cli ping
PONG
[vagrant@client1 ~]$
```

If I'm building a puppet module that needs redis I can now add the following to it's init.pp ( or more properly create a module for redis and request it from the module I'm building ) 

```
  package { 'redis':
    ensure => 'present';
  }
```

Of course Debian/Ubuntu doesn't use Yum/RPM for package management.    I'd love to accept a pull request from somebody who wants to extend it to also support a local APT repository.   I left breadcrumbs in the `repository` module for some appropriate classes to be spliced in...