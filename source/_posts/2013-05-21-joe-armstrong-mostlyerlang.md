---
layout:     post
title:      "Joe Armstrong at MostlyErlang"
date:       2013-05-21 19:13:01
categories: [Erlang,Programming]
tags:       [erlang]
published:  true
---

The main guest at today's [MostlyErlang](http://mostlyerlang.wordpress.com/2013/05/21/43/) podcast is Joe Armstrong, the co-inventor of Erlang, a wise man, and a brilliant speaker. I listened to the podcast few times, and every time enjoyed it. I highly recommend it to all programmers regardless whether they are using Erlang or not.

While we are waiting for the oficial transcript, this post provides some highlights. I put here the quotes I found particularly interesting or funny.

## Concurrency

> I've got a blog on my GitHub, and I thouhgt I'd put two different types of articles there and see what the response is. One theme I'm pursuing is "explaining to 5-year-old" theme, and the other is "technically complicated" theme.

Then he describes the "[concurrency vs parallelism](http://joearms.github.io/2013/04/05/concurrent-and-parallel-programming.html)" picture for 5 y.o. 

> That blog entry got 25,000 people read it, and they got 10 responses and quite a lot of criticism actually from people who know what concurrency and parallelism is. And then I compare it to some other articles I've written that technically deeper. There you got only few hundred people read it.

I noticed it too. All those people, who responded with [long blogs](http://www.yosefk.com/blog/parallelism-and-concurrency-need-different-tools.html) about concurrency, missed the point: This picture was for kids, and for kids it's a pretty good picture. Adults should go and read the technical papers, but they didn't.

<!-- more -->

## Let it crash

> Once upon a time we had a Leadership Election algorithm in Erlang. We got some theoreticians involved, and they proved the algorithm to be correct. It's been in practice for some time when we sent them a bug report, which showed that the thing elected two leaders, and we can reproduce it. And then they looked at this bug and went back at the code, and said the code is correct, but we had to make certain assumptions that this failure couldn't happen.

I like it. We should always keep it in mind: Before you prove anything, make sure, your assumptions are valid.

## Favourite bad idea

> The first bad idea we put in Erlang is the process priorities. I don't think it's documented anywhere. It's definitely not in my book. It's not supposed to be documented — that's deliberate.

[Erlang Programming](http://www.erlangprogramming.org/) mentions the process priorities on page 113, but they wrote the whole paragraph explaining why it's a bad idea.

> I was forced into position where I had to put them in [the language]. So I waited a few weeks and I put in an Erlang code that sets process priorities some integer, and this code didn't do anything at all. It was commented out. I shipped the system, and then I asked "How do you like it with priorities?" And they said "Now it's much better!"

Brilliant! Although, it pisses them off when people eventually find what's going on in the code.

## Performance

> One thing I like about Erlang is it's got pretty predictible performance model.

> There was some guy on the mailing list that said "I wrote a code that finds line breaks in the large file, and the result surprised me. I expected it to take this long, and it took a heck of a lot longer." I wrote a little benchmark and I couldn't see that effect. I mailed back "mine didn't behave like that." Then he said "There is another funny thing: the distribution of my file is not uniformed. I've got extremly long lines in it, and the rest are the same." And from then it took Patrik five minutes to find a bug in the system. But if he hadn't made an observation "this thing surprises me" and didn't give this clue that it happened under these circumstances, that bug could be in the system for years. But people find performance thing and they don't tell us the fact they are surprised, so we can't debug it.

Lesson learned: If you [don't like][1] the performance of your Erlang code, don't be shy to speak up about it. Either you are doing something wrong or there is a bug in Erlang. Both things are fixable.

## Scalability

> [MME](http://www.ericsson.com/ourportfolio/products/sgsn-mme) is a thing in the backbone of mobile telephony network, it makes all mobile data work. And it's written in Erlang. Smartphones wouldn't be able to connect to the network at all without MME. Now, Ericsson's got the biggest market share in the world for mobile base stations, for we do 40% of the total world market of 3G and CDMA and 60% of the world market for LTE and 4G. This gives us about 50% of the world market. That means Erlang's controlling 50% of all smartphones world wide.

That's impressive, and at the same time it's sad because people don't know about it. People are talking about Twitter scale, Google scale, or other sorts of "web-scale", but they don't realize that most of the data Twitter and Google receive nowadays is comming through mobile network, which means most of Twitter and Google data is coming throug Erlang. Think about it.

On the other hand, those who see the power of Erlang get big competitive advantage.

> At Erlang user conference somebody from gaming company came to me and said "Erlang's great! We have it on our server, and we managed to get 3 million connections on it!" And he was overjoyed. It's fantastic. I said "Great! You got to tell everybody…" And he said "No, no." — "Why not?" — "Because we tell our competitiors we use Jigsaw."

> There is no commercial advantage from using Ruby on Rails or Java or anything else. You might go to the conferences and swap battle stories and swap tips. But if you actually found something which is bloody better, you are not going to tell anybody, are you?

Hm, maybe that's why I cannot find any software company in Toronto that's doing Erlang. Hey Toronto programmers, if you are programming in Erlang, please let me know — I won't tell your competitors, I promise.

## Broken software

> Most of my time as a programmer is spent fixing broken stuff that shouldn't be broken. Whenever I'm trying to do something, it doesn't work. I used to spend more than 50% of my time fixing trivial things, and the percentage of my time is increasing by year. If I look back 20 years ago, I din't spend 50% of my time fixing broken software. The software that was available was a lot lot simpler, and it was written by engineers and teams and by small group of people, and it didn't have errors in it. If we extrapolate this in 20 years time, it's going to be even more software, and 90% of it will be completely broken, unless we find better ways of structuring it and gluing things together.

So true. I have the same statistics. One third of my time I'm shaving software yaks, and another third I'm debugging protocols trying to figure out why my code is not working with another code.

## Conference driven development

> I'm giving a keynote at LambdaJam. I've written a [flushing title](http://lambdajam.com/sessions#armstrong) but I haven't written the talk yet.

Lesson learned: Next time you want to give a talk at a conference, come up with a catchy title, and think about the talk itself later.

> I deliberately give myself titles for stuff I know nothing about, and I implement it all. You know, Robert did that for Lua. You know why Lua got developed in Erlang? Robert was going to give a talk on writing DSL in Erlang. I remember him talking to me "I'm giving a talk on implementing DSL in Erlang." I said "Oh, yeah." And he said "I don't know anything about it. I better implement Lua."

It's a really good idea to submit a conference proposal for the stuff you want to learn. When the proposal is accepted, you don't have a choice but go and study the thing. Otherwise you would procrastinate forever. I notice actually many people do that. Maybe I'll do the same with Erlang, at some point.


### Additional Resources

- Joe Armstrong's [reply](https://groups.google.com/forum/#!msg/erlang-programming/OiyGQ4UHqxw/HgGma01CGqYJ) about efficiency of Erlang.


[1]: http://code.mixpanel.com/2011/08/05/how-and-why-we-switched-from-erlang-to-python/
