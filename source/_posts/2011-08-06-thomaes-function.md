---
layout:     post
title:      "Thomae's function"
date:       2011-08-06 12:00:00
categories: [Lisp,Math]
tags:       [lisp,math]
published:  true
---

[Thomae's function][1] (a.k.a. Riemann function) is defined on the interval (0, 1) as follows

$$
f(x) = \left\{
\begin{array}{l l}
  1/q & \quad \text{if $x = p/q$ is rational and $gcd(p, q) = 1$}\\
  0 & \quad \text{if $x$ is irrational}\\
\end{array} \right.
$$

Here is the graph of this function with some points highlighted as plus symbols for better view.

{% img center /images/posts/thomae.jpg %}

This function has interesting property: it's continuous at all irrational points. It's easy to see this if you notice that for any positive *ε* there is a finite number of dots above the line *y* = *ε*. That means for any irrational number *x*<sub>0</sub> you can always construct a *δ*-neighbourhood that doesn't contain any dot from the area above the line *y* = *ε*.

{% img center /images/posts/thomae-e-d.jpg %}

To generate the data file with point coordinates I wrote Common Lisp program:

{% codeblock lang:common-lisp %}
(defun rational-numbers (max-denominator)
  (let ((result (list)))
    (loop for q from 2 to max-denominator do
      (loop for p from 1 to (1- q) do
        (pushnew (/ p q) result)))
    result))

(defun thomae-rational-points (abscissae)
  (mapcar (lambda (x) (list x (/ 1 (denominator x)))) abscissae))

(defun thomae (max-denominator)
  (let ((points (thomae-rational-points (rational-numbers max-denominator))))
    (with-open-file (stream "thomae.dat" :direction :output)
      (loop for point in points do
        (format stream "~4$ ~4$~%" (first point) (second point))))))

(thomae 500)
{% endcodeblock %}

To create the images I used [gnuplot][2] commands:

    plot "thomae.dat" using 1:2 with dots
    plot "thomae.dat" using 1:2 with points

and Photoshop.


[1]: http://en.wikipedia.org/wiki/Thomae%27s_function
[2]: http://www.gnuplot.info/
