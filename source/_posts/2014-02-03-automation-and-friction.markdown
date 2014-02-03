---
layout: post
title: "Automation And Friction"
date: 2014-02-03 14:06:45 -0500
comments: true
categories: 
---

## [Automation And Friction](/blog/2013/06/20/automation-and-friction.html)

I'll admit that the devo.ps team is a lazy bunch; we like to forget about things, especially the hard stuff. Dealing with a complex process invariably leads one of us to vent about how "we should automate that stuff". That's what our team does day and night:

  1. Dumb things down, lower barriers of entry, and then…
  2. **Automate all the things!**

This has transpired through every layer of our company, from engineering to operations. Recently we've started pushing on a third point, but first let me rant a bit…

### The ever increasing surface of friction

The past few years have seen a healthy push on UI and UX. Even developer tools and enterprise software, historically less user-friendly, have started adopting that trend. We now have things like Github. Great.

This trend grew in parallel with the adoption of SaaS. SaaS are the results of teams focused on specific problems, with the user experience often being a key component (not to undervalue good engineering). It's pretty standard for these services to offer an API for integration's sake. [Our CRM](https://getbase.com) plays nicely with Dropbox, GMail and a gazillion other services. Again, great.

**However, the success of SaaS means the surface of interfaces we're dealing with is constantly stretching. This is far more difficult to overcome than poor UI or UX.** Many of us have witnessed teams struggling to get adoption on a great tool that happen to be one too many. There's not much you can do about it.

### A bot to rule them all…

![Borat is omnipotent](http://devo.ps/images/posts/borat.png)

Our team has tried a lot of different approaches over the years. We kicked the tires on a lot of products and ended up doing as usual:

  1. **Simplify**. For example, we use Github to manage most tasks and discussions, including operations (HR, admin, …), and marketing. We used [Trello](http://trello.com/) alongside Github for a while and we loved it. But it silo-ed the discussions. Everything from our employee handbook to tasks for buying snacks for the office are now on Github. It also had an interesting side effect on transparency, but I'll talk about this another time.

  2. **Automate**. We automate pretty much everything we can. When you apply to one of our job by email for example, we push the attachments in Dropbox (likely your resume) and create a ticket with the relevant information on Github. [Zapier](http://zapier.com) is great for this kind of stuff by the way.

  3. **Make it accessible**. That's the most important point for us at this stage. Borat, our [Hubot](http://hubot.github.com) chat bot, is hooked up with most of our infrastructure and is able to pass on requests to the services we use as well as some of our automation. If one of us is awake, chances are you can find us on the chat, making it the most ubiquitous interface for our team:

    * Need to deploy some code on production or modify some configuration on a server? Ask Borat, he'll relay your demands to the [devo.ps](http://devo.ps) API.
    * Your latest commit broke the build? A new mail came from support? Expect to hear about it from Borat.
    * Need to use our time tracker? Just drop a message to the bot when you're starting your task and let him know when you're done.
    * Need to call for a SCRUM? Just mention the Github team you want to chat with and Borat will create a separate channel and invite the right people to join.
    * Somebody is at the door? Ask the bot to open it for you (you gotta love hacking on Raspberry PI).

Anybody with access to our bot's repository can add a script to hook him up to a new service. Git push, kill the bot and wait for him to come back to life with new skills. The tedious stuff ends up sooner or later scripted and one sentence away.

Really, try it. It's worth the investment.