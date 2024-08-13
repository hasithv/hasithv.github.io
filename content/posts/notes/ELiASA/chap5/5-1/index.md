+++
tags = ["stochastics", "probability","lecture"]
title = '5.1 - Axiomatic Construction of Stochastic Process'
date = 2024-08-03T16:45:46-07:00
draft = false
+++

## Definition of a stochastic process

A stochastic process is a parameterized random variable $\{X_t\}_{t\in\mathbf{T}}$ defined on a probability space $(\Omega, \mathcal{F}, \mathbb{P})$ taking on values in $\mathbb{R}$. $\mathbf{T}$ can seemingly be any subset of $\mathbb{R}$. For any fixed $t \in \mathbf{T}$, we can define the random variable

$$X_t: \Omega \rightarrow \mathbb{R}, \quad \omega \rightarrowtail X_t(\omega)$$

Thinking of a simple random walk, this means that $X_t$ is a random variable that takes in some subset of $\Omega = \{H,T\}^\mathbb{N}$ and outputs a real valued number (the sum of the first $t$ values in $\omega$): $\{\omega_1, \omega_2, \ldots \} \rightarrow \sum_{n \leq t} X(\omega_n)$

On the other side of the coin, for a fixed $\omega \in \Omega$, we can define a real-valued measureable function on $\mathbf{T}$ called the trajectory of $X$

$$X_.(\omega): \mathbf{T} \rightarrow \mathbb{R}, \quad t \rightarrowtail X_t(\omega)$$

Again, back to the random walk, this means that we can get a real valued output for any given $t$. To be even more compact, we can say taht a stochastic process is a measureable function from $\Omega \times \mathbf{T}$ to $\mathbb{R}$

$$(\omega, t) \rightarrowtail X(\omega, t) := X_t(\omega)$$

The largest probability space that one can take is the infinite product space $\Omega = \mathbb{R}^\mathbf{T}$. Essentially, this is a space which can takeon any real value at any moment in time (:warning: why are we restricting ourselves to $\mathbb{R}$? Why can't it be a vector valued function?)

For finite dimension distributions, we are interested in
$$\mu_{1,\ldots,t_k}(F_1 \times \ldots \times F_k) = \mathbb{P[X_{t_1}\in F_1, \ldots X_{t_k} \in F_k]}$$

> **Theorem 5.2:** (Kolmogorov's extension theorem). Kolmogorov's extension theorem allows us to say, for any $\mu$ invariant under permuting the order of $t_k$ and $F_k$ and also adding additional time points with their associated $F$ being $\mathbb{R}$, that there exists a probability space and a stochastic prcess such that
> $$\mu_{1,\ldots,t_k}(F_1 \times \ldots \times F_k) = \mathbb{P[X_{t_1}\in F_1, \ldots X_{t_k} \in F_k]}$$ 

Kolmogorov's extension theorem is very general. In fact, so general that it does not give us a very good idea of what the process actually looks like. Usually, we start with this extremely general definition and then impose stricter conditions to prove that the measure can be defined on a smaller probability space rather than $\Omega$
