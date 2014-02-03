---
layout:     post
title:      "Reversing Groovy switch statement"
date:       2011-06-01 12:00:00
categories: [Groovy]
tags:       [groovy]
published:  true
---

Recently I've been working on a Groovy code that had many methods with long multibranch conditionals like this:

    def parse(message, options) {
        if (options.contains('A')) {
            parseARule message
        } else if (options.contains(2)) {
            parseSmallDigitRule message
        ...
        } else if (options.contains(something)) {
            parseSomeRule message
        } else {
            parseSomeOtherRule message
        }
    }

Although this code is working, it is hard to see which branch is called under which condition. It would be much better if we could replace this code with something like Lisp `cond` macro. The best candidate for such a task in Groovy would be a `switch` statement. If we could only refactor the code above to something like following, it would significantly improve readability:

    def parse(message, options) {
        switch (options) {
            case 'A' : return parseARule(message)
            case 2   : return parseSmallDigitRule(message)
            ...
            case ... : return parseSomeRule(message)
            default  : return parseSomeOtherRule(message)
        }
    }

Unfortunately, this code doesn't work out of the box in Groovy, but it works if we do some metaprogramming.

The way `switch` statement works in Groovy is a bit [different][1] than in Java. Instead of equals() it uses isCase() method to match case-value and switch-value. The default implementation of isCase() method falls back to equals() method, but some classes, including [Collection][2], override this behaviour. That's why in Groovy you can do things like this:

    switch (value) {
        case ['A','E','I','O','U'] : return 'vowel'
        case 0..9                  : return 'digit'
        case Date                  : return 'date'
        default                    : return 'something else'
    }

For our purposes we need some sort of reverse `switch`, where collection is used as a switch-value, and String and Integer are used as a case-value. To do this we need to override default implementation of isCase() method on String and Integer classes. It's not possible in Java, but is very easy in Groovy. You can change method implementation globally by replacing it in corresponding meta class, or locally with the help of categories. Let's create a category that swaps object and subject of isCase() method:

    class CaseCategory {
        static boolean isCase(String string, Collection col) {
            reverseCase(string, col)
        }
        static boolean isCase(Integer integer, Collection col) {
            reverseCase(integer, col)
        }
        // Add more overloaded methods here if needed
    
        private static boolean reverseCase(left, right) {
            right.isCase(left)
        }
    }

Now we can use this category to achieve the goal we stated at the beginning of this post:

    def parse(message, options) {
        use (CaseCategory) {
            switch (options) {
                case 'A' : return parseARule(message)
                case 2   : return parseSmallDigitRule(message)
                ...
                case ... : return parseSomeRule(message)
                default  : return parseSomeOtherRule(message)
            }
        }
    }

If you are comfortable with global method replacement, you can amend String and Integer meta classes. In this case you don't need to wrap `switch` statement with `use` keyword.


[1]: http://docs.codehaus.org/display/GROOVY/Logical+Branching#LogicalBranching-switchstatement
[2]: http://groovy.codehaus.org/groovy-jdk/java/util/Collection.html#isCase(java.lang.Object)
