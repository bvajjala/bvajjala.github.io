---
layout: post
title: "Farewell to Regular Web Development Approaches"
date: 2014-02-03 13:52:12 -0500
comments: true
categories: [SDLC, Web Development]
---

## [Farewell to Regular Web Development Approaches](/blog/2013/01/31/farewell-to-regular-web-development-approaches.html)

At my [previous company](http://wiredcraft.com), we built Web applications for medium to large organizations, often in the humanitarian and non-profit space, facing original problems revolving around data. Things like building the
[voting infrastructure for the Southern Sudan Referendum](http://wiredcraft.com/work/southern-sudan-referendum/index.html) helped us diversify our technical chops. But until a year ago, we were still mostly building regular Web applications; the user requests a page that we build and serve back.

**Until we started falling in love with APIs and static clients.**

It's not that we fundamentally disliked the previous approach. We just reached a point where we felt our goals weren't best served by this model. With lots of dynamic data, complex visualizations and a set of "static" interfaces, the traditional way was hindering our development speed and our ability to experiment. And so we got busy, experimenting at first with smaller parts of our projects (blog, help section, download pages…). We realized our use of complex pieces of softwares like content management systems had seriously biased our approach to problem solving. **The CMS had become a boilerplate, an unchallenged dependency.**

We've gradually adopted a pattern of building front-ends as static clients (may they be Web, mobile or 3rd party integrations) combined with, usually, one RESTful JSON API in the backend. And it works marvelously, thanks in part
to some awesome tech much smarter people figured out for us:

  * [Marionette](http://marionettejs.com) and [Backbone.js](http://backbonejs.org), [Component](http://github.com/component/component) (a personal favorite) and [Jekyll](http://github.com/mojombo/jekyll) allow us to build static HTML5 + JS + CSS clients for Web and mobile,
  * [node.js](http://nodejs.org) and [carcass](http://github.com/devo-ps/carcass) (alpha quality at this stage) in the backend for our APIs.

Most of what we build at devo.ps is stemming from this accelerated recovery and follow usually that order:

  1. We start by defining our API interface through user stories and methods,
  2. Both backend and front-end teams are getting cranking on implementing and solving the challenges specific to their part,

A lot of things happen in parallel and changes on one side rarely impact the other: we can make drastic changes in the UI without any change on the backend. And there were a lot of unexpected gains too, in security, speed and
overall maintainability. More importantly, we've freed a lot of our resources to focus on building compelling user experiences instead of fighting a large piece of software to do what we want it to do.

If you haven't tried building things that way yet, give it a try honestly (when relevant); it may be a larger initial investment in some cases but you'll come out on top at almost every level.