---
layout: post
title: "ZooKeeper vs. Doozer vs. Etcd"
date: 2014-02-03 13:18:39 -0500
comments: true
categories: [Zookeper, Doozer, Etcd]
---
## [ZooKeeper vs. Doozer vs. Etcd](/blog/2013/09/11/zookeeper-vs-doozer-vs-etcd.html)

While [devo.ps](http://devo.ps) is fast approaching a public release, the team has been dealing with an increasingly complex infrastructure. We more recently faced an interesting issue; how do you share configuration across a cluster of servers? More importantly, how do you do so in a resilient, secure, easily deployable and speedy fashion?

That's what got us to evaluate some of the options available out there;
ZooKeeper, Doozer and etcd. These tools all solve similar sets of problems but their approach differ quite significantly. Since we spent some time evaluating them, we thought we'd share our findings.

### ZooKeeper, the old dog

[ZooKeeper](http://zookeeper.apache.org/) is the most well known (and oldest) project we've looked into. It's used by a few big players (Rackspace, Yahoo, eBay, [Youtube](https://news.ycombinator.com/item?id=6367979)) and is pretty mature.

It was created by Yahoo to deal with distributed systems applications. I strongly recommend you [read the "making of"](http://developer.yahoo.com/blogs/hadoop/apache-zookeeper-making-417.html) if you're interested in understanding where Yahoo came from when they wrote it.

It stores variables in a structure similar to a file system, an approach that both Doozer and etcd still follow. With ZooKeeper, you maintain a cluster of servers communicating with each other that share the state of the distributed configuration data. Each cluster elects one "leader" and clients can connect to any of the servers within the cluster to retrieve the data. Zookeeper uses its own algorithm to handle distributed storage.

  * **Pros**:

    * **Mature technology**; it is used by some big players (eBay, Yahoo et al).
    * **Feature-rich**; lots of client bindings, tools, API…
  * **Cons**:

    * **Complex**; ZooKeeper is not for the faint of heart. It is pretty heavy and will require you to maintain a fairly large stack.
    * **It's… Java**; not that we especially hate Java, but it is on the heavy side and introduce a lot of dependencies. We wanted to keep our machines as lean as possible and usually shy away from dependency heavy technologies.
    * **Apache…**; we have mixed feelings about the Apache Foundation. ["Has Apache Lost Its Way?"](http://www.infoworld.com/d/open-source-software/has-apache-lost-its-way-225267) summarizes it pretty well.

### Doozer, kinda dead

[Doozer](https://github.com/ha/doozerd) was developed by Heroku a few years
ago. It's written in Go (yay!), which means it compiles into a single binary
that runs without dependencies. On a side-note, if you're writing code to
manage infrastructure, you should spend some time [learning
Go](http://golang.org/).

Doozer got some initial excitement from the developer community but seems to
have stalled more recently, with many forks being sporadically maintained and
no active core development.

It is composed of [a daemon](https://github.com/ha/doozerd) and [a
client](https://github.com/ha/doozer). Once you have at least one Doozer
server up, you can add any number of servers and have clients get and set data
by talking to any of the servers within that cluster.

It was one of the first practical implementations (as far as I know) of the
[Paxos algorithm](http://en.wikipedia.org/wiki/Paxos_(computer_science)). This
means operations can be slow when compared to dealing with a straight database
since cluster-wide consensus needs to be reached before committing any
operation.

Doozer was a step in the right direction. It is simple to use and setup.
However, after using it for a while we started noticing that a lot of its
parts felt unfinished. Moreover, it wasn't answering some of our needs very
well (encryption and ACL).

  * **Pros**:

    * **Easy to deploy, setup and use** (Go, yay!)
    * **It works**; lots of people have actually used it in production.
  * **Cons**:

    * **Pretty much dead**: the core project hasn't been active in a while (1 commit since May) and is pretty fragmented (150 forks…).
    * **Security**; no encryption and a fairly simple secure-word based authentication.
    * **No ACL**; and we badly needed this.

### etcd

After experiencing the shortcomings of Doozer, we stumbled upon a new
distributed configuration storage called
[etcd](https://github.com/coreos/etcd). It was first released by the
[CoreOS](http://coreos.com) team a month ago.

Etcd and Doozer look pretty similar, at least on the surface. The most obvious
technical difference is that ectd uses the [Raft
algorithm](http://en.wikipedia.org/wiki/Raft_%28computer_science%29) instead
of Paxos. Raft is designed to be [simpler](https://ramcloud.stanford.edu/wiki/
download/attachments/11370504/raft.pdf) and
[easier](http://kellabyte.com/2013/05/09/an-alternative-to-paxos-the-raft-
consensus-algorithm/) to implement than Paxos.

Etcd's architecture is similar to Doozer's. It does, however, store data
persistently (writes log and snapshots), which was of value to us for some
edge cases. It also has a better take on security, with CA's, certs and
private keys. While setting it up is not straightforward, it adds conveniency
and safety of mind.

Beyond the fact that it answered some of our more advanced needs, we were
seduced (and impressed) by the development pace of the project.

  * **Pros**:

    * **Easy to deploy, setup and use** (yay Go and yay HTTP interfaces!).
    * **Data persistence**.
    * **Secure**: encryption and authentication by private keys.
    * **Good documentation** (if a little bit obscure at times).
    * Planned ACL implementation.
  * **Cons**:

    * (Very) **young project**; interfaces are still moving pretty quickly.
    * Still not a perfect match, especially in the way that data is spread.

### The DIY approach (yeah, right..?)

It is only fair that technical teams may rely on their understanding of their
infrastructure and coding skills to get _something that just works™_ in place.
We haven't seriously considered this approach as we felt that getting security
and distributed state sharing right was going to be a bigger endeavor than we
could afford (the backlog is full enough for now).

### Conclusion

In the end, we decided to give etcd a try. So far it seems to work well for
our needs and the very active development pace seems to validate our choice.
It has proven resilient and will likely hold well until we have the resources
to either customize its data propagation approach, or build our own solution
that will answer some needs it is not likely to answer (we've already looked
into doing so with ZeroMQ and Go).