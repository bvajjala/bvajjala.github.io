---
layout: post
title: "Code Reuse With Node.js"
date: 2014-02-03 14:09:35 -0500
comments: true
categories: 
---

## [Code Reuse With Node.js](/blog/2013/07/11/code-reuse-with-nodejs.html)

![Code recycling](http://devo.ps/images/posts/recycle.png)

Any project that grows to a decent size will need to re-use parts of its code extensively. That often means, through the development cycle, a fair amount of rewrites and refactoring exercises. Elegant code re-use is hard to pull off.

With node.js, which we use quite a bit at [devo.ps](http://devo.ps), the most common ways to do this often rely on prototype or class inheritance. The problem is, as the inheritance chain grows, managing attributes and functions
can become quite complex.

The truth is, people usually just need the objects. This led us to adopt a certain form of object-based prototyping. We believe it to be leaner and more straightforward in most cases. But before we get there, let's have a look at how people usually approach this issue.

## The "Function copy"

Usually in the form of `this[key] = that[key]`. A quick example:

    
    var objectA = {
        lorem: 'lorem ipsum'
    };
    var objectB = {};
    
    // Direct copy of a string, but you get the idea
    objectB.lorem = objectA.lorem;
    console.log(objectB); // Will output: { lorem: 'lorem ipsum' }

Crude, but it works. Next…

## Object.defineProperties()

The previous method may work with simple structures, but it won't hold when
your use cases become more complex. That's when I usually call my buddy
`Object.defineProperties()`:

    
    var descriptor = Object.getOwnPropertyDescriptor;
    var defineProp = Object.defineProperty;
    
    var objectA = {};
    var objectB = {};
    var objectC = {};
    
    objectA.__defineGetter__('lorem', function() {
        return 'lorem ipsum';
    });
    console.log(objectA); // Will output: { lorem: [Getter] }
    
    // Direct copy, which copies the result of the getter.
    objectB.lorem = objectA.lorem;
    console.log(objectB); // Will output: { lorem: 'lorem ipsum' }
    
    // Copying with Object.defineProperty(), and it copies the getter itself.
    defineProp(objectC, 'lorem', descriptor(objectA, 'lorem'));
    console.log(objectC); // Will output: { lorem: [Getter] }

I often use a library for that. A couple examples (more or less the same stuff
with different coding styles):

  1. **[es5-ext](https://github.com/medikoo/es5-ext)**
    
     var extend = require('es5-ext/lib/Object/extend-properties');
     
     var objectA = {};
     var objectC = {};
     
     objectA.__defineGetter__('lorem', function() {
         return 'lorem ipsum';
     });
     
     extend(objectC, objectA);
     console.log(objectC); // Will output: { lorem: [Getter] }

  2. **[Carcass](https://github.com/devo-ps/carcass)**
    
     var carcass = require('carcass');
     
     var objectA = {};
     var objectC = {};
     
     objectA.__defineGetter__('lorem', function() {
         return 'lorem ipsum';
     });
     
     carcass.mixable(objectC);
     objectC.mixin(objectA);
     console.log(objectC); // Will output: { mixin: [Function: mixin], lorem: [Getter] }

Slightly better, but not optimal. Now, let's see what we end up doing more and
more often:

## Prototyping through objects

The basic idea is that we prepare some functions, wrap them into an object which then becomes a "feature". That feature can then be re-used by simply merging it with the targeted structure (object or prototype).

Let's take the example of the [loaderSync](https://github.com/devo-ps/carcass/blob/master/lib/proto/loaderSync.js) script in [Carcass](https://github.com/devo-ps/carcass):

    
    module.exports = {
        source: source,
        parser: parser,
        reload: reload,
        get: get
    };
    
    function get() {
    
    (...)

Once you copy the functions to an object, this object becomes a "loader" that can load a "source" synchronously with a "parser". A "source" can be a file path and the "parser" can be simply Node.js' `require` function.

Let's now see how to use this with a couple object builders. Once again, I'll borrow an example from Carcass; the [loaderSync benchmark script](https://github.com/devo-ps/carcass/blob/master/benchmark/proto.loaderSync.js). The first builder generates a function and copies the methods from what we've prepared. The second one copies the methods to the prototype of a builder class:

    
    (...)
    
    function LoaderA(_source) {
        function loader() {
            return loader.get();
        }
        loader.mixin = mixin;
        loader.mixin(loaderSync);
        loader.source(_source);
        return loader;
    }
    
    (...)
    
    function LoaderC(_source) {
        if (!(this instanceof LoaderC)) return new LoaderC(_source);
        this.source(_source);
    }
    LoaderC.prototype.mixin = mixin;
    LoaderC.prototype.mixin(loaderSync);
    
    (...)

Here we can see the two approaches. Let's compare them quickly:

FeatureLoader ALoader C

**Instantiating**
`var a = LoaderA(...)`

`var c = LoaderC(...)` or `var c = new LoaderC(...)`

**Appearance**
Generates a function

Builds a typical instance which is an object.

**Invoking directly**
`a()` or `a.get()`

`c.get()`

**Invoking as a callback**
`ipsum(a)`

`ipsum(c.get.bind(c))`

**Performance † of instantiating**
-
100x faster

**Performance of invoking**
_idem_

_idem_

**†**: (check it yourself by [benchmarking Carcass with `make bm`](https://github.com/devo-ps/carcass/blob/master/Makefile))

### "Protos" and beyond

That last approach is gaining traction among our team; we prepare functions for our object builders (which, by the way, we call "protos"). While we still choose to use prototypes in some occurrences, it is mainly because it is
faster to get done. For the sake of convenience, we also sometimes rely on functions rather than objects to invoke our "protos", however keep in mind that this is a performance trade-off.

I'll wrap this up mentioning one more method we use, admittedly less often: "Object alter". The idea is to rely on an "alter" function designed to change objects passed to it. This is sometimes also called a "mixin". An example from [vsionmedia's trove of awesomeness on Github](https://github.com/visionmedia/configurable.js):

    
    (...)
    
    module.exports = function(obj){
    
        obj.settings = {};
    
        obj.set = function(name, val){
            (...)
        };
    
        (...)
    
        return obj;
    };

### Resources

  * [A case for prototypes](http://killdream.github.io/2011/10/09/understanding-javascript-oop.html)
  * [MDN Object reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)


