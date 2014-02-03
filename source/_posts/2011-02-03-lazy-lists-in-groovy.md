---
layout:     post
title:      "Lazy lists in Groovy"
date:       2011-02-03 12:00:00
categories: [Functional Programming,Groovy]
tags:       [fp,groovy,haskell,lisp]
published:  true
---

I like lazy evaluation, and it's one of the reasons I like Haskell and Clojure. Although from engineering perspective lazy evaluation is probably not the most needed feature, it's definitely very useful for solving some mathematical problems.

Most languages don't have lazy evaluation out of the box, but you can implement it using some other language features. This is an interesting task, and I use it as a code [kata][1] which I practice every time I learn a new strict language.

So, how to implement lazy lists in a strict language? Very simple, if the language is functional. Namely, you *build lazy list recursively by wrapping strict list within a function*. Here is, for example, the strict empty list in Groovy:

{% codeblock lang:groovy %}
[]
{% endcodeblock %}

If we wrap it with a closure, it becomes lazy empty list:

{% codeblock lang:groovy %}
{-> [] }
{% endcodeblock %}

If we need a list with one element, we prepend (or speaking Lisp terminology *cons*) an element to lazy empty list, and make the result lazy again:

{% codeblock lang:groovy %}
{-> [ element, {-> [] } ] }
{% endcodeblock %}

To add more elements we continue the same process until all elements are lazily consed. Here is, for example, a lazy list with three elements *a*, *b* and *c*:

{% codeblock lang:groovy %}
{-> [a, {-> [b, {-> [ c, {-> [] } ] } ] } ] }
{% endcodeblock %}

Now, when you have an idea how to build lazy lists, let's build them Groovy way. We start by creating a class:

{% codeblock LazyList.groovy lang:groovy %}
class LazyList {
    private Closure list

    private LazyList(list) {
        this.list = list
    }
}
{% endcodeblock %}

The variable `list` encapsulates the closure wrapper of the list. We need to expose some methods that allow constructing lists using procedure described above:

{% codeblock LazyList.groovy (cont'd) lang:groovy %}
    static LazyList nil() {
        new LazyList( {-> []} )
    }

    LazyList cons(head) {
        new LazyList( {-> [head, list]} )
    }
{% endcodeblock %}

Now we can construct lists by consing elements to empty list:

{% codeblock lang:groovy %}
def lazylist = LazyList.nil().cons(4).cons(3).cons(2).cons(1)
{% endcodeblock %}

To access elements of the list we implement two standard functions, `car` and `cdr`, that return head and tail of the list respectively.

{% codeblock LazyList.groovy (cont'd) lang:groovy %}
    def car() {
        def lst = list.call()
        lst ? lst[0] : null
    }

    def cdr() {
        def lst = list.call()
        lst ? new LazyList(lst[1]) : nil()
    }
{% endcodeblock %}

Here is how you use these functions to get first and second elements of the list constructed above

{% codeblock lang:groovy %}
assert lazylist.car() == 1
assert lazylist.cdr().car() == 2
{% endcodeblock %}

In Lisp there are built-in functions for various `car` and `cdr` compositions. For example, the previous assertion would be equivalent to function `cadr`. Instead of implementing all possible permutations, let's use Groovy metaprogramming to achieve the same goal.

{% codeblock LazyList.groovy (cont'd) lang:groovy %}
    def methodMissing(String name, args) {
        def matcher = name =~ /^c([ad]+)r$/
        if (matcher) {
            matcher[0][1].reverse().toList().inject(this) {
                del, cr -> del."c${cr}r"()
            }
        } else {
            throw new MissingMethodException(name, this.class, args)
        }
    }
{% endcodeblock %}

It might look complicated, but in reality it's pretty simple if you are familiar with Groovy regex and functional programming. It's easier to explain by example. If we pass "caddr" as a value of `name` parameter, the method will create a chain on method calls `.cdr().cdr().car()` which will be applied to delegate of the operation which is our LazyList object.

With this method in place we can call car/cdr functions with arbitrary depth.

{% codeblock lang:groovy %}
assert lazylist.caddr() == 3
{% endcodeblock %}

If you create nested lazy lists, you can access any element of any nested list with this dynamic method.

{% codeblock lang:groovy %}
def lmn = LazyList.nil().cons('N').cons('M').cons('L')
def almnz = LazyList.nil().cons('Z').cons(lmn).cons('A')
assert almnz.cadadr() == 'M'
{% endcodeblock %}

With so many cons methods it's hard to see the structure of the list. Let's implement `lazy` method on ArrayList class that converts strict list to lazy. Again, we will use metaprogramming and functional techniques.

{% codeblock lang:groovy %}
ArrayList.metaClass.lazy = {
    -> delegate.reverse().inject(LazyList.nil()) {list, item -> list.cons(item)}
}
{% endcodeblock %}

Now we can rewrite the previous example as follows

{% codeblock lang:groovy %}
def lazyfied = ['A', ['L','M','N'].lazy(), 'Z'].lazy()
assert lazyfied.cadadr() == 'M'
{% endcodeblock %}

What have we accomplished so far? We learned how to build lazy lists from scratch and from strict lists. We know how to add elements to lazy lists, and how to access them. The next step is to implement `fold` function. `fold` is the [fundamental][2] operation in functional languages, so our lazy lists must provide it.

{% codeblock LazyList.groovy (cont'd) lang:groovy %}
    boolean isEmpty() {
        list.call() == []
    }

    def fold(n, acc, f) {
        n == 0 || isEmpty() ? acc : cdr().fold(n-1, f.call(acc, car()), f)
    }

    def foldAll(acc, f) {
        isEmpty() ? acc : cdr().foldAll(f.call(acc, car()), f)
    }
{% endcodeblock %}

The only difference between this `fold` function and the standard one is the additional parameter *n*. We will need it later when we implement infinite lists. `foldAll` function to lazy lists is the same as standard `fold` to strict lists.

{% codeblock lang:groovy %}
assert [1,2,3,4,5].lazy().foldAll(0){ acc, i -> acc + i } == 15
assert [1,2,3,4,5].lazy().fold(3, 1){ acc, i -> acc * i } == 6
{% endcodeblock %}

First example calculates the sum of all elements of the list, second calculates the product of first three elements.

If you have `fold` functions you can easily implement `take` functions

{% codeblock LazyList.groovy (cont'd) lang:groovy %}
    def take(n) {
        fold(n, []) {acc, item -> acc << item}
    }

    def takeAll() {
        foldAll([]) {acc, item -> acc << item}
    }
   
    def toList() {
        takeAll()
    }
{% endcodeblock %}

`take` is an inverse operation to `lazy`

{% codeblock lang:groovy %}
assert [1,2,3,4,5].lazy().takeAll() == [1,2,3,4,5]
assert [1,2,3,4,5].lazy().take(3) == [1,2,3]
{% endcodeblock %}

Our next goal is `map` function on lazy lists. Ideally I want the implementation look like this

{% codeblock lang:groovy %}
    def map(f) {
        isEmpty() ? nil() : cdr().map(f).cons(f.call(car()))
    }
{% endcodeblock %}

For some reason it doesn't work lazy way in Groovy &mdash; it's still strictly evaluated. Therefore I have to implement it directly with closure syntax

{% codeblock LazyList.groovy (cont'd) lang:groovy %}
    def map(f) {
        isEmpty() ? nil() : new LazyList( {-> [f.call(car()), cdr().map(f).list]} )
    }
{% endcodeblock %}

Unlike `fold`, lazy `map` is identical to strict `map`

{% codeblock lang:groovy %}
assert [1,2,3,4,5].lazy().map{ 2 * it }.take(3) == [2,4,6]
{% endcodeblock %}

The following example shows one of the benefits of laziness

{% codeblock lang:groovy %}
assert [1,2,3,0,6].lazy().map{ 6 / it }.take(3) == [6,3,2]
{% endcodeblock %}

`map` didn't evaluate the entire list, hence there was no exception. If you evaluate the expression for all the elements, the exception will be thrown 

{% codeblock lang:groovy %}
try {
    [1,2,3,0,6].lazy().map{ 6 / it }.takeAll()
}
catch (Exception e) {
    assert e instanceof ArithmeticException
}
{% endcodeblock %}

For strict lists this is a default behaviour of `map` function.

The last function I want to implement is `filter`

{% codeblock LazyList.groovy (cont'd) lang:groovy %}
    def filter(p) {
        isEmpty() ? nil() :
            p.call(car()) ? new LazyList( {-> [car(), cdr().filter(p).list]} ) :
                cdr().filter(p)
    }
{% endcodeblock %}

In the following example we find first two elements greater than 2

{% codeblock lang:groovy %}
assert [1,2,3,4,5].lazy().filter{ 2 < it }.take(2) == [3,4]
{% endcodeblock %}

With the help of `car`/`cdr`, `fold`, `map` and `filter` you can implement any other function on lazy lists yourself. Here is, for example, the implementation of `zipWith` function

{% codeblock LazyList.groovy (cont'd) lang:groovy %}
    static def zipWith(alist, blist, f) {
        alist.isEmpty() || blist.isEmpty() ? nil() :
            new LazyList( {-> [
                f.call(alist.car(), blist.car()),
                zipWith(alist.cdr(), blist.cdr(), f).list
            ]} )
    }
{% endcodeblock %}

Now, after we implemented all lazy functions we need, let's define infinite lists

{% codeblock LazyList.groovy (cont'd) lang:groovy %}
    private static sequence(int n) {
        {-> [n, sequence(n+1)]}
    }

    static LazyList integers(int n) {
        new LazyList(sequence(n))
    }

    static LazyList naturals() {
        integers(1)
    }
{% endcodeblock %}

Infinite lists, from my point of view, is the most useful application of lazy lists

{% codeblock lang:groovy %}
def naturals = LazyList.naturals()
assert naturals.take(3) == [1,2,3]

def evens = naturals.map { 2 * it }
assert evens.take(3) == [2,4,6]

def odds = naturals.filter { it % 2 == 1 }
assert odds.take(3) == [1,3,5]

assert naturals.cadddddddddr() == 10

def nonnegatives = naturals.cons(0)
assert nonnegatives.cadr() == 1

assert LazyList.zipWith(evens, odds){ x, y -> x * y }.take(4) == [2,12,30,56]
{% endcodeblock %}

At this point you have all basic functionality implemented, and you should be able to extend this model to whatever you need in regards to lazy (infinite) lists. Happy lazy programming!

## Resources and links

- [Source code](http://gist.github.com/810702) for this blog
- Lazy list implementation in [Erlang](https://github.com/ndpar/erlang/blob/master/src/lazy.erl)
- Lazy list implementation in [Lisp](https://github.com/ndpar/land-of-lisp/blob/master/lazy.lisp)


[1]: http://en.wikipedia.org/wiki/Kata_(programming)
[2]: http://www.cs.kent.ac.uk/people/staff/dat/miranda/whyfp90.pdf
