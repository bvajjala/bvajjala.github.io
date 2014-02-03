---
layout:     post
title:      "Groovy regular expressions"
date:       2009-09-25 12:00:00
categories: [Groovy,Regex]
tags:       [groovy,regex]
published:  true
---

Because of the compact syntax regular expressions in Groovy are more readable than in Java. Here is how Jeffrey Friedl's [example][1] looks in Groovy:

{% codeblock lang:groovy %}
def subDomain  = '(?i:[a-z0-9]|[a-z0-9][-a-z0-9]*[a-z0-9])' // simple regex in single quotes
def topDomains = $/
    (?x-i : com         \b     # you can put whitespaces and comments
          | edu         \b     # inside regex in eXtended mode
          | biz         \b
          | in(?:t|fo)  \b     # backslash is not escaped
          | mil         \b     # in dollar-slash strings
          | net         \b
          | org         \b
          | [a-z][a-z]  \b
    )/$

def hostname = /(?:${subDomain}\.)+${topDomains}/  // variable substitution in slashy string

def NOT_IN   = /;\"'<>()\[\]{}\s\x7F-\xFF/     // backslash is not escaped in slashy strings
def NOT_END  = /!.,?/
def ANYWHERE = /[^${NOT_IN}${NOT_END}]/
def EMBEDDED = /[$NOT_END]/                        // you can ommit {} around var name

def urlPath  = "/$ANYWHERE*($EMBEDDED+$ANYWHERE+)*"

def url =
    """(?x:
             # you have to escape backslash in multi-line double quotes
             \\b

             # match the hostname part
             (
               (?: ftp | http s? ): // [-\\w]+(\\.\\w[-\\w]*)+
             |
               $hostname
             )

             # allow optional port
             (?: :\\d+ )?

             # rest of url is optional, and begins with /
             (?: $urlPath )?
       )"""

assert 'http://www.google.com/search?rls=en&q=regex&ie=UTF-8&oe=UTF-8' ==~ url
assert 'pages.github.io' ==~ url
{% endcodeblock %}

As you can see, there are several notations, and for every subexpression you can choose the one that is most expressive.

### Resources

- Martin Fowler on [composed regexes][2]
- [Introducing dollar-slash][3] notation into Groovy
- [Mastering Regular Expressions][4] — the best regex book
- Groovy [Pattern][5] and [Matcher][6] classes


[1]: http://regex.info/listing.cgi?ed=3&p=208
[2]: http://martinfowler.com/bliki/ComposedRegex.html
[3]: http://jira.codehaus.org/browse/GROOVY-2701
[4]: http://regex.info/
[5]: http://mrhaki.blogspot.com/2009/09/groovy-goodness-using-regular.html
[6]: http://mrhaki.blogspot.com/2009/09/groovy-goodness-matchers-for-regular.html
