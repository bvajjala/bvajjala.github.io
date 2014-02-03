---
layout: post
title: "Vagrant, Docker and Ansible, WTF?"
date: 2013-09-25 12:00:00
comments: true
categories: [Vagrant, DOcker, Ansible]
---

## [Vagrant, Docker and Ansible. WTF?](/blog/2013/09/25/vagrant-docker-and-ansible-wtf.html)

Given that we're building a SaaS that helps our client managing their infrastructure, our team is pretty familiar with leveraging VMs and configuration management tools. We've actually been heavy users of Vagrant and [Ansible](http://devo.ps/blog/2013/07/03/ansible-simply-kicks-ass.html) for
the past year, and it's helped us tremendously normalize our development process.

As our platform grew in complexity, some additional needs emerged:

  * **Containerization**; we needed to be able to safely execute custom, and potentially harmful, code.
  * **Weight**; as we added more sub-systems to devo.ps, having full blown VMs proved to be hard to juggle with when testing and developing.

And that's why we ended up adding Docker to our development workflow. We were already familiar with it (as it powers some parts of the devo.ps infrastructure) and knew there would be obvious wins. In practice, we are shipping Docker containers in a main Vagrant image and drive some of the customization and upgrade with Ansible.

We'll probably write something about this approach in the coming weeks, but given the amount of confusion there is around what these technologies are, and how they're used, we thought we'd give you a quick tour on how to use them together.

Let's get started.

## Vagrant

You've probably heard about [Vagrant](http://www.vagrantup.com/); a healthy number of people have been writing about it in the past 6 months. For those of you who haven't, think of it as a VM without the GUI. At its core, Vagrant is a simple wrapper around Virtualbox/VMware.

A few interesting features:

  * **Boatloads of existing images**, just check [Vagrantbox.es](http://www.vagrantbox.es/) for example. 
  * **Snapshot and package your current machine** to a Vagrant box file (and, consequently, share it back).
  * **Ability to fine tune settings of the VM**, including things like RAM, CPU, APIC…
  * **Vagrantfiles**. This allows you to setup your box on init: installing packages, modifying configuration, moving code around…
  * **Integration with CM tools** like Puppet, Chef and Ansible.

Let's get it running on your machine:
{% codeblock %}
  1. First, [download Vagrant](http://downloads.vagrantup.com/) 
     and [VirtualBox](https://www.virtualbox.org/wiki/Downloads).
  2. Second, let's download an image, spin it up and SSH in:
	 $ vagrant init precise64 http://files.vagrantup.com/precise64.box
	 $ vagrant up
	 $ vagrant ssh
  3. There's no 3.
  4. There's a 4 if you want to access your (soon to be) deployed app; 
  5. you will need to dig around the Vagrant documentation to 
     [perform port forwarding](http://docs.vagrantup.com/v2/networking/forwarded_ports.html), 
     [proper networking](http://docs.vagrantup.com/v2/networking/private_network.html) 
     and update manually your `Vagrantfile`.
{% endcodeblock %}

## Docker

[Docker](http://docker.io) is a Linux container, written in [Go](http://golang.org) (yay!) and based on [lxc](http://en.wikipedia.org/wiki/LXC) (self-described as "chroot on steroids") and [AUFS](http://en.wikipedia.org/wiki/Aufs). Instead of providing a full VM, like you get with Vagrant, Docker provides you lightweight containers, that share the same kernel and allow to safely execute independent
processes.

Docker is attractive for many reasons:

  * **Lightweight**; images are much lighter than full VMs, and spinning off a new instance is lightning fast (in the range of seconds instead of minutes).
  * **Version control of the images**, which makes it much more convenient to handle builds.
  * **Lots of images** (again), just have a look at [the docker public index of images](https://index.docker.io).

Let's set up a Docker container on your Vagrant machine:
{% codeblock %}

  1. SSH in Vagrant if you're not in already:
    
     $ vagrant ssh

  2. Install Docker, 
  [as explained on the official website](http://http://docs.docker.io/en/latest/installation/ubuntulinux/#ubuntu-precise-12-04-lts-64-bit):
    
     $ sudo apt-get update
     $ sudo apt-get install linux-image-generic-lts-raring linux-headers-generic-lts-raring
     $ sudo reboot
     $ sudo sh -c "curl https://get.docker.io/gpg | apt-key add -"
     $ sudo sh -c "echo deb http://get.docker.io/ubuntu docker main &gt; /etc/apt/sources.list.d/docker.list"
     $ sudo apt-get update
     $ sudo apt-get install lxc-docker

  3. Verify it worked by trying to build your first container:
    
     $ sudo docker run -i -t ubuntu /bin/bash

  4. Great, but we'll need more than a vanilla Linux. To add our dependencies, for example to run a Node.js + MongoDB app, we're gonna start by creating a `Dockerfile`:
    
     FROM ubuntu
     MAINTAINER My Self me@example.com
     
     # Fetch Nodejs from the official repo (binary .. no hassle to build, etc.)
     ADD http://nodejs.org/dist/v0.10.19/node-v0.10.19-linux-x64.tar.gz /opt/
     
     # Untar and add to the PATH
     RUN cd /opt &amp;&amp; tar xzf node-v0.10.19-linux-x64.tar.gz
     RUN ln -s /opt/node-v0.10.19-linux-x64 /opt/node
     RUN echo "export PATH=/opt/node/bin:$PATH" &gt;&gt; /etc/profile
     
     # A little cheat for upstart ;)
     RUN dpkg-divert --local --rename --add /sbin/initctl
     RUN ln -s /bin/true /sbin/initctl
     
     # Update apt sources list to fetch mongodb and a few key packages
     RUN echo "deb http://archive.ubuntu.com/ubuntu precise universe" &gt;&gt; /etc/apt/sources.list
     RUN apt-get update
     RUN apt-get install -y python git
     RUN apt-get install -y mongodb
     
     # Finally - we wanna be able to SSH in
     RUN apt-get install -y openssh-server
     RUN mkdir /var/run/sshd
     
     # And we want our SSH key to be added
     RUN mkdir /root/.ssh &amp;&amp; chmod 700 /root/.ssh
     ADD id_rsa.pub /root/.ssh/authorized_keys
     RUN chmod 400 /root/.ssh/authorized_keys &amp;&amp; chown root. /root/.ssh/authorized_keys
     
     # Expose a bunch of ports .. 22 for SSH and 3000 for our node app
     EXPOSE 22 3000
     
     ENTRYPOINT ["/usr/sbin/sshd", "-D"]

  5. Let's build our image now:
    
     $ sudo docker build .
     
     # Missing file id_rsa.pub ... hahaha ! You need an ssh key for your vagrant user
     $ ssh-keygen
     $ cp -a /home/vagrant/.ssh/id_rsa.pub .
     
     # Try again
     $ sudo docker build .
     
     # Great Success! High Five!

  6. Now, let's spin off a container with that setup and log into it (`$MY_NEW_IMAGE_ID` is the last id the build process returned to you):
    
     $ sudo docker run -p 40022:22 -p 80:3000 -d $MY_NEW_IMAGE_ID
     $ ssh root@localhost -p 40022
{% endcodeblock %}

You now have a Docker container, inside a Vagrant box (_Inception_ style),
ready to run a Node.js app.

## Ansible

[Ansible](http://ansible.cc) is an orchestration and configuration management tool written in Python. If you want to learn more about Ansible (and you should…), [we wrote about it a few weeks ago](/blog/2013/07/03/ansible-simply-kicks-ass.html).

Let's get to work. We're now gonna deploy an app in our container:

{% codeblock %}

  1. [Install Ansible](/blog/2013/07/03/ansible-simply-kicks-ass.html), as we showed you in our previous post.

  2. Prepare your inventory file (`host`):
    
     app ansible_ssh_host=127.0.0.1 ansible_ssh_port=40022

  3. Create a simple playbook to deploy our app (`deploy.yml`):
    
     ---
     - hosts: app
       user: root
       tasks:
         # Fetch the code from github
         - name: Ensure we got the App code
           git:
             repo=git://github.com/madhums/node-express-mongoose-demo.git
             dest=/opt/node-express-mongoose-demo
         
         # NPM may or may not succeed, if you give it time, care, etc. it eventually works
         - name: Ensure the npm dependencies are installed
           command:
             chdir=/opt/node-express-mongoose-demo
             /opt/node/bin/npm install
           ignore_errors: yes
           
         # We will assume no changes in the default sample - or we should consider templates instead
         - name: Ensure the config files of the app
           command:
             creates=/opt/node-express-mongoose-demo/config/$item.js
             cp /opt/node-express-mongoose-demo/config/$item.example.js /opt/node-express-mongoose-demo/config/$item.js
           with_items:
             - config
             - imager
             
         # `initctl` is now linking to `true` and we have no access to services
         # Need to fake the start
         - name: Ensure mongodb data folders
           file:
             state=directory
             dest=$item
             owner=mongodb
             group=mongodb
           with_items:
             - /var/lib/mongodb
             - /var/log/mongodb
             
         # Super cheat combo !
         - name: Ensure mongodb is running
           shell:
             LC_ALL='C' /sbin/start-stop-daemon --background --start --quiet --chuid mongodb --exec  /usr/bin/mongod -- --config /etc/mongodb.conf
         
         # Cheating some more !
         - name: Ensure the App is running
           shell:
             chdir=/opt/node-express-mongoose-demo
             /opt/node/bin/npm start &amp;

  4. Run that baby:
    
     $ ansible-playbook -i host deploy.yml

  5. We're done, point your browser at `http://localhost:80` - assuming you have performed the redirection mentioned in the initial setup of your vagrant box.

{% endcodeblock %}

That's it. You've just deployed your app on Docker (in Vagrant).

## Let's wrap it up

So we just saw (roughly) how these tools can be used, and how they can be
complementary:

  1. Vagrant will provide you with a full VM, including the OS. It's great at providing you a Linux environment for example when you're on MacOS.
  2. Docker is a lightweight VM of some sort. It will allow you to build contained architectures faster and cheaper than with Vagrant.
  3. Ansible is what you'll use to orchestrate and fine-tune things. That's what you want to structure your deployment and orchestration strategy.

It takes a bit of reading to get more familiar with these tools, and we'll likely follow up on this post in the next few weeks. However, especially as a small team, this kind of technology allows you to automate and commoditize huge parts of your development and ops workflows. We strongly encourage you to make that investment. It has helped us tremendously increase the pace and
quality of our throughput.