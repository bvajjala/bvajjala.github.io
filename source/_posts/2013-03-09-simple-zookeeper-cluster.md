---
layout:     post
title:      "Simple Zookeeper cluster"
date:       2013-03-09 12:00:00
categories: [Clustering,ZooKeeper]
tags:       [clustering,zookeeper]
published:  true
---

Sometimes I need to run [ZooKeeper][1] ensemble on my development box to test my application on the production-like environment. I found that recreating the whole ensemble from scratch is much faster than cleaning it up using ZooKeeper CLI tool. To automate this process I created a bash script which I want to share in this blog post. I hard-coded all the paths in the script using my regular conventions. You might need to change them to yours — it should be fairly straightforward.

Before you can use the script, you need to install ZooKeeper on your box. That's what I did on my machine

    $ cd /opt
    $ sudo mkdir zookeeper
    $ sudo chown -R andrey:admin zookeeper
    $ cd zookeeper
    $ wget http://apache.mirror.rafal.ca/zookeeper/zookeeper-3.4.5/zookeeper-3.4.5.tar.gz
    $ tar xf zookeeper-3.4.5.tar.gz
    $ rm zookeeper-3.4.5.tar.gz
    $ ln -s zookeeper-3.4.5 zookeeper

In the end you should have a ZooKeeper installed in */opt/zookeeper/zookeeper* directory.

Now download, chmod, and run the [script][2]. It will create the following files

    /opt/zookeeper/zookeeper/cluster
    ├── server1
    │   ├── conf
    │   │   ├── log4j.properties
    │   │   └── zoo.cfg
    │   ├── data
    │   │   └── myid
    │   └── logs
    ├── server2
    │   ├── conf
    │   │   ├── log4j.properties
    │   │   └── zoo.cfg
    │   ├── data
    │   │   └── myid
    │   └── logs
    ├── server3
    │   ├── conf
    │   │   ├── log4j.properties
    │   │   └── zoo.cfg
    │   ├── data
    │   │   └── myid
    │   └── logs
    ├── start.sh
    └── stop.sh

This is the minimum configuration for 3-node ensemble (cluster), which is recommended for production. To start the cluster, run the following command

    $ cd /opt/zookeeper/zookeeper
    $ cluster/start.sh

Check the log files to see if the cluster is successfully started

    $ tail -f cluster/server{1,2,3}/logs/zookeeper.out

When the cluster is up and running, you can test your application. After you are done, shutdown the cluster using the following command

    $ cluster/stop.sh
    $ ps -ef | grep java

To recreate a clean cluster, just run the script again

    $ ./zookeeper-init-ensemble.sh


[1]: http://zookeeper.apache.org
[2]: https://gist.github.com/ndpar/5105486
