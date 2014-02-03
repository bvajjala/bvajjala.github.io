---
layout: post
title: "Ansible Simply Kicks Ass"
date: 2014-02-03 14:12:31 -0500
comments: true
categories: 
---
## [Ansible Simply Kicks Ass](/blog/2013/07/03/ansible-simply-kicks-ass.html)

The devo.ps team has been putting quite a few tools to the test over the years when it comes to managing infrastructures. We've developed some ourselves and have adopted others. While the choice to use one over another is not always as clear-cut as we'd like (I'd love to rant about monitoring but will leave that for a later post), we've definitely developed kind of a crush for [Ansible](https://github.com/ansible/ansible) in the past 6 months. We went through years of using Puppet, then Chef and more recently Salt Stack, before Ansible gained unanimous adoption among our team.

What makes it awesome? Well, on top of my head:

  * It's **agent-less** and works by default in **push mode** (that last point is subjective, I know).
  * It's **easy to pick up** (honestly, try and explain Chef or Puppet to a developer and see how long that takes you compared to Ansible).
  * It's **just Python**. It makes it easier for people like me to contribute (Ruby is not necessarily that mainstream among ops) and also means minimal dependency on install (Python is shipped by default with Linux).
  * It's **picking up steam** at an impressive pace (I believe we're at 10 to 15 pull requests a day).
  * And it has all of the good stuff: idempotence, roles, playbooks, tasks, handlers, lookups, callback plugins…

Now, Ansible is still very much in its infancy and some technologies may not yet be supported. But there are a great deal of teams pushing hard on contributions, including us. In the past few weeks, for example, we've
contributed both Digital Ocean and Linode modules. And we have a lot more coming, including some experimentations with Vagrant.

Now, an interesting aspect of Ansible, and one that makes it so simple, is that it comes by default with a tool-belt. Understand that it is shipped with a range of modules that add support for well known technologies: [EC2,
Rackspace, MySQL, PostgreSQL, rpm, apt,…](http://www.ansibleworks.com/docs/modules.html). This now includes our
Linode contribution. That means that with the latest version of Ansible you can spin off a new Linode box as easily as:

    
    ansible all -m linode -a "name='my-linode-box' plan=1 datacenter=2 distribution=99 password='p@ssword' "

Doing this with Chef would probably mean chasing down a knife plugin for adding Linode support, and would simply require a full Chef stack (say hello to RabbitMQ, Solr, CouchDB and a gazillion smaller dependencies). Getting
Ansible up and running is as easy as: 

    
    pip install ansible

Et voila! You gotta appreciate the simple things in life. Especially the life of a sysadmin.