---
layout:     post
title:      "Modulo who?"
date:       2011-09-16 12:00:00
categories: [Math,Programming]
tags:       [math,programming]
published:  true
---

When programmer and mathematician are talking about modulus or modulo, there is often a confusion what this term means. For programmer [modulo][1] means an operator that finds the *remainder* of division of one number by another, e.g. 5&nbsp;mod&nbsp;2 = 1. For mathematician [modulo][2] is a *congruence* relation between two numbers: *a* and *b* are said to be congruent modulo *n*, written *a*&nbsp;&#8801;&nbsp;*b*&nbsp;(mod&nbsp;*n*), if their difference *a*&nbsp;&#8722;&nbsp;*b* is an integer multiple of *n*.

These two definitions are not equivalent. The former is a special case of the latter: if *b*&nbsp;mod&nbsp;*n* = *a* then *a*&nbsp;&#8801;&nbsp;*b*&nbsp;(mod&nbsp;*n*). The inverse is not true in general case. 5&nbsp;mod&nbsp;2 = 1, and 1&nbsp;&#8801;&nbsp;5&nbsp;(mod&nbsp;2) because 1&nbsp;-&nbsp;5&nbsp;=&nbsp;–4 is integer multiple of 2. Now 5&nbsp;&#8801;&nbsp;1&nbsp;(mod&nbsp;2), because 5&nbsp;-&nbsp;1&nbsp;=&nbsp;4 is evenly divisible by 2, but 1&nbsp;mod&nbsp;2 = 1, not 5.

The biggest confusion happens when programmer and mathematician start arguing about Gauss' famous [Golden Theorem][3] where both definitions of modulus can be used.


[1]: http://en.wikipedia.org/wiki/Modulo_operator
[2]: http://en.wikipedia.org/wiki/Modular_arithmetic
[3]: http://mathworld.wolfram.com/QuadraticReciprocityTheorem.html
