---
layout:     post
title:      "Functional Groovy switch statement"
date:       2011-06-08 12:00:00
categories: [Groovy]
tags:       [groovy]
published:  true
---

In the previous [post][1] I showed how to replace chained if-else statements in Groovy with one concise switch. It was done for the special case of if-stement where every branch was evaluated using the same condition function. Today I want to make a generalization of that technique by allowing to use different conditionals.

Suppose your code looks like this:

    if (param % 2 == 0) {
        'even'
    } else if (param % 3 == 0) {
        'threeven'
    } else if (0 < param) {
        'positive'
    } else {
        'negative'
    }

As long as every condition operates on the same parameter, you can replace the entire chain with a switch. In this scenario `param` becomes a switch parameter and conditions become `case` parameters of Closure type. The only thing we need to do is to override `Closure.isCase()` method as I described in the previous post. The safest way to do it is to create a category class:

    class CaseCategory {
        static boolean isCase(Closure casePredicate, Object switchParameter) {
            casePredicate.call switchParameter
        }
    }

Now we can replace if-statement with the following switch:

    use (CaseCategory) {
        switch (param) {
            case { it % 2 == 0 } : return 'even'
            case { it % 3 == 0 } : return 'threeven'
            case { 0 < it }      : return 'positive'
            default              : return 'negative'
        }
    }

We can actually go further and extract in-line closures:

    def even = {
        it % 2 == 0
    }
    def threeven = {
        it % 3 == 0
    }
    def positive = {
        0 < it
    }

After which the code becomes even more readable:

    use (CaseCategory) {
        switch (param) {
            case even     : return 'even'
            case threeven : return 'threeven'
            case positive : return 'positive'
            default       : return 'negative'
        }
    }

[1]: /2011/06/01/reversing-groovy-switch-statement
