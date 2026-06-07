---
title: Linear System
date: 2026-06-07 0:00:00.000000000 +0800 CST
tags: [CyberPhysicalSystem]
categories: [CyberPhysicalSystem]
math: true
---

## linear system analysis
$x_1' = 5x_1+3x_2$\
$x_2'=-6x_1-4x_2$\
can be solve in linear algebra
$$\begin{bmatrix} x_1'  \\ x_2'  \end{bmatrix} = \begin{bmatrix} 5 & 3 \\ 6 & -4 \end{bmatrix}\begin{bmatrix} x_1 \\ x_2 \end{bmatrix}$$
we can change $x_1'=ax_1$ to $x_1(t)=Ce^{at}$

and can we do some translation for 
$$\begin{cases} x_1'= a_1x_1+a_2x_2 \\ x_2'=a_3x_1+a_4x_2  \end{cases}$$
 into 
$$\begin{cases} y_1'= a_ay_1 \\ y_2'=a_by_2  \end{cases}$$
than use 
$$\begin{cases} y_1(t) \\ y_2(t)  \end{cases}$$
? is it useful?

we can change it first change 
$$\begin{bmatrix} x_1'  \\ x_2'  \end{bmatrix} = \begin{bmatrix} a_1 & a_2 \\ a_3 & a_4 \end{bmatrix}\begin{bmatrix} x_1 \\ x_2 \end{bmatrix}$$
 into 
 $$\begin{bmatrix} y_1'  \\ y_2'  \end{bmatrix} = \begin{bmatrix} a_a & 0 \\ 0 & a_b \end{bmatrix}\begin{bmatrix} y_1 \\ y_2 \end{bmatrix}$$

$x' = Ax$ that A is operator, x is operand

Every vector x in $E$ (n dimension) can be uniquely express as $x=\sum_{i=1}^{n}t_ie_i$ \
$t_i$ are coordinate of x in the basis $e_i$

Each coordinate is a linear function $E\to R$, the coordinate system is an isomorphism(one-to-one and onto) $R\to E$

eigenvalues of $\begin{bmatrix} a & b \\ c & d \end{bmatrix}$ (A)\
compute $Det(A-\lambda I)=0$ 
$$ Det\begin{bmatrix} a -\lambda & b \\ c & d -\lambda \end{bmatrix}=0 $$
$(a-\lambda)(d-\lambda) - (bc)=0$ $\Rightarrow$ $\lambda_1=\alpha, \lambda_2 =\beta$\
$B = \begin{bmatrix} \lambda_1 & 0 \\ 0 & \lambda_2 \end{bmatrix} = \begin{bmatrix} \alpha & 0 \\ 0 & \beta \end{bmatrix}$\
$x' = Ax; y = Qx; x = Q^{-1}y;x'=Q^{-1}y'$\
$y'=Qx'=QAx=QAQ^{-1}y=By$\
$\begin{bmatrix} y_1' \\ y_2' \end{bmatrix} = \begin{bmatrix} \alpha & 0 \\ 0 & \beta \end{bmatrix}\begin{bmatrix} y_1  \\ y_2 \end{bmatrix} = \begin{bmatrix} \alpha y_1 \\ \beta y_2 \end{bmatrix}$

if eigenvalue $\lambda$ of $A$ has i（maginary part）, the system $x=Ax$ can't solve by $x(t)=Ce^{\lambda t}$

## $e^{\begin{bmatrix} a & b \\ c & d \end{bmatrix}}$

$e^T = \sum_{n=0}^\infty \frac{T^k}{k!}$ = $\frac{1}{k_0}T^{k_0} + \frac{1}{k_1}T^{k_1} + \cdots$ where $k_i$ is the smallest integer such that $T^{k_i}$ is linearly dependent on $I, T, T^2, \cdots, T^{k_i-1}$

Proposition: Let $P$, $S$, $T$ denote generators on $R^n$.Then:
1. if $Q = PTP^{-1}$, then $e^Q = Pe^TP^{-1}$
2. if ST = TS, then $e^{S+T} = e^Se^T$
3. if $e^{-S} = (e^S)^{-1}$
4. if $n = 2$ and $T = \begin{bmatrix} a & -b \\ c & d \end{bmatrix}$, than $e^T = e^a\begin{bmatrix} \cos b & -\sin b \\ \sin b & \cos b \end{bmatrix}$

### example $e^B$
$B = \begin{bmatrix} \lambda & 0 \\ 0 & \mu \end{bmatrix}$, find $e^B$\
$\Rightarrow e^B = \sum_{k=0}^\infty \frac{B^n}{k!}
= \sum_{k=0}^\infty \frac{1}{k!}\begin{bmatrix} \lambda & 0 \\ 0 & \mu \end{bmatrix} ^k
= \begin{bmatrix} \sum_{k=0}^\infty \frac{\lambda^k}{k!} & 0 \\ 0 & \sum_{k=0}^\infty \frac{\mu^k}{k!} \end{bmatrix} 
= \begin{bmatrix} e^\lambda & 0 \\ 0 & e^\mu \end{bmatrix}$

### example $e^T$
$T = \begin{bmatrix} a & 0 \\ b & a \end{bmatrix}$, find $e^T$\
$\Rightarrow T= aI+B$ where $B = \begin{bmatrix} 0 & 0 \\ b & 0 \end{bmatrix}$\
$e^T = e^{aI+B} = e^{aI}e^B (\because (aI)B = B(aI))$\
$= e^a\sum_{k=0}^\infty \frac{1}{k!}\begin{bmatrix} 0 & 0 \\ b & 0 \end{bmatrix}^k$
$= e^a e^B$ ($e^{aI} \cdot e^B$ since B is a matrix, so $e^{aI} = \sum_{n=0}^\infty \frac{(aI)^k}{k!}$ 
= $\frac{1}{k_0}a^{k_0}I + \frac{1}{k_1}a^{k_1}I + \cdots$ , then since it will times $e^B$ then it can don't need $I$)\
$e^B = \sum_{k=0}^\infty \frac{1}{k!}\begin{bmatrix} 0 & 0 \\ b & 0 \end{bmatrix}^k = \frac{B^0}{0!} + \frac{B^1}{1!}
= I + B$
$e^T = e^a(I+B) = e^a\begin{bmatrix} 1 & 0 \\ b & 1 \end{bmatrix}$

### example $e^T$
$T = \begin{bmatrix} \lambda & 0 \\ 1 & \lambda \end{bmatrix}$, find $e^T$\
$e^T = \begin{bmatrix} e^\lambda & 0 \\ e^\lambda & e^\lambda \end{bmatrix}$ (since $T = \lambda I + B$ where $B = \begin{bmatrix} 0 & 0 \\ 1 & 0 \end{bmatrix}$, then $e^T = e^{\lambda I}e^B = e^\lambda e^B$ and $e^B = I + B$) \
$= e^\lambda\begin{bmatrix} 1 & 0 \\ 1 & 1 \end{bmatrix}$

### EXAMPLE
if $x = R^n$ is an eignenvector of $T$ with eigenvalue $\alpha$, then $x$ is also an eigenvector of $e^T$ with eigenvalue $e^\alpha$\

### Proof
From $Tx = \alpha x$\
$\Rightarrow e^Tx=\sum_{k=0}^\infty \frac{T^k}{k!}x
= \sum_{k=0}^\infty \frac{1}{k!}\alpha^kx$
$= \lim_{n\to \infty} \sum_{k=0}^n \frac{T^kx}{k!}$\
$= \lim_{n\to \infty} \sum_{k=0}^n \frac{\alpha^kx}{k!}$ ($Tx = \alpha x$ so $T^kx = \alpha^kx$)\
$= \sum_{k=0}^\infty \frac{\alpha^k}{k!}x = e^\alpha x$
## Proposition $\frac{d}{dt}e^{tA} = Ae^{tA} = e^{tA}A$
### Part I show that $\frac{d}{dt}e^{tA} = Ae^{tA}$
$\frac{d}{dt}e^{tA} = \lim_{h\to 0} \frac{e^{(t+h)A} - e^{tA}}{h}$\
$= \lim_{h\to 0} \frac{e^{tA}e^{hA} - e^{tA}}{h}$ (since $e^{(t+h)A} = e^{tA}e^{hA}$)\
$= \lim_{h\to 0} e^{tA}\frac{e^{hA} - I}{h}$\
$= e^{tA}\lim_{h\to 0} \frac{e^{hA} - I}{h}$\
$= e^{tA}\lim_{h\to 0} \frac{\sum_{k=0}^\infty \frac{(hA)^k}{k!} - I}{h}$\
$= e^{tA}\lim_{h\to 0} \frac{\sum_{k=0}^\infty \frac{h^kA^k}{k!}}{1}$ (Loppital's rule, since $\sum_{k=0}^\infty \frac{(hA)^k}{k!} - I$ is 0 when $h=0$)\
$= e^{tA}A$
### Part II show that $e^{tA}A = Ae^{tA}$
$e^{tA}A = \sum_{k=0}^\infty \frac{(tA)^k}{k!}A$\
$\Rightarrow e^{tA}A = \sum_{k=0}^\infty \frac{t^kA^k}{k!}A$\
$= A \sum_{k=0}^\infty \frac{t^kA^k}{k!}$ ($\frac{t^kA^k}{k!} = \beta A^k$, then $A\beta A^k = \beta A^{k+1}$, so $A$ can move to the front of the sum)\
$= Ae^{tA}$

### Theorem: For $x'=Ax$ ($A\in R^n$), $x(0) = k \in R^n$ the general solution is $x(t) = e^{tA}k$ and the solution is unique
#### Proof
$\frac{d}{dt}e^{tA}k = Ae^{tA}k$ (from the proposition above)\

Let $x(t) be any solution and let $y(t) = e^{-tA}x(t)$\
$\Rightarrow y'(t) = -Ae^{-tA}x(t) + e^{-tA}x'(t) = -Ae^{-tA}x(t) + e^{-tA}Ax(t) = 0$\
$\Rightarrow y'(t) = \frac{d}{dt}(e^{-tA}x(t))
= \frac{d}{dt}e^{-tA}x(t) + e^{-tA}\frac{d}{dt}x(t) = (-Ae^{-tA})x(t) + e^{-tA}Ax(t) = 0$\
$\Rightarrow y(t) = y(0) = e^{-0A}x(0) = C$\
$\Rightarrow y(t) = e^{-tA}\cdot x(t)$
$x(t) = e^{tA}y(t) = e^{tA}C$
### Example
Find the general solution of $x' = Ax$ where $a = \begin{bmatrix} a & 0 \\ b & a \end{bmatrix}$\
$\Rightarrow$ The general solution is $e^{tA}k$ where $k = \begin{bmatrix} k_1 \\ k_2 \end{bmatrix}$\
from previous example, we know\
$e^A = e^a\begin{bmatrix} 1 & 0 \\ b & 1 \end{bmatrix}$\
$e^{tA} = e^{ta}\begin{bmatrix} 1 & 0 \\ tb & 1 \end{bmatrix}$\
$\Rightarrow e^{tA}k = e^{ta}\begin{bmatrix} 1 & 0 \\ tb & 1 \end{bmatrix}\begin{bmatrix} k_1 \\ k_2 \end{bmatrix} = e^{ta}\begin{bmatrix} k_1 \\ tbk_1 + k_2 \end{bmatrix}$\
$\Rightarrow \begin{cases} x_1(t) = e^{ta}k_1 \\ x_2(t) = e^{ta}(tbk_1 + k_2) \end{cases}$

## predict the behavior of the system $x' = Ax$ by looking at the eigenvalues of $A$
### $y' = By$ where $B = \begin{bmatrix} \lambda_1 & 0 \\ 0 & \lambda_2 \end{bmatrix}$
then $y_1' = \lambda_1 y_1$ and $y_2' = \lambda_2 y_2$, the solution is for some negative $\lambda_1$ and $\lambda_2$, it will converge to $(0, 0)$ for $(y_1, y_2)$ as time $t$ goes by, for some positive $\lambda_1$ and $\lambda_2$, it will diverge to infinity for $(y_1, y_2)$, for some $\lambda_1$ and $\lambda_2$ with different sign, it will diverge to infinity for some direction and converge to (0, 0) for some direction for $(y_1, y_2)$
#### three kinds of behavior for $y' = By$:
$\det(\begin{bmatrix} \lambda_1-a & b \\ c & \lambda_2-a \end{bmatrix}) > 0$ and it will be $\alpha\lambda^2 + \beta\alpha + \gamma = 0$ 
- if $\beta^2 - 4\alpha\gamma > 0$, $B= \begin{bmatrix} \lambda_1 & 0 \\ 0 & \lambda_2 \end{bmatrix}$,             it will have two different eigenvalues
- if $\beta^2 - 4\alpha\gamma < 0$ ($B= \begin{bmatrix} a & -b \\ b & a \end{bmatrix}$), it will be $\begin{bmatrix}y_1(t) \\ y_2(t) \end{bmatrix} = e^{a t}\begin{bmatrix} k_1\cos b t & -k_2\sin b t \\ k_1\sin b t & k_2\cos b t \end{bmatrix}\begin{bmatrix} k_1 \\ k_2 \end{bmatrix}$, it will be a spiral, if $a < 0$ it will be a spiral sink, if $a > 0$ it will be a spiral source, if $a = 0$ it will be a center
- if $\beta^2 - 4\alpha\gamma = 0$, it will have one eigenvalue, $B = \begin{bmatrix} \lambda & 1 \\ 0 & \lambda \end{bmatrix}$, the solution is $\begin{bmatrix} y_1(t) \\ y_2(t) \end{bmatrix} = e^{\lambda t}\begin{bmatrix} k_1 + k_2 t \\ k_2 \end{bmatrix}$, if $\lambda < 0$ it will be a degenerate node sink, if $\lambda > 0$ it will be a degenerate node source, if $\lambda = 0$ it will be a line of fixed points

### tune the system behavior
We will change $A$ to $G$ and let it become any kind of converge model. Bring some diverge system to converge system.

## system stability
### Definition(stability)
Suppose $\bar{x} \in W$ is an equilibrium of the differential equation $x' = f(x)$, then $\bar{x}$ is said to be a "stable equilibrium" if for every neiborhood $U$ of $\bar{x}$ in $W$, there is a neighborhood $U_1$ of $\bar{x}$ in $U$ such that every solution $x(t)$ with $x(0)$ in $U_1$ is defined and in U fo all $t<0$

if $\lim_{t\to \infty} x(t) = \bar{x}$ then $\bar{x}$ is said to be an "asymptotically stable equilibrium"


## from $x'=Ax$ to system control
$\Rightarrow x'=Mx$ where $M = A - []$ and we control matrix $[]$\
e.g. A will make system be circle, but we want system to be converge, so we can add some control matrix to make the system be converge, so does deverge.
## the state space model (morden control)
### damper effect
in the physics, we have know the Hooke's law $F = k\Delta x$ can apply in spring, and there is another thing called damper（阻尼）\
damper is a device that can dissipate energy, it can be used to reduce the oscillation of a system, it can be used to make a system be more stable, it can be used to make a system be more comfortable, it can be used to make a system be more safe
### Example: spring-mass-damper system
```
            -> x(t)
<--f_k(t)-spring--|====| f(t)->
<--f_b(t)-damper--|====|
```
$f_k(t) = -kx(t)$(Hooke's law) by spring\
$f_b(t) = -b \cdot \frac{d\lambda (t)}{dt}$ ($\frac{d\lambda (t)}{dt}$ is the velocity of the mass, $b$ is the damping coefficient)\
$F=ma = m\frac{d^2x(t)}{dt^2}$ we have $f(t) + f_k(t) + f_b(t) = m\frac{d^2x(t)}{dt^2}$\
$\Rightarrow m\frac{d^2x(t)}{dt^2} + b\frac{dx(t)}{dt} + kx(t) = f(t)$\
$\Rightarrow m\cdot x''+bx'+kx = f(t)$

define state variable $z_1(t) = x(t)$ and $z_2(t) = x'(t) = z_1'(t)$\
Further, let $u(t)$ be system's input $\Rightarrow u(t) = f(t)$, let $y(t)$ be system's output $\Rightarrow y(t) = x(t)$

$D \Rightarrow z_2'(t)=\frac{1}{m}u(t) - \frac{b}{m}z_2(t) - \frac{k}{m}z_1(t)$\

$\begin{bmatrix} z_1'(t) \\ z_2'(t) \end{bmatrix} = \begin{bmatrix} 0 & 1 \\ -\frac{k}{m} & -\frac{b}{m} \end{bmatrix}\begin{bmatrix} z_1(t) \\ z_2(t) \end{bmatrix} + \begin{bmatrix} 0 \\ \frac{1}{m} \end{bmatrix}u(t)$\
and y(t) = $\begin{bmatrix} 1 & 0 \end{bmatrix}\begin{bmatrix} z_1(t) \\ z_2(t) \end{bmatrix} + \begin{bmatrix} 0 \end{bmatrix}u(t)$

In general, we can write the state space model as\
$\begin{cases} z'(t) = Az(t) + Bu(t) \\ y(t) = Cz(t) + Du(t) \end{cases}$ where $z(t) = \begin{bmatrix} z_1(t) \\ z_2(t) \end{bmatrix}$\
$A$: state matrix / system matrix (suppose $u(t) = 0 \Rightarrow z'(t) = Az(t) \Rightarrow x'=Ax$)\
$B$: input matrix / control matrix (i.e., how the input imapcts the state variables)\
$C$: output matrix (i.e., how the state variables affect output)\
$D$: direct transfer matrix (i.e., how the input directly affects output)\
$z(t)$: state variables (a vector)\
$u(t)$: input\
$y(t)$: output

#### apply example
suppose $u = -Kz(t) \Rightarrow Az(t) + B(-Kz(t)) = (A-BK)z(t)$, that $A-BK$ is the $M$ we mentioned before, so we can change the system behavior by changing $K$
## state feedback control

## Lyapunov stability criteria
Let $\bar{x}\in W$ br an equilibrium for a dynamical system $x' = f(x)$\
Let $V: U\to R$ be a continuous function defined on a neighborhood $U\in W$ of $x$, differentiable on $U - \bar{x}$\
such that
1. $V(\bar{x}) = 0$ and $V(x) > 0$ if $x \neq \bar{x}$
2. $V(x)\leq 0$ in $U - \bar{x}$, if 1, 2 are true, then the system is stable at $\bar{x}$\
3. $V(x) < 0$ in $U-\bar{x}$, if 1, 2, 3 are true, then the system is asymptotically stable


asymptotically stable is when give any initial state, the system will converge to the equilibrium point as time goes by, so it is more stronger than stable\
since stable only means that the system will not diverge to infinity, but it can still oscillate around the equilibrium point
## PID control (classical control)
P: portional, I: integral, D: differential\
there are three parameters in P control, I control and D control

For example, spring-mass-damper system, the $K$ in $u = -Kz(t)$ is PD control\
we can use LQR(linear quadratic regulator) to find the optimal $K$ for the system
