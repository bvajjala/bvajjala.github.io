---
layout: post
title: "Creating a Github Pages Blog With Octopress"
date: 2012-12-15 11:10
comments: true
categories: [ octopress, blog, github ]
---

A lot of tech bloggers will write their blog posts in [Markdown](http://daringfireball.net/projects/markdown/), convert it to HTML and paste that HTML into their blog of choice and then in the blog's editor clean it up to suit their blog.   This is an excellent way to create easy to read portable documents that can easily be published in multiple formats.

However what if there was a way to skip the second part of that and just create a markdown page, submit it into your source control ( you *do* use source control right? ) and your blog would automagically update.    

In comes [Octopress](http://octopress.org/),  it's a framework that wraps around [Jekyll](https://help.github.com/articles/using-jekyll-with-pages) which is [Github's](https://github.com/) blogging engine that powers [Github Pages](http://pages.github.com/).   Essentially you edit Markdown files and [Octopress](http://octopress.org/) will compile it into a static-html [Jekyll](https://help.github.com/articles/using-jekyll-with-pages) blog.     This means that your blog will be lightning fast ( no need to run an interpreted language in your web server ) and ultra portable.   

Another side benefit is that you can host it for free on [Github](https://github.com/) ( as long as you're okay with sharing your source ... and you should be! ) or for free on [Heroku](http://www.heroku.com/) ( don't have to share your source ) or host it on any simple no frills Apache, LightHTTP, nginx, node.js, etc server.

Here is how I'm porting my blogger site to Octopress hosted on Github Pages.   I'm not using any of the fancy [Jekyll migration tools](https://github.com/mojombo/jekyll/wiki/blog-migrations) as I only have a few posts and it will help me get used to the extended syntax that Octopress uses in Markdown.

<!--more-->

As usual the first step is to install any dependencies.   These instructions are for [Ubuntu 12.10](http://www.ubuntu.com/) ... modify to suit your OS of choice.

Most of these steps are taken directly from the [Octopress Documentation](http://octopress.org/docs/),   I'm just condensing them into a single document to suit the exact scenario being described in this post.

## Before You Begin

1. Install Git 
2. Install Ruby 1.9.3 via your OS package management or [rbenv](http://octopress.org/docs/setup/rbenv/) or [RVM](http://octopress.org/docs/setup/rvm/).    
*If using package management may need to install ruby-dev*

Check your Ruby version is at least 1.9.3 and install bundler:

```
ruby --version 
sudo gem install bundler
```

## Initial Setup 

Clone the octopress repository and set it up

```
git clone git://github.com/imathis/octopress.git octopress
cd octopress
bundle install

rake install
```

We're going to use Github pages.   Octopress has some rake tasks to make this easier for you.    Your blog will be hosted at `http://username.github.com` and you need to create a [new Github repository](https://github.com/repositories/new) called `username.github.com` that github pages will use the master branch as the html source for your blog.


```
rake setup_github_pages
```

This rake points our clone to the new repistory we just set up, configures your blog's URL and sets up a master branch in the `_deploy` directory for deployment.    

edit `_config.yml` and fill in your blog name and other details.   There's also some configs for twitter/G+/etc plugins that are worth configuring.

## Write some blog content
*Great time to read [Blogging Basics](http://octopress.org/docs/blogging)*

Create an `About` page and a `First Post!` blog post:

```
rake new_page["About"]
rake new_post["First Post!"]
```

Edit the Markdown pages that it creates for you with your preferred [Markdown editor](http://sourceforge.net/p/retext/home/ReText/).   The output of the rake commands should provide appropriate hints as to the location of the created files.



Generate and preview the blog:

```
rake generate
rake preview
```

This will generate the contents of your blog and allow you to preview it at [http://localhost:4000].

Once you're happy with the contents we can deploy your blog for the first time.

```
rake deploy
```
This will copy the generated files into `_deploy/`, add them to git, commit and push them up to the master branch. In a few seconds you should get an email from Github telling you that your commit has been received and will be published on your site.   Being your first commit it could take 10 minutes for the blog to be available at [http://username.github.com]

Don't forget to commit your changes to the source branch:

```
git add .
git commit -m 'Added About page and first post!'
git push origin source
```

## Want to edit your blog from another machine,  or edit an existing octopress blog?

This is pretty simple ( assuming you have the prerequisites already install ).  

*If you run Dropbox you can do this inside of your dropbox folders to make this instantly avaiable on any system you use.*

```
git clone git@github.com:username/username.github.com.git
cd username.github.com
git checkout source
mkdir _deploy
cd _deploy
git init
git remote add origin git@github.com:username/username.github.com.git
git pull origin master
cd ..
```

once this is done you can run `rake new_post["title"]` and all the other rake commands needed to edit/preview/publish your blog.