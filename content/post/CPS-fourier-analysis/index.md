---
title: NTNU CPS Fourier Analysis
date: 2026-04-13 0:00:00.000000000 +0800 CST
tags: [CyberPhysicalSystem, NTNUCSIE]
categories: [CyberPhysicalSystem]
math: true
---

## Function Decomposition

Fourier analysis represents a function as a weighted sum of basis functions:

$$
f(x)=a_1f_1(x)+a_2f_2(x)+\cdots.
$$

Two functions $f_1$ and $f_2$ are orthogonal on $[a,b]$ if their inner product is zero:

$$
\langle f_1,f_2\rangle=\int_a^b f_1(x)f_2(x)\,dx=0.
$$

### Vector-space analogy

For an orthogonal basis $\{\mathbf{i},\mathbf{j}\}$,

$$
\mathbf{y}=a_1\mathbf{i}+a_2\mathbf{j},
$$

and each coefficient can be obtained by projection:

$$
a_1=\frac{\langle\mathbf{y},\mathbf{i}\rangle}{\lVert\mathbf{i}\rVert^2},
\qquad
a_2=\frac{\langle\mathbf{y},\mathbf{j}\rangle}{\lVert\mathbf{j}\rVert^2}.
$$

The same idea is used to compute Fourier coefficients.

## Orthogonal Trigonometric Basis

The function set

$$
\left\{1,\cos\frac{n\pi x}{P},\sin\frac{n\pi x}{P}\right\},
\qquad n\in\mathbb{N},
$$

is orthogonal over any interval of length $2P$, such as $[a,a+2P]$.

The proof considers five pairings:

1. $1$ and $\cos(n\pi x/P)$
2. $1$ and $\sin(n\pi x/P)$
3. $\cos(n\pi x/P)$ and $\sin(m\pi x/P)$
4. $\cos(n\pi x/P)$ and $\cos(m\pi x/P)$ for $n\ne m$
5. $\sin(n\pi x/P)$ and $\sin(m\pi x/P)$ for $n\ne m$

For example,

$$
\int_a^{a+2P}\cos\frac{n\pi x}{P}\,dx
=\frac{P}{n\pi}
\left[\sin\frac{n\pi x}{P}\right]_a^{a+2P}=0.
$$

The remaining cases follow from periodicity and the product-to-sum identities.

## Fourier Series

For a function $f(x)$ defined on $[-P,P]$, its Fourier series is

$$
f(x)=\frac{a_0}{2}
+\sum_{n=1}^{\infty}
\left(
a_n\cos\frac{n\pi x}{P}
+b_n\sin\frac{n\pi x}{P}
\right),
$$

where

$$
\begin{aligned}
a_0&=\frac{1}{P}\int_{-P}^{P}f(x)\,dx, \\
a_n&=\frac{1}{P}\int_{-P}^{P}f(x)\cos\frac{n\pi x}{P}\,dx, \\
b_n&=\frac{1}{P}\int_{-P}^{P}f(x)\sin\frac{n\pi x}{P}\,dx.
\end{aligned}
$$

The coefficients are projections onto the orthogonal trigonometric basis. For example,

$$
a_n=
\frac{\left\langle f(x),\cos\frac{n\pi x}{P}\right\rangle}
{\left\lVert\cos\frac{n\pi x}{P}\right\rVert^2}.
$$

### Example

Consider

$$
f(x)=
\begin{cases}
0, & -\pi<x<0, \\
\pi-x, & 0\le x<\pi.
\end{cases}
$$

With $P=\pi$,

$$
a_0=\frac{1}{\pi}\int_0^\pi(\pi-x)\,dx=\frac{\pi}{2},
$$

$$
a_n=\frac{1-(-1)^n}{n^2\pi},
\qquad
b_n=\frac{1}{n}.
$$

## From Fourier Series to Fourier Integral

Let

$$
\alpha_n=\frac{n\pi}{P},
\qquad
\Delta\alpha=\frac{\pi}{P}.
$$

As $P\to\infty$, the spacing $\Delta\alpha\to0$ and the Fourier-series sum becomes an integral:

$$
f(x)=\frac{1}{\pi}\int_0^\infty
\left[A(\alpha)\cos(\alpha x)+B(\alpha)\sin(\alpha x)\right]d\alpha,
$$

where

$$
A(\alpha)=\int_{-\infty}^{\infty}f(t)\cos(\alpha t)\,dt,
$$

$$
B(\alpha)=\int_{-\infty}^{\infty}f(t)\sin(\alpha t)\,dt.
$$

## Complex Fourier Series

Euler's formula gives

$$
e^{ix}=\cos x+i\sin x,
\qquad
e^{-ix}=\cos x-i\sin x.
$$

Therefore,

$$
\cos x=\frac{e^{ix}+e^{-ix}}{2},
\qquad
\sin x=\frac{e^{ix}-e^{-ix}}{2i}.
$$

The Fourier series can be written in complex form as

$$
f(x)=\sum_{n=-\infty}^{\infty}C_ne^{i n\pi x/P},
$$

with

$$
C_n=\frac{1}{2P}\int_{-P}^{P}f(x)e^{-i n\pi x/P}\,dx.
$$

## Fourier Transform

Using the standard angular-frequency convention, the Fourier transform is

$$
F(\omega)=\int_{-\infty}^{\infty}f(t)e^{-i\omega t}\,dt,
$$

and the inverse transform is

$$
f(t)=\frac{1}{2\pi}\int_{-\infty}^{\infty}F(\omega)e^{i\omega t}\,d\omega.
$$

The value $F(\omega)$ describes the contribution of the complex exponential $e^{i\omega t}$ at angular frequency $\omega$.

## Discrete Fourier Transform

Sampling a continuous signal every $T$ seconds gives samples

$$
x_n=f(nT).
$$

For $N$ samples, the discrete Fourier transform (DFT) is

$$
X_k=\sum_{n=0}^{N-1}x_ne^{-i2\pi kn/N},
\qquad k=0,1,\ldots,N-1.
$$

The inverse DFT is

$$
x_n=\frac{1}{N}\sum_{k=0}^{N-1}X_ke^{i2\pi kn/N}.
$$

## Fast Fourier Transform

The fast Fourier transform (FFT) is an efficient family of algorithms for computing the DFT. A direct DFT requires $O(N^2)$ operations, while an FFT reduces the complexity to $O(N\log N)$.
