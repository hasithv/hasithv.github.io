+++
tags = ["stochastics","lecture"]
title = '6.2 - The Invariance Principle'
date = 2024-08-12T00:29:17-05:00
draft = false
+++

Let $\{\xi_m\}_{n \in \mathbb{N}}$ be a sequence of i.i.d. random variables such that $\mathbb{E}[\xi_n] = 0$ and $\mathbb{E}[\xi_n^2] = 1$. Then, define
$$S_0 = 0, \quad S_N = \sum_{i=1}^N \xi_i$$

and by the Central Limit Theorem, rescaling $S_N$ by $\sqrt{N}$, we get that
$$\frac{S_N}{\sqrt{N}} \xrightarrow{d} \mathcal{N}(0,1)$$
(the $\xrightarrow{d}$ means convergence in distribution) as $N \rightarrow \infty$. Using this, we can define a continuous random function $W^N_t$ on $t \in [0,1]$ such that $W_0^N = 0$ and
$$W_t^N = \frac{1}{\sqrt{N}}(\theta S_k + (1-\theta)S_{k+1}), \quad Nt \in (k,k+1], \quad k = 0,1,\ldots,N-1$$

where $\theta = \lceil Nt \rceil - Nt$. Essentially, this equation is just a linear interpolation between the points of $S_N$ and $S_{N+1}$. The rescaling factor of $\frac{1}{\sqrt{N}}$ is to ensure that the variance of $W_t^N$ is 1 after 1 second.

Now, we will define the Wiener process.

> **Theorem 6.4:**_(Wiener Process)_ As $N \rightarrow \infty$, 
> $$W^N \xrightarrow{d} W$$
> where $W$ is the Wiener process and the distribution of $W$ on $\Omega \in C[0,1]$ is called the Wiener measure.
