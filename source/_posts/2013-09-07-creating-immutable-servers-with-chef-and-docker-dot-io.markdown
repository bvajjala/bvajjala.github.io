---
layout: post
title: "Creating immutable servers with chef and docker.io"
date: 2013-09-07 18:18
comments: true
categories: docker chef berkshelf
---

Building applications in a [docker.io](http://docker.io) Dockerfile is relatively simple,  but sometimes you want to just install the application exactly as you would normally via already built chef cookbooks.   Turns out this is actually pretty simple.


The first thing you'll need to do is build a container with chef-client and berkshelf installed.   You can grab the one I've built by running `docker pull paulczar/chef-solo` or build one youself from a `Dockerfile` that looks a little something like the following...

<!--more-->

### Creating a docker.io container with chef and berkshelf ###

``` ruby Dockerfile
# DOCKER-VERSION 0.5.3
FROM ubuntu:12.10
MAINTAINER Paul Czarkowski "paul@paulcz.net"

RUN apt-get -y update
RUN apt-get -y install curl build-essential libxml2-dev libxslt-dev git
RUN curl -L https://www.opscode.com/chef/install.sh | bash
RUN echo "gem: --no-ri --no-rdoc" > ~/.gemrc
RUN /opt/chef/embedded/bin/gem install berkshelf
```

_you'll notice I'm using the embedded chef ruby to install the berkshelf gem,  this is a handy shortcut to avoid messing around with random ruby versions from your distributions packaging._

run `$ docker build -t paulczar/chef-solo .` to build a usable docker container from the above `Dockerfile`.

### Using chef-solo and berkshelf to build an application in a docker.io container ###

My [example application](https://github.com/paulczar/docker-chef-solo) will install `Kibana3` to your docker container.   I'll step through how it works below.

#### Chef-Solo ####

To run `chef-solo` successfully we require two files.   `solo.rb` to set up file locations, and `solo.json' to set up the json / run list required for your application.

``` ruby chef.rb
root = File.absolute_path(File.dirname(__FILE__))

file_cache_path root
cookbook_path root + '/cookbooks'
```

``` json chef.json
{
  "kibana": {
    "webserver_listen": "0.0.0.0"
  },
  "run_list": [
    "recipe[kibana::default]"
  ]
} 
```

#### Berkshelf ####

To run `berkshelf` we need to build a Berksfile which contains a list of all the chef cookbooks required for the applocation.   Berkshelf will download these cookbooks to a local directory which will be usable by chef-solo.

``` ruby Berksfile
site :opscode

cookbook 'build-essential'
cookbook 'apache2'
cookbook 'git'
cookbook 'kibana', github: 'lusis/chef-kibana'
cookbook 'nginx' , github: 'opscode-cookbooks/nginx'
```

_You can see some of the cookbooks are being pulled from the opscode repository,  whereas others are being pulled directly from github._

#### Dockerfile ####

All that's left now is to create a Dockerfile that will bring it all together.

``` ruby Dockerfile
# DOCKER-VERSION 0.5.3
FROM paulczar/chef-client
MAINTAINER Paul Czarkowski "paul@paulcz.net"

RUN apt-get -y update
ADD . /chef
RUN cd /chef && /opt/chef/embedded/bin/berks install --path /chef/cookbooks
RUN chef-solo -c /chef/solo.rb -j /chef/solo.json
RUN echo "daemon off;" >> /etc/nginx/nginx.conf

CMD ["nginx"]
```

Run `$ docker build -t demo/kibana3 .` to build your application.

It will add the local files ( `solo.rb`, `solo.json`, `Berksfile` ) to /chef in the server and then call berkshelf to download the cookbooks and chef-solo to install your application.   Finally it will give `nginx` a directive to run in the foreground so that we don't have to do any sneaky prcess control to get it to work with the way `docker.io` runs processes.

To run the resultant `docker.io` container you simply need to run `$ docker run -d -p 80 demo/kibana3`

