---
layout:     post
title:      "Fourier analysis: Fourier series"
date:       2011-08-11 12:00:00
categories: [Math]
tags:       [math]
published:  false
---

## Introduction

> Every signal has a spectrum and is determined by its spectrum. You can analyze the signal either in the time (or spatial) domain or in the frequency domain.

The question of convergence of Fourier series, believe it or not, led G. Cantor to investigate and invent the theory of infinite sets, and to distinguish different sizes of infinite sets.

> Extremely important mathematical principle: If you can compute the same thing two different ways, chances are you’ve done something significant.

## Periodicity

$$
f(t) = \cos{2\pi t} + \frac{1}{2}\cos{4\pi t}
$$

The overall pattern repeats every 1 second, but if this function represented some kind of wave would you say it had frequency 1 Hz? Somehow I don’t think so. It has *one period* but you’d probably say that it has, or contains, *two frequencies*, one cosine of frequency 1 Hz and one of frequency 2 Hz.

## It All Adds Up

Switching from amplitudes and phases to general trigonometric sum

$$
\sum\limits_{n=1}^N A_n \sin(2\pi nt + \phi_n) = \sum\limits_{n=1}^N (a_n \cos{2\pi nt} + b_n \sin{2\pi nt})
$$

$$
A_n, \phi_n, a_n, b_n \in \mathbb{R}
$$

$$
a_n = A_n \sin\phi_n, \quad b_n = A_n \cos\phi_n
$$

$$
A_n = \sqrt{a_n^2 + b_n^2}, \quad \phi_n = \arcsin\frac{a_n}{\sqrt{a_n^2 + b_n^2}} = \arccos\frac{b_n}{\sqrt{a_n^2 + b_n^2}}
$$

Switching from trigonometric sums to complex exponentials

$$
\frac{a_0}{2} + \sum\limits_{n=1}^N (a_n \cos{2\pi nt} + b_n \sin{2\pi nt}) = \sum\limits_{n=-N}^N c_n e^{2\pi int}
$$

$$
a_{-n} := a_n, \quad b_{-n} := b_n, \quad b_0 := 0
$$

$$
c_n = \frac{a_n - ib_n}{2}, \quad c_{-n} = \frac{a_n + ib_n}{2} = \overline{c}_n, \quad c_n \in \mathbb{C}
$$

$$
|c_n| = \tfrac{1}{2} \sqrt{a_n^2 + b_n^2}
$$

$$
a_n = c_n + \overline{c}_n = 2 \Re c_n, \quad b_n = i(c_n - \overline{c}_n) = -2 \Im c_n
$$

and back to amplitudes and phases

$$
A_n = 2 |c_n|, \quad \phi_n = \operatorname{Arg}c_n + \tfrac{\pi}{2}
$$

## Period, Frequencies, and Spectrum

The set of frequencies present in a given periodic signal is the *spectrum* of the signal. Note that it’s the frequencies, like ±2, ±7, ±325, that make up the spectrum, *not* the values of the Fourier coefficients f(±2), f(±7), f(±325).

$$
f(t) = \sum\limits_{n=-\infty}^{\infty} c_n e^{2\pi int/T}
$$

In the time domain the signal repeats after T seconds, while the points in the spectrum are 0, ±1/T, ±2/T, …, which are spaced 1/T apart.

> The larger the period in time the smaller the spacing of the spectrum. The smaller the period in time, the larger the spacing of the spectrum.

If you allow yourself to imagine letting T → ∞ you can allow yourself to imagine the discrete set of frequencies becoming a continuum of frequencies.


## Triangle wave

The sum of two (or a finite number) of continuous functions is continuous. The sum of two (or a finite number) of smooth functions is smooth. Therefore, only smooth functions can be represented as a finite Fourier series. Any lack of smoothness forces infinite sum.

> It takes high frequencies to make sharp corners.

$$
f(t) = \left\{
           \begin{array}{l l}
             \tfrac{1}{2} + t  & -\tfrac{1}{2} \leq t \leq 0 \\
             \tfrac{1}{2} - t  & 0 \leq t \leq \tfrac{1}{2}  \\
           \end{array} \right.
$$

0th Fourier coefficient

$$
\hat{f}(0) = \int\limits_{-1/2}^{1/2}f(t)\mathrm{d}t = S_{\Delta} = \frac{1}{2}\cdot\left(\frac{1}{2} - 0\right)\cdot\left(\frac{1}{2} - \frac{-1}{2}\right) = \frac{1}{4}
$$

Helper integral

$$
\begin{align*}
I(t) & = \int t e^{-2\pi int} \mathrm{d}t \\
     & = -\frac{1}{2\pi in} \int t\, \mathrm{d}e^{-2\pi int} \\
     & = -\frac{1}{2\pi in} \left[ te^{-2\pi int} - \int e^{-2\pi int} \mathrm{d}t \right] \\
     & = \frac{1}{2\pi in} \int e^{-2\pi int} \mathrm{d}t - \frac{te^{-2\pi int}}{2\pi in} \\
     & = \frac{e^{-2\pi int}}{4\pi^2 n^2} - \frac{te^{-2\pi int}}{2\pi in}
\end{align*}
$$

*n*-th Fourier coefficient

$$
\begin{align*}
\hat{f}(n) & = \int\limits_{-1/2}^{1/2} e^{-2\pi int} f(t) \mathrm{d}t \\
           & = \int\limits_{-1/2}^{0} e^{-2\pi int} \left(\frac{1}{2} + t\right) \mathrm{d}t + \int\limits_{0}^{1/2} e^{-2\pi int} \left(\frac{1}{2} - t\right) \mathrm{d}t \\
           & = \frac{1}{2} \int\limits_{-1/2}^{0} e^{-2\pi int} \mathrm{d}t + \int\limits_{-1/2}^{0} t\,e^{-2\pi int} \mathrm{d}t + \frac{1}{2} \int\limits_{0}^{1/2} e^{-2\pi int} \mathrm{d}t - \int\limits_{0}^{1/2} t\,e^{-2\pi int} \mathrm{d}t \\
           & = \frac{1}{2} \int\limits_{-1/2}^{1/2} e^{-2\pi int} \mathrm{d}t + \int\limits_{-1/2}^{0} t\,e^{-2\pi int} \mathrm{d}t - \int\limits_{0}^{1/2} t\,e^{-2\pi int} \mathrm{d}t \\
           & = \left.-\frac{e^{-2\pi int}}{4\pi in}\right|_{-1/2}^{1/2} + 2I(0) - I(-1/2) - I(1/2) \\
           & = \frac{e^{\pi in}}{4\pi in} - \frac{e^{-\pi in}}{4\pi in} + \frac{1}{2\pi^2n^2} - \frac{e^{\pi in}}{4\pi^2 n^2} - \frac{e^{\pi in}}{4\pi in} - \frac{e^{-\pi in}}{4\pi^2 n^2} + \frac{e^{-\pi in}}{4\pi in} \\
           & = \frac{1}{2\pi^2n^2} - \frac{e^{\pi in}}{4\pi^2 n^2} - \frac{e^{-\pi in}}{4\pi^2 n^2} \\
           & = \frac{1}{2\pi^2n^2} - \frac{1}{2\pi^2 n^2} \cdot \frac{e^{\pi in} + e^{-\pi in}}{2} \\
           & = \frac{1}{2\pi^2n^2} \left(1 - \cos{\pi n}\right) \\
           & = \left\{
                 \begin{array}{l l}
                   1/\pi^2n^2 & \text{if $n$ is odd} \\
                   0          & \text{if $n$ is even} \\
                 \end{array} \right.
\end{align*}
$$

Fourier series for triangle wave

$$
\begin{align*}
f(t) & = \frac{1}{4} + \sum\limits_{\text{$n$ odd}} \frac{1}{\pi^2n^2} \, e^{2\pi int} \\
     & = \frac{1}{4} + \sum\limits_{\substack{\text{$n$ odd} \\ n>0}} \frac{1}{\pi^2n^2} \left( e^{2\pi int} + e^{-2\pi int} \right) \\
     & = \frac{1}{4} + \sum\limits_{\substack{\text{$n$ odd} \\ n>0}} \frac{2}{\pi^2n^2} \cos{2\pi nt} \\
     & = \frac{1}{4} + \sum\limits_{k = 0}^{\infty} \frac{2}{\pi^2(2k+1)^2} \cos{2\pi (2k+1) t} \\
\end{align*}
$$

One interesting application

$$
\frac{1}{2} = f(0) = \frac{1}{4} + \frac{2}{\pi^2} \sum\limits_{k = 0}^{\infty} \frac{1}{(2k+1)^2} \Rightarrow \sum\limits_{k = 0}^{\infty} \frac{1}{(2k+1)^2} = \frac{\pi^2}{8}
$$

## The Math, the Majesty, the End

Fourier coefficients

$$
f(t) = \sum\limits_{n=-\infty}^{\infty} \hat{f}(n) e^{2\pi int}, \quad \hat{f}(n) = \int\limits_0^1 f(t) \, e^{-2\pi int} \, \mathrm{d}t
$$

Notice that although $f(t)$ is defined for a continuous variable t, the transformed function $\hat{f}(n)$ is defined on the integers.

## Orthogonality

> I think of the inner product as measuring how much one vector “knows” another; two orthogonal vectors don’t know each other.

For the definition of $L^2([0, 1])$ we assume

$$
\int\limits_0^1 |f(t)|^2 \mathrm{d}t < \infty
$$

The inner product of complex-valued functions $f(t)$ and $g(t)$ in $L^2([0,1])$ is defined to be

$$
(f,g) = \int\limits_0^1 f(t) \, \overline{g(t)} \mathbb{d}t
$$

Orthonormal basis of $L^2([0, 1])$

$$
e_n(t) = e^{2\pi int}
$$

Fourier series is a decomposition in terms of an orthonormal basis and associated inner product

$$
f = \sum\limits_{n=-\infty}^{\infty} \hat{f}(n) \, e^{2\pi int} = \sum\limits_{n=-\infty}^{\infty} (f,e_n) e_n
$$

To prove that the complex exponentials are a *basis* we have to show that

$$
\lim_{N \to \infty} \, \left|\left| f \, - \sum_{n=-N}^N (f,e_n) e_n \right|\right| = 0
$$

**Theorem** If $f(t)$ is in $L^2([0,1])$ and $\alpha_{-N}, ..., \alpha_N$ are any complex numbers, then

$$
\left|\left| f - \sum\limits_{n=-N}^N (f,e_n) e_n \right|\right| \leq \left|\left| f - \sum\limits_{n=-N}^N \alpha_n e_n \right|\right|
$$

Furthermore, equality holds only when $\alpha_n = (f, e_n)$ for every $n$.

## Norm

Rayleigh’s identity

$$
||f||^2 = \sum\limits_{n=-\infty}^{\infty} |(f,e_n)|^2 \quad \text{or} \quad \int\limits_0^1 |f(t)|^2 \mathrm{d}t \, = \sum\limits_{n=-\infty}^{\infty} |\hat{f}(n)|^2
$$

Cauchy-Schwarz inequality

$$
|(\mathbf{v}, \mathbf{w})| \leq ||\mathbf{v}|| \; ||\mathbf{w}||
$$

Cauchy-Schwarz inequality for $L^2([0,1])$

$$
\left| \int\limits_0^1 f(t) \overline{g(t)} \mathrm{d}t \right| \leq \left( \int\limits_0^1 |f(t)|^2 \mathrm{d}t \right)^{1/2} \left( \int\limits_0^1 |g(t)|^2 \mathrm{d}t \right)^{1/2}
$$

## Ring heating equation

$$
u_t = \tfrac{1}{2}u_{xx} , \quad u(x+1,t) = u(x,t) , \quad u(x,0) = f(x)
$$

We can try expanding it as a Fourier series with coefficients that depend on time:

$$
u(x,t) = \sum\limits_{n=-\infty}^{\infty} c_n(t) e^{2\pi inx} \quad \text{where} \quad c_n(t) = \int\limits_0^1 u(x,t) e^{-2\pi inx} \mathrm{d}x
$$

Differentiate $c_n(t)$ with respect to $t$

$$
c^\prime_n(t) = \int\limits_0^1 u_t(x,t) e^{-2\pi inx} \mathrm{d}x = \int\limits_0^1 \tfrac{1}{2} u_{xx}(x,t) e^{-2\pi inx} \mathrm{d}x
$$

Solve a simple ordinary differential equation to find $c_n(t)$

$$
c_n(t) = \hat{f}(n) e^{-2\pi^2n^2t} \quad \text{where} \quad \hat{f}(n) = \int\limits_0^1 f(y) e^{-2\pi iny} \mathrm{d}y
$$

The general *solution* of the heat equation is

$$
u(x,t) = \sum\limits_{n=-\infty}^{\infty} \hat{f}(n) e^{-2\pi^2n^2t} e^{2\pi inx}
$$

or in another form

$$
u(x,t) = \int\limits_0^1 g(x-y,t) f(y) \, \mathrm{d}y
$$

where function

$$
g(x,t) = \sum\limits_{n=-\infty}^{\infty} e^{-2\pi^2n^2t} e^{2\pi inx}
$$

is called *Green’s function*, or the *fundamental solution* for the heat equation for a circle. In words, $u(x,t)$ is given by the [*convolution*][1] of the initial temperature $f(x)$ with Green’s function $g(x,t)$.

> In our example, as a formula for the solution, the convolution may be interpreted as saying that for each time $t$ the temperature $u(x,t)$ at a point $x$ is a kind of smoothed average of the initial temperature distribution $f(x)$.

Observe that the convolution makes sense only if $g$ is periodic.

### Example: $u(x,0) = \sin{2\pi x}$

Simple solution

$$
f(x) = \sin{2\pi x} = \tfrac{1}{2i}(e^{2\pi ix} - e^{-2\pi ix}) \Rightarrow \hat{f}(-1) = -\tfrac{1}{2i}, \; \hat{f}(1) = \tfrac{1}{2i}, \; \hat{f}(n) = 0 \; \text{for} \; n \neq \pm 1
$$

$$
u(x,t) = -\tfrac{1}{2i} e^{-2\pi^2t}e^{-2\pi ix} + \tfrac{1}{2i} e^{-2\pi^2t}e^{2\pi ix} = e^{-2\pi^2t} \sin{2\pi x}
$$

{% codeblock gnuplot lang:bash %}
set xrange[0:0.18]
set yrange[-1:1]
set hidden3d
set isosamples 50
splot exp(-2*(pi**2)*x)*sin(2*pi*y) notitle
{% endcodeblock %}

{% img center /images/posts/fourier-ring-heating.png %}

Generic solution (just for fun)

$$
\begin{align*}
F(n) & = \int e^{-2\pi iny} \sin{2\pi y} \,\mathrm{d}y \\
     & = -\frac{1}{2\pi} \int e^{-2\pi iny} \,\mathrm{d}\cos{2\pi y} \\
     & = -\frac{1}{2\pi} \left( e^{-2\pi iny} \cos{2\pi y} - \int \cos{2\pi y} \,\mathrm{d}e^{-2\pi iny} \right) \\
     & = -\frac{1}{2\pi} \left( e^{-2\pi iny} \cos{2\pi y} + 2\pi in \int e^{-2\pi iny} \cos{2\pi y} \,\mathrm{d}y \right) \\
     & = -\frac{1}{2\pi} \left( e^{-2\pi iny} \cos{2\pi y} + in \int e^{-2\pi iny} \,\mathrm{d}\sin{2\pi y} \right) \\
     & = -\frac{1}{2\pi} \left( e^{-2\pi iny} \cos{2\pi y} + in \left( e^{-2\pi iny} \sin{2\pi y} - \int \sin{2\pi y} \,\mathrm{d}e^{-2\pi iny} \right) \right) \\
     & = -\frac{1}{2\pi} \left( e^{-2\pi iny} \cos{2\pi y} + in e^{-2\pi iny} \sin{2\pi y} - 2\pi n^2 \int e^{-2\pi iny} \sin{2\pi y} \,\mathrm{d}y \right) \\
     & = -\frac{1}{2\pi} \left( e^{-2\pi iny} \cos{2\pi y} + in e^{-2\pi iny} \sin{2\pi y} - 2\pi n^2 F(n) \right) \\
     & = n^2 F(n) -\frac{e^{-2\pi iny}}{2\pi} \left( \cos{2\pi y} + in \sin{2\pi y} \right) \Rightarrow \\
F(n) & = \frac{e^{-2\pi iny}}{2\pi(n^2-1)} \left( \cos{2\pi y} + in \sin{2\pi y} \right) \; \text{for} \; n \neq \pm 1 \Rightarrow \\
\hat{f}(n) & = \left. \frac{e^{-2\pi iny}}{2\pi(n^2-1)} \left( \cos{2\pi y} + in \sin{2\pi y} \right) \right|_0^1 = 0 \; \text{for} \; n \neq \pm 1 \\
F(1) & = \int e^{-2\pi iy} \sin{2\pi y} \,\mathrm{d}y \\
     & = \int (\cos{2\pi y} - i \sin{2\pi y}) \sin{2\pi y} \,\mathrm{d}y \\
     & = \frac{1}{2} \int \cos{4\pi y} \,\mathrm{d}y - i \int \sin^2{2\pi y} \,\mathrm{d}y \\
     & = \frac{1}{2} \int \cos{4\pi y} \,\mathrm{d}y - \frac{i}{2} \int (1 - \cos{4\pi y}) \,\mathrm{d}y \\
     & = \frac{1+i}{2} \int \cos{4\pi y} \,\mathrm{d}y - \frac{iy}{2} \\
     & = \frac{1+i}{8\pi} \sin{4\pi y} - \frac{iy}{2} \Rightarrow \\
\hat{f}(1) & = \left. \frac{1+i}{8\pi} \sin{4\pi y} \,\right|_0^1 - \left. \frac{iy}{2} \right|_0^1 = -\frac{i}{2}
\end{align*}
$$


## Cellar equation

An equation for Fourier coefficients

$$
c^{\prime\prime}_n(x) = 4\pi in \, c_n(x)
$$

has a solution

$$
c_n(x) = A_n e^{\sqrt{2\pi n}(1+i)x} + B_n e^{-\sqrt{2\pi n}(1+i)x} \quad \text{for} \quad n > 0 \\
c_n(x) = A_n e^{\sqrt{-2\pi n}(1-i)x} + B_n e^{-\sqrt{-2\pi n}(1-i)x} \quad \text{for} \quad n < 0
$$




<!-- {% img http://mathworld.wolfram.com/images/gifs/convrect.gif %}
{% img http://mathworld.wolfram.com/images/gifs/convgaus.gif %} -->


[1]: http://mathworld.wolfram.com/Convolution.html
