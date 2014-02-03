---
layout: post
title: "I Can Haz Init Script"
date: 2014-02-03 14:21:03 -0500
comments: true
categories: 
---

## [I Can Haz Init Script](/blog/2013/06/19/I-can-haz-init-script.html)

Something went awfully wrong, and a rogue process is eating up all of the resources on one of your servers. You have no other choice but to restart it. No big deal, really; this is the age of disposable infrastructure after all. Except when it comes back up, everything starts going awry. Half the stuff supposed to be running is down and it's screwing with the rest of your setup.

![INIT SCRIPTS, Y U NO LIKE?](/images/posts/y-u-no-guy.png)

You don't get to think about them very often, but init scripts are a key piece of a sound, scalable strategy for your infrastructure. It's a [mandatory best practice](). Period. And there are quite a few things in the way of getting them to work properly at scale in production environments. It's a tough world out there.

### What we're dealing with…

#### Packages

Often enough, you're gonna end up installing a service using the package manager of your distro: `yum`, `apt-get`, you name it. These packages usually come with an init script that should get you started.

Sadly, as your architecture grows in complexity, you'll probably run into some walls. Wanna have multiple memcache buckets, or several instances of redis running on the same box? You're out of luck buddy. Time to hack your way
through:

  * Redefine your start logic,
  * Load one or multiple config files from `/etc/defaults` or `/etc/sysconfig`,
  * Deal with the PIDs, log and lock files,
  * Implement conditional logic to start/stop/restart one or more of the services,
  * Realize you've messed something up,
  * Same player shoot again.

Honestly: PITA.

#### Built from source

First things first: **you shouldn't be building from source** (unless you really, really need to).

Now if you do, you'll have to be thorough: there may be samples of init scripts in there, but you'll have to dig them out. `/contrib`, `/addons`, …it's never in the same place.

And that makes things "fun" when you're [trying to unscrew things on a box](http://devo.ps/blog/2013/03/06/troubleshooting-5minutes-on-a-yet-unknown-box.html):

  * You figured out that MySQL is running from `/home/user/src/mysql`,
  * You check if there's an init script: no luck this time…
  * You try to understand what exactly launched `mysqld_safe`,
  * You spend a while digging into the bash history smiling at typos,
  * You stumble on a `run.sh` script (uncommented, of course) in the home directory. Funny enough, it seems to be starting everything from MySQL, NGINX and php-fpm to the coffee maker.
  * You make a mental note to try and track down the "genius" who did that mess of a job, and get busy with converting everything to a proper init script.

Great.

### Why existing solutions suck

Well, based on what we've just seen, you really only have two options:

  1. **DIY**; but if you're good at what you do, you're probably also lazy. You may do it the first couple times, but that's not gonna scale, especially when dealing with the various flavors of init daemons (upstart, systemd…),
  2. **Use that thing called "the Internet"**; you read through forum pages, issue queues, gists and if you're lucky you'll find a perfect one (or more likely 10 sucky ones). Kudos to all those of whom shared their work, but you'll probably be back to option 1.

### We can do better than this

You'll find a gazillion websites for pictures of kittens, but as far as I know, there is no authoritative source for init scripts. That's just not right: we have to fix it. A few things I'm aiming for:

  * **Scalable**; allow for multiple instances of a service to be started at once from different config files (see the memcache/redis example),
  * **Secure**; ensure `configtest` is run before a restart/reload (because, you know, a faulty config file preventing the service to restart is kind of a bummer),
  * **Smart**; ensuring for example that the cache is aggressively flushed before restarting your database (so that you don't end-up waiting 50 min for the DB to cleanly shutdown).

[I've just created a repo](https://github.com/devo-ps/init-scripts) where I'll be dumping various init scripts that will hopefully be helpful to others. I'd love to get suggestions or help.

And by the way, things are not much better with applications, though we're trying our best to improve things there too with things like [pm2](https://github.com/Unitech/pm2) (fresh and shinny, more about it in a later post).