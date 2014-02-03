---
layout:     post
title:      "Math and Physics of Benderama"
date:       2011-06-27 12:00:00
categories: [Humor,Math]
tags:       [humor,math]
published:  true
---

The last episode of [Futurama][1] has interesting formula involved. The entire plot is based on the Professor's latest invention &mdash; [Banach-Tarski][2] Dupla-Shrinker &mdash; the machine that produces two copies of any object at a 60% scale. It was just a matter of time when Bender found a proper usage of this machine: to replicate himself. Then two small copies of Bender replicated themselves making four smaller copies, and so forth. At some point the Professor horrified the crew that if they don't stop this unlimited growth, the total mass of all Benders will eventually be so big that the entire Earth will be consumed during the process of replication. As a proof he demonstrated this formula of the mass of all generations of Bender

{% img center http://pool.theinfosphere.org/images/thumb/0/04/Benderama_Maths.png/800px-Benderama_Maths.png %}

This is a perfect toy for a science geek. The first obvious question it brings: is this formula mathematically correct? As it turns out, it is not. Considering the scale of 60%, the cubic dependency of volume on linear dimension, and the constant density of all copies, the formula should be the following

$$
M = \displaystyle\sum\limits_{n=0}^\infty 2^n\left[\left(\frac{3}{5}\right)^{3n} M_0\right] = \displaystyle\sum\limits_{n=0}^\infty 2^n\left[\left(\frac{27}{125}\right)^n M_0\right] = \displaystyle\sum\limits_{n=0}^\infty\left(\frac{54}{125}\right)^n M_0 = \frac{125}{71} M_0
$$

As you can see the total mass of infinite number of Benders actually converges to approximately 1.76 *M*<sub>0</sub>. So from Math perspective there is nothing to worry about. But what if our assumption of constant density is invalid. Would it be a problem from Physics perspective? Let's see.

Knowing that every new copy has a size of 0.6 of the original it was made from, we have the following formula for the size of Bender in the *n*<sup>th</sup> generation

$$
L_n = 0.6^n L_0
$$

This exponential function becomes very small pretty soon. In the [154][3]<sup>th</sup> generation it already reaches the [Planck length][4], after which the further replication is physically impossible. If we calculate the total mass of 154 Bender's generations using the Professor's formula, we get [H(154)][5] &#215; [238][6] kg &#8776; 1,337.56 kg, which is nothing comparing to the Earth mass.

So we have to admit that from both Math and Physics perspective the Professor was wrong, and there was no real threat to the Earth.

Although the Professor's formula doesn't describe the replication process adequately, it's still a beautiful piece of Math because it's a formula of [harmonic series][7]. If you want to know why harmonic series is beautiful and which real processes it describes, read this nice [article][8] of John H. Webb.

And don't miss the next [episode][9] of Futurama this Thursday :-)


[1]: http://theinfosphere.org/Benderama
[2]: http://en.wikipedia.org/wiki/Banach&#8211;Tarski_paradox
[3]: http://www.wolframalpha.com/input/?i=0.6%5E154
[4]: http://en.wikipedia.org/wiki/Planck_length
[5]: http://www.wolframalpha.com/input/?i=H%28154%29
[6]: http://www.peelified.com/Futurama-Forum-1/Topic-4095-0-Benders_Weight.html
[7]: http://en.wikipedia.org/wiki/Harmonic_series_(mathematics)
[8]: http://plus.maths.org/content/perfect-harmony
[9]: http://theinfosphere.org/Ghost_in_the_Machines
