+++
categories = ["puzzles"]
tags = ["probability"]
title = "That's not how Probability Works!"
date = 2024-07-30T00:07:20-07:00
draft = false
+++

I was recently doing a probability puzzle that I can't quite remember the context of, but I came across the answer that the probability would be
$$\mathbb{P}(X) = n p^n \; \quad \forall \: n\in\mathbb{N}, p \in [0,1].$$

But this is obviously wrong! Plug in $p=.9, n=2$, and you get that $\mathbb{P}(X) = 1.62$. However, for $p=0.5$, $\mathbb{P}(X)$ will remain $\leq 1$ for all $n \in \mathbb{N}$. So, somewhere in the interval $(0.5,0.9)$, we reach a critical value where any $p$ greater than that will result in a probability greater than one, and any value less than it will be a bit more reasonable.

So, what is this critical value that will help me save face?

Well, the question we are trying to answer, phrased a bit more formally, is:

> find the largest $p \in [0,1]$ such that $np^n \leq 1$ for all $n \in \mathbb{N}.$ 

First, we rephrase the problem by stating
$$np^n \leq 1 \iff p^n \leq \frac{1}{n}.$$

Visually, this means that the exponential graph of $f_p(n) = p^n$ can never go above $g_p(n) = \frac{1}{n}$ for some fixed $p$. From this, we can deduce that the critical value of $p$, which we will denote as $p_0$ will satisfy the following relation:

> Given the parametrized forms of $f_p$ and $g_p$ 
> $$F_{p}(t) = \begin{bmatrix}
t \\
f_p(t)
\end{bmatrix}, \; 
G_p(t) = \begin{bmatrix}
t \\
g_p(t)
\end{bmatrix},$$
> the critical $p_0$ value will be such that 
> $$F_{p_0}(t_0) = G_{p_0}(t_0),\text{ and } \dot{F}_{p_0}(t_0) = \lambda \dot{G}_{p_0}(t_0)$$
> for some $\lambda \in \mathbb{R}, t_0 \in \mathbb{R}^+$. In other words, their velocities will point in the same direction (and, perhaps more intuitively, the outward normals of each curve will be parallel, so the graphs 'kiss' at some $t_0$ with the choice of $p_0$).

Now, we have a fairly simple problem to solve. Because the $x$ component of $F$ and $G$ are always equal, we immediately find that $\lambda = 1$ for their time derivatives to be equal to each other. Now, that leads us to solve for a $p_0$ and $n$ such that
$$
\begin{aligned}
    f_{p_0}(t_0) &= g_{p_0}(t_0) \\
    \implies p_0^n &= \frac{1}{t_0}
\end{aligned}
$$
and
$$
\begin{aligned}
     \dot{f}_{p_0}(t_0) &= \dot{g}_{p_0}(t_0) \\
    \implies -\ln(p_0)p_0^n &= \frac{1}{t_0^2}
\end{aligned}
$$
So, rather unsatisfyingly, we boiled it down to a system of nonlinear equations
$$
\begin{cases}
    p_0^n = \frac{1}{t_0}, \\
    \ln(p_0)p_0^n = -\left(\frac{1}{t_0}\right)^2
\end{cases}
$$
which I cannot solve, but Desmos tells me that $p_0 \approx 0.6922$ and $t_0 \approx 2.7181$.

Thus, my answer would have been reasonable in *some* convoluted scenario in which $p < 0.6922$. 

(This answer, too, is not totally right! This is because there may be a larger $p$ value that satisfies $np^n \leq 1$ for $n \in \mathbb{N}$ but *not* for $n\in \mathbb{R}^+$. We solved for the $n \in \mathbb{R}^+$ case, which would technically give us a lower bound for $p_0$)

