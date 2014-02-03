---
layout:     post
title:      "RabbitMQ, ActiveMQ, ZeroMQ, HornetQ"
date:       2012-12-15 12:00:00
categories: [ActiveMQ,MongoDB,Python,RabbitMQ,ZeroMQ]
tags:       [activemq,mongodb,python,rabbitmq,stackoverflow,zeromq]
published:  true
---

*Warning*: In this post I'm going to compare [RabbitMQ](http://www.rabbitmq.com/), [ZeroMQ](http://www.zeromq.org/), [ActiveMQ](http://activemq.apache.org/), and [HornetQ](http://www.jboss.org/hornetq). The basis of the comparison is not the performance, or the scalability, or any other serious feature. The comparison is done *purely* based on the popularity of those systems. Therefore, if you came here to see some performance metrics, you will be disappointed — there is none in this post.

*Note*: To calculate popularity, I'm going to use MongoDB and Python, so if you don't care about message brokers, but you want to see some examples of MongoDB scripts, this post might be interesting to you.

<!-- more -->

## Popularity

What is the best messaging system out there? If you read my blog regularly, you probably know my biased answer. But to give an objective answer, we have to compare the candidates based on some criteria. There are many possible criteria, some of which are more relevant to your project than the others. One of them is how popular the candidate solutions are. In other words, if you choose a message broker and then you encounter a problem, how easy would it be to solve it? Is there anybody who can help you? One way to find it out is to check how many people are interested in the same solution. And the obvious way to do it is to ask Google.

Here is the Google trend graph for the last five years. It turns out, my personal preferences coincide with the public interest.

<div class="separator" style="clear: both; text-align: center;"><script type="text/javascript" src="//www.google.ca/trends/embed.js?hl=en-US&q=activemq,+rabbitmq,+zeromq,+hornetq&date=1/2008+60m&cmpt=q&content=1&cid=TIMESERIES_GRAPH_AVERAGES_CHART&export=5&w=800&h=330"></script></div>

At this point I can stop and say "Well, you see who's the winner". There are 5 times more people interested in RabbitMQ than HornetQ, so if you bet on Rabbit you have more chances to get the help from your fellow programmers, if you need to.

But before we make the final decision, I want to hear another opinion about the popularity of our candidates. Where do people go nowadays when they have software related problems? Right, they go to…

## StackOverflow

The best thing about StackOverflow is their [REST API](http://api.stackoverflow.com/1.0/usage). For our purposes we need two API queries: get all [questions by a tag](http://api.stackoverflow.com/1.1/usage/methods/search), and get all [answers](http://api.stackoverflow.com/1.1/usage/methods/question-answers) for the question. In fact, the second one is optional. Even the first query alone can give us most of what we want to know:

- how many questions have been posted for every candidate on our list?
- how many answers did those questions receive?
- how many answers were accepted?
- how many questions and answers were marked useful?

When we get all the numbers, we should know what people are actually using. We can also check if there is any correlation between Google data and StackOverflow.

So how do we proceed? We cannot use API directly to run analytics, because we would quickly exhaust the daily quota. What we can do is to fetch the data, save it locally, and run analytics against the local data. Here is another good thing about StackOverflow API: it comes in JSON format. What is the best way to analyze JSON data? Obviously, saving it in a JSON-oriented database that supports aggregated queries. And that's where MongoDB comes into play.

[Here](https://gist.github.com/4276136#file-questions-py) is the Python script that downloads all the questions for the specified tags from StackOverflow, and saves the results in the local MongoDB instance. I chose Python because I want to draw some graphs later, which is easy to do in Python. Plus, it's a simple and expressive language.

After we run this script, we get all the questions we need in our database. The next step is to get all the answers for those questions. [Here](https://gist.github.com/4276136#file-answers-py) is the script that does exactly that.

Depends on how many questions we have saved on the first step, there might be quite a lot of queries to run to get all the answers. With my second script I exceeded the daily quota, so I had to wait for the next day to get the rest of the answers.

Now, when we have all the data, let's take a look how we can use it. Here is a typical StackOverflow record. I highlighted the fields that might be useful for our analysis.

<pre>{
     "_id" : 269363,
     <font color="#0000FF">"accepted_answer_id"</font> : 290764,
     <font color="#0000FF">"answer_count" : 4</font>,
     "answers" : [
          ...
          {
               "view_count" : 0,
               "answer_comments_url" : "/answers/303710/comments",
               "answer_id" : 303710,
               "title" : "ActiveMQ .net client locks up",
               "community_owned" : false,
               "down_vote_count" : 0,
               "last_activity_date" : 1317300099,
               "creation_date" : 1227135282,
               "score" : 1,
               <font color="#0000FF">"up_vote_count" : 1</font>,
               "owner" : {
                    "display_name" : "HitLikeAHammer",
                    <font color="#0000FF">"reputation" : 1152</font>,
                    "user_id" : 35165,
                    "user_type" : "registered",
                    "email_hash" : "584cd9905db85f744e7e96740b11b7c0"
               },
               "accepted" : false,
               "last_edit_date" : 1317300099,
               "question_id" : 269363
          },
          ...
     ],
     "community_owned" : false,
     "creation_date" : 1225989513,
     "down_vote_count" : 0,
     "favorite_count" : 1,
     "last_activity_date" : 1317300112,
     "owner" : {
          "display_name" : "HitLikeAHammer",
          "reputation" : 1152,
          "user_id" : 35165,
          "user_type" : "registered",
          "email_hash" : "584cd9905db85f744e7e96740b11b7c0"
     },
     "question_answers_url" : "/questions/269363/answers",
     "question_comments_url" : "/questions/269363/comments",
     "question_id" : 269363,
     "question_timeline_url" : "/questions/269363/timeline",
     "score" : 1,
     <font color="#0000FF">"tags" : [
          ".net",
          "activemq"
     ]</font>,
     "title" : "ActiveMQ .net client locks up",
     <font color="#0000FF">"up_vote_count" : 1</font>,
     "view_count" : 1183
}</pre>

First of all, we want to know how many questions are posted for each messaging system from our list. Here is the MongoDB query for that. The first snippet is a query, the second is the result.

{% codeblock Number of questions lang:javascript %}
db.stackoverflow.aggregate([
     {$unwind:'$tags'},
     {$group:{_id:'$tags', questions:{$sum:1}}},
     {$match:{_id:{$in:['activemq', 'rabbitmq', 'zeromq', 'hornetq']}}},
     {$sort:{questions:-1}}
])['result'];
{% endcodeblock %}

{% codeblock lang:javascript %}
[
     {
          "_id" : "activemq",
          "questions" : 1039
     },
     {
          "_id" : "rabbitmq",
          "questions" : 988
     },
     {
          "_id" : "zeromq",
          "questions" : 373
     },
     {
          "_id" : "hornetq",
          "questions" : 185
     }
]
{% endcodeblock %}

The next query is to get the total number of answers by tag

{% codeblock Number of answers lang:javascript %}
db.stackoverflow.aggregate([
     {$unwind:'$tags'},
     {$group:{_id:'$tags', answers:{$sum:'$answer_count'}}},
     {$match:{_id:{$in:['activemq', 'rabbitmq', 'zeromq', 'hornetq']}}},
     {$sort:{answers:-1}}
])['result'];
{% endcodeblock %}

{% codeblock lang:javascript %}
[
     {
          "_id" : "activemq",
          "answers" : 1382
     },
     {
          "_id" : "rabbitmq",
          "answers" : 1322
     },
     {
          "_id" : "zeromq",
          "answers" : 572
     },
     {
          "_id" : "hornetq",
          "answers" : 227
     }
]
{% endcodeblock %}

It seems that the number of answers is proportional to the number of questions. With MongoDB we can quickly verify it.

{% codeblock Answers to questions ratio lang:javascript %}
db.stackoverflow.aggregate([
     {$unwind:'$tags'},
     {$group:{_id:'$tags', answers:{$sum:'$answer_count'}, questions:{$sum:1}}},
     {$match:{_id:{$in:['activemq', 'rabbitmq', 'zeromq', 'hornetq']}}},
     {$project:{answers:1, questions:1, ratio:{$divide:['$answers', '$questions']}}},
     {$sort:{ratio:-1}}
])['result'];
{% endcodeblock %}

{% codeblock lang:javascript %}
[
     {
          "_id" : "zeromq",
          "answers" : 572,
          "questions" : 373,
          "ratio" : 1.5335120643431635
     },
     {
          "_id" : "rabbitmq",
          "answers" : 1322,
          "questions" : 988,
          "ratio" : 1.3380566801619433
     },
     {
          "_id" : "activemq",
          "answers" : 1382,
          "questions" : 1039,
          "ratio" : 1.3301251203079885
     },
     {
          "_id" : "hornetq",
          "answers" : 227,
          "questions" : 185,
          "ratio" : 1.227027027027027
     }
]
{% endcodeblock %}

Indeed, the answers/question ratio is almost the same for every tag. That means we can use just the number of questions for our analysis.

Here is the query that calculates the number of *accepted* answers by tag. Again, it correlates fairly well with the total number of answers and questions.

{% codeblock Number of accepted answers lang:javascript %}
db.stackoverflow.aggregate([
     {$match:{accepted_answer_id:{$ne:null}}},
     {$unwind:'$tags'},
     {$group:{_id:'$tags', accepted_answers:{$sum:1}}},
     {$match:{_id:{$in:['activemq', 'rabbitmq', 'zeromq', 'hornetq']}}},
     {$sort:{accepted_answers:-1}}
])['result'];
{% endcodeblock %}

{% codeblock lang:javascript %}
[
     {
          "_id" : "activemq",
          "accepted_answers" : 531
     },
     {
          "_id" : "rabbitmq",
          "accepted_answers" : 500
     },
     {
          "_id" : "zeromq",
          "accepted_answers" : 221
     },
     {
          "_id" : "hornetq",
          "accepted_answers" : 94
     }
]
{% endcodeblock %}

The next query is more interesting. It calculates the number of question up-votes by tag. In other words, it shows the number of *useful* questions. If we divide it by the total number of questions, we should see which messaging system has bigger rate of useful questions than others

{% codeblock Number of useful questions lang:javascript %}
db.stackoverflow.aggregate([
     {$unwind:'$tags'},
     {$group:{_id:'$tags', upvotes:{$sum:'$up_vote_count'}, questions:{$sum:1}}},
     {$match:{_id:{$in:['activemq', 'rabbitmq', 'zeromq', 'hornetq']}}},
     {$project:{upvotes:1, questions:1, ratio:{$divide:['$upvotes', '$questions']}}},
     {$sort:{ratio:-1}}
])['result'];
{% endcodeblock %}

{% codeblock lang:javascript %}
[
     {
          "_id" : "zeromq",
          "upvotes" : 1078,
          "questions" : 373,
          "ratio" : 2.8900804289544237
     },
     {
          "_id" : "rabbitmq",
          "upvotes" : 1864,
          "questions" : 988,
          "ratio" : 1.8866396761133604
     },
     {
          "_id" : "activemq",
          "upvotes" : 1459,
          "questions" : 1039,
          "ratio" : 1.4042348411934553
     },
     {
          "_id" : "hornetq",
          "upvotes" : 233,
          "questions" : 185,
          "ratio" : 1.2594594594594595
     }
]
{% endcodeblock %}

Interesting. The ZeroMQ users seem to ask more useful questions than the users of other brokers.

Let's do the same analysis for the answers. Here is the query that calculates the number of answer up-votes by tag.

{% codeblock Number of useful answers lang:javascript %}
db.stackoverflow.aggregate([
     {$unwind:'$answers'},
     {$unwind:'$tags'},
     {$group:{_id:{question:'$_id', tag:'$tags'}, upvotes:{$sum:'$answers.up_vote_count'}}},
     {$group:{_id:'$_id.tag', upvotes:{$sum:'$upvotes'}, questions:{$sum:1}}},
     {$match:{_id:{$in:['activemq', 'rabbitmq', 'zeromq', 'hornetq']}}},
     {$project:{upvotes:1, questions:1, ratio:{$divide:['$upvotes', '$questions']}}},
     {$sort:{ratio:-1}}
])['result'];
{% endcodeblock %}

{% codeblock lang:javascript %}
[
     {
          "_id" : "zeromq",
          "upvotes" : 1469,
          "questions" : 338,
          "ratio" : 4.346153846153846
     },
     {
          "_id" : "rabbitmq",
          "upvotes" : 2437,
          "questions" : 858,
          "ratio" : 2.84032634032634
     },
     {
          "_id" : "activemq",
          "upvotes" : 2199,
          "questions" : 902,
          "ratio" : 2.4379157427937916
     },
     {
          "_id" : "hornetq",
          "upvotes" : 262,
          "questions" : 156,
          "ratio" : 1.6794871794871795
     }
]
{% endcodeblock %}

Again, ZeroMQ users post more useful answers than others.

To complete the picture of typical users, let's run the following query that calculates an average reputation of people that post answers

{% codeblock Reputation of respondents lang:javascript %}
db.stackoverflow.aggregate([
     {$unwind:'$answers'},
     {$unwind:'$tags'},
     {$group:{_id:{question:'$_id', tag:'$tags'}, reputation:{$avg:'$answers.owner.reputation'}}},
     {$group:{_id:'$_id.tag', reputation:{$avg:'$reputation'}}},
     {$match:{_id:{$in:['activemq', 'rabbitmq', 'zeromq', 'hornetq']}}},
     {$sort:{reputation:-1}}
])['result'];
{% endcodeblock %}

{% codeblock lang:javascript %}
[
     {
          "_id" : "zeromq",
          "reputation" : 10088.29552338687
     },
     {
          "_id" : "activemq",
          "reputation" : 7298.7539383380845
     },
     {
          "_id" : "rabbitmq",
          "reputation" : 6082.172231934734
     },
     {
          "_id" : "hornetq",
          "reputation" : 3472.9658119658116
     }
]
{% endcodeblock %}


Wow. ZeroMQ users not only ask more useful questions and give useful answers, they also have higher reputation on average in the StackOverflow community.

As a final exercise, I want to build a graph of question distribution over time. After all, ActiveMQ is the oldest broker, and it might have got more questions just because it was launched first. For this purpose I created [this](https://gist.github.com/4276136#file-trends-py) Python script that uses amazing [matplotlib](http://matplotlib.org/gallery.html) library. And here is the result for the last 60 months

{% img center /images/posts/mom-trends-so.png %}

It shows that the proportion of interest in different massaging systems was approximately the same all the time. Furthermore, the StackOverflow statistics of this year correlates well with the Google statistics.

## Conclusion

1. RabbitMQ and ActiveMQ are very popular. If you choose one of them for your messaging infrastructure, you shouldn't have any problem with the community support. HornetQ might be a good message broker but it definitely lacks the community interest. Finally, as I [suspected](/2012/11/12/pycon-canada-2012/#learn) before, ZeroMQ is worth looking at. There are bunch of smart and helpful people in ZeroMQ community.

2. MongoDB is really nice! Its aggregation framework is powerful and easy to use. It was fun playing with it.
