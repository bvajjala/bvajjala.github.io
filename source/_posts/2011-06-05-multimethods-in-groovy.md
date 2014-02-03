---
layout:     post
title:      "Multimethods in Groovy"
date:       2011-06-05 12:00:00
categories: [Groovy,Java]
tags:       [groovy,java]
published:  true
---

Every time I switch from Groovy to Java I have to remind myself that some things that seem so natural and work as expected in Groovy, don't work in Java. One of such differences is method dispatching. Groovy supports [multiple dispatch][1], while Java does not. Therefore the following [code][2] works differently in Groovy and Java:

    public class A {
        public void foo(A a) { System.out.println("A/A"); }
        public void foo(B b) { System.out.println("A/B"); }
    }
    public class B extends A {
        public void foo(A a) { System.out.println("B/A"); }
        public void foo(B b) { System.out.println("B/B"); }
    }
    public class Main {
        public static void main(String[] args) {
            A a = new A();
            A b = new B();
            a.foo(a);
            b.foo(b);
        }
    }

    $ java Main
    A/A
    B/A

    $ groovy Main.groovy
    A/A
    B/B

[1]: http://en.wikipedia.org/wiki/Multiple_dispatch
[2]: http://www.gigamonkeys.com/book/object-reorientation-generic-functions.html#multimethods
