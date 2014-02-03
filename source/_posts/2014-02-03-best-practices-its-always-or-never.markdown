---
layout: post
title: "Best Practices: It's Always or Never"
date: 2014-02-03 13:45:48 -0500
comments: true
categories: 
---

## [Best Practices: It's Always Or Never ( And Preferably Always)](/blog/2013/02/11/best-practices-it-s-always-or-never.html)

![Messy cables](http://farm3.staticflickr.com/2584/4103140420_b98ee1ac62_z.jpg)

It's Monday morning. The development team needs a box and you're already contemplating the gazillion other urgent tasks that need to be done on the existing infrastructure. _Just that one time_TM, you're going to forget about your own rules. You're just gonna spawn an instance, set up the few services needed and be done with it. You'll drop some of the usual time suckers: backup strategy, access rules, init scripts, documentation… You can't just do the whole of it AND handle the rest of your day-to-day responsibilities. After all, it's just a development server and you'll probably fold it in a couple weeks, or you'll clean it up once your plate is a tad less full.

A few weeks later, the box is still there and your backlog is far from looking less crowded. The development team just rolled out their production application on the same box. **And things start crashing… badly.**

After a couple of not so courteous emails from the dev team mentioning repetitive crashes, you log in the box and the fun starts. You can't figure out what services have been deployed, or how exactly they were installed. You can't restore the database because you don't know where the bloody backups are. You waste time to find out that CouchDB wasn't started at boot. All of this while receiving emails of "encouragement" from your colleagues.

Just because of that "one time". Except that it's never just that one time.

### Best practices are not freaking optional

I hear you: coming up with these best practices and sticking to it **systematically** is hard. It's high investment. But based on our common experience, it's one you can't afford not making. The "quick and dirty that
one time" approach will ultimately fail you.

A few things you should never consider skipping:

  * **Document the hell out of everything as you go**. You probably won't have time to get it done once you shipped it, and you probably won't remember what you did or why you did it in a few weeks from now. Your colleagues will probably appreciate too.

  * **Off-site backups for everything**. Don't even think of keeping your backups on the same physical box. Disks fail (a lot) and storage like S3/Glacier is dirt cheap. Find out a way to backup your code and data and stick to it.

  * **Full setup and reliable sources**. Avoid random AWS AMIs or RPM repositories. And when settings things up, go through the whole shebang: init script, dedicated running user, environment variables and such are not optional. Some of us also think that you shouldn't use rc.local for your Web services ever again.

### Infrastructure As Code And Automation

Obviously, given what we're working on at [devo.ps](http://devo.ps), we're pretty strong adopters of infrastructure as code and automation. What tools to use is a much larger discussion. Go have a look at the comments on [the
announcement of the new version of Chef](http://news.ycombinator.com/item?id=5197389) to get an idea of what's
out there.

Ultimately these are just opinions, but behind them are concepts worth investing in. Capturing the work you do on your infrastructure in repeatable and testable code, and automating as much as you can helps removing yourself
from the equation. Doing so is helping you to reduce the human factor and free yourself of the repetitive boilerplate while you focus on the challenging tasks that only a creative brain can tackle.

Not building upon best practices is simply not an option. By doing so, you fail at investing in the foundation for a more robust infrastructure, and more importantly it is depriving you from scaling yourself.

_[Picture from comedy_nose](http://www.flickr.com/photos/comedynose/4103140420/)_

