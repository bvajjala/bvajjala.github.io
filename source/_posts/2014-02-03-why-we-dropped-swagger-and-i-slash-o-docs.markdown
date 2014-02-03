---
layout: post
title: "Why We Dropped Swagger And I/O Docs"
date: 2014-02-03 13:49:24 -0500
comments: true
categories: 
---

## [Why We Dropped Swagger And I/O Docs](/blog/2013/02/07/why-we-dropped-iodocs-and-swagger.html)

As we started investing in [our new strategy](/blog/2013/01/31/farewell-to-regular-web-development-approaches.html) at [my previous company](http://wiredcraft.com), we looked around for solutions to document APIs. It may not be the sexiest part of the project, but documentation is the first step to designing a good API. And I mean **first** as in "before you even start writing tests" (yes, you should be writing tests first too).

We originally went with a simple Wiki page on Github, which served us just fine in the past. But it quickly became clear that it wasn't going to cut it. We started thinking about what good documentations is. We're fans of the single page approach that the [Backbone.js documentation](http://backbonejs.com) illustrates well and clearly remembered [Github](http://developer.github.com/) and [Stripe](https://stripe.com/docs) as easy and well organized resources. Some Googling later, we were contemplating Wordnik's [Swagger](http://developers.helloreverb.com/swagger/) and Mashery's [I/O Docs](http://www.mashery.com/product/io-docs). We later settled for I/O Docs as it is built with node.js and was more straightforward to set up (for us at least).

Once again, we hit a wall with this approach:

  1. **No proper support for JSON body**: we don't do much with parameters and mostly send JSON objects in the body of our requests, [using HTTP verbs for the different types of operations](http://blog.apigee.com/detail/restful_api_design_nouns_are_good_verbs_are_bad) we perform on our collections and models in the backend. Swagger and I/O Docs fall short of support for it, letting you simply dump your JSON in a field: not ideal.

  2. **You're querying the actual API**: to be fair, this is an intentional feature. Now some of you may find it interesting that your documentation allows users to easily run calls against your API. That's what [Flickr does with their API explorer](http://www.flickr.com/services/api/explore/flickr.activity.userComments), and we used to think it was pretty neat. But once we started using it, we saw the risks of exposing so casually API calls that can impact your platform (especially with [devo.ps](http://devo.ps) which deals with your actual infrastructure). I guess you could set up a testing API for that very purpose, but that's quite a bit of added complexity (and [we're lazy](http://blogoscoped.com/archive/2005-08-24-n14.html)).

And that's how we ended up putting together [Carte](https://github.com/devo- ps/carte), a very lightweight Jekyll-based solution: drop a new post for each API call, following some loose format and specifying a few bits of meta data in the YAML header (type of the method, path…) and you're good to go.

[![Screenshot of Carte](/images/posts/carte-screenshot.png)](http://devo-ps.github.com/carte)

We're real suckers for Jekyll. We've actually used it to build quite a few static clients for our APIs. One of the advantages of this approach is that we can bundle our documentation with our codebase by simply pushing it on the
`gh-pages` branch, and it pops up as a Github page. That's tremendously important for us as it make it very easy for developers to keep the documentation and the code in synch.

Carte is intentionally crude: [have a look at the README and hack at will](https://github.com/devo-ps/carte). Drop us a shout at [@devo_ps](https://twitter.com/devo_ps) if you need help or want to suggest a feature.
