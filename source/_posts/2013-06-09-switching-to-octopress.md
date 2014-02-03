---
layout:     post
title:      "Switching to Octopress"
date:       2013-06-09 14:03:00
categories: [Blog]
tags:       [blog]
published:  false
---


I'm switching to different blogging platform. Again. There are few reasons for that. The main reason is I want to own the sources of my articles. All platforms I used before save blog posts in HTML format, which is not the source format but rather one of the representations. If I want to update an article I have to edit HTML in some online editor which is not the best way of editing. It's especially difficult on Blogspot, because after some time Blogspot converts the articles to a single-line of HTML which is very ugly.

I'd like to have my articles in a simple format like Markdown, and the blogging engine should just render them to HTML. I also like to keep the sources on my hard drive where I can quickly grep for phrases. Essentially what I want is this

- Separate the source of the articles from their representation.
- Being able to save sources locally on my disk and upload them if needed to the Internet.
- Keep the history of article changes.
- Have flexibility in representation, i.e. being able to change the layout of the blog to whatever I want.

Neither Blogspot nor the previous platforms I used to publish my articles to provide these features. Therefore I'm switching to Jekyll.

With [Jekyll][1] you write the article in Markdown, Textile, or Liquid, and it generates the static HTML which can be uploaded to any Jekyll-aware hosting. Github, for example, is one of them.

To get even more power, I'm going to use [Octopress][2], which is built on top of Jekyll. Apart from all Jekyll features, it also provides syntax highlighting (which is very handy when you post code snippets), professional mobile layout, and tons of plugins you can use to customize the blog.

As a part of the migration I also want to revisit my old articles and update the interesting ones. You will find them in the archives section of my new blog.

Without further ado, here is the link to my new blog: [blog.ndpar.com][3]. The sources can be found on [Github][4].

The [Blogspot][5] blog is now deprecated.


[1]: http://jekyllrb.com/
[2]: http://octopress.org/
[3]: http://blog.ndpar.com/
[4]: http://github.com/ndpar/ndpar.github.com/tree/source/source/_posts
[5]: http://ndpar.blogspot.com
