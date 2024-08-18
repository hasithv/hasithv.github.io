+++
tags = ["stochastics", "probability", "lecture"]
title = '5.4 - Gaussian Processes'
author = "Hasith Vattikuti"
date = 2024-08-06T21:17:49-07:00
draft = false
+++

> **Definition 5.9:** A stochasitc process $\{X_t\}_{t \geq 0}$ is a _Gaussian Process_ if its finite dimensional distributions are consistent Gaussian measures for any $0 \leq t_1 < t_2 < \ldots < t_k$.

Recall that a Gaussian random vector $\mathbf{X} = (X_1, X_2,\ldots,X_n)^T$ is completely characterized by its first and second moments
$$\mathbf{m} = \mathbb{E}[\mathbf{X}], \quad \mathbf{K} = \mathbb{E}[(\mathbf{X} - \mathbf{m}) (\mathbf{X} - \mathbf{m})^T]$$

Meaning that the characteristic function is expressed only in terms of $\mathbf{m}$ and $\mathbf{K}$
$$\mathbb{E}\left[e^{i \mathbf{\xi} \cdot \mathbf{X}}\right] = e^{i \mathbf{\xi} \cdot \mathbf{m} - \frac{1}{2}\mathbf{\xi}^T \mathbf{K} \mathbf{\xi}} $$
This means that for any $0 \leq t_1 < t_2 < \ldots < t_k$, the measure $\mu_{t_1, t_2, \ldots, t_k}$ is uniquely determined by an $\mathbf{m} = (m(t_1), \ldots, m(t_k))$ and a covariance matrix $\mathbf{K}_{ij} = K(t_i, t_j)$. Because our $\mu$ satisifes the conditions for Kolomorov's extension theorem, we have a probability space and a stochastic process associated with $\mu$.

:warning: Isn't one of the conditions of Kolmogorov's extension theoren that we need to be able to permute the $t_i$? How would this work if we require the $t_i$ to be increasing?

> **Theorem 5.10:** Assuming that the stochastic process $\{X_t\}_{t\in[0,T]}$ satisfies
> $$\mathbb{E} \left[ \int_0^T X_t^2 dt \right] < \infty$$
> then $m \in L^2_t$. Also, the operator
> $$\mathcal{K} f(s) := \int_0^T K(s,t) f(t) dt$$
> is nonnegative, compact on $L^2_t$

> **Proof:** For the first statement,
> $$\int_0^T \mathbb{E}[X]^2 dt \leq \int_0^T \mathbb{E}[X_t^2] dt < \infty$$
> For the second,
>$$\begin{align}
\int_0^T \int_0^T K^2(s,t) ds dt &= \int_0^T \int_0^T \mathbb{E}[\left((X_t - m(t))(X_s - m(s))\right)^2]ds dt \\
&\leq \int_0^T \int_0^T \mathbb{E}[(X_t - m(t))^2]\mathbb{E}[(X_s - m(s))^2]ds dt \\
&\leq \left( \int_0^T \mathbb{E}[X_t] dt \right) \\
&\leq \infty 
\end{align}$$
> which lets us conclude that $K \in L^2([0,T] \times [0,T])$, which tells us that $\mathcal{K}$ is compact on $L^2_t$
>
> :warning: I can't properly find what theorem lets us say the last statement, but I can trust it for now.
> 
> Furthermore, noting that $K$ is symmetric which means that $\mathcal{K}$ is self adjoint, and we can say
> $$(\mathcal{K}f, f) = \int_0^T \int_0^T \mathbb{E}[(X_t - m(t))]\mathbb{E}[(X_s - m(s))]f(t)f(s) ds dt \geq 0$$
> by symmetry of $s$ and $t$.

If we want to extend the characteristic to $L_t$ rather than the finite dimensional version, we can write
$$\mathbb{E}[e^{i(\xi,X)}] = e^{i(\xi,m) - \frac{1}{2} (\xi, \mathcal{K} \xi)}, \quad \xi \in L^2_t$$
With $(\xi,m) = \int_a^b \xi(t) m(t) dt$ and $\mathcal{K}\xi (t) = \int_a^b K(t,s) \xi(s) ds$. This is a fairly reasonable extrapolation from the finite dimensional case.

> **Theorem 5.13:** _Karhunen-Loeve expansion_. Let $(X_t)_{t \in [0,1]}$ be a Gaussian process with mean 0 and covariance function $K(s,t)$, Assume that $K$ continuous and $\{\lambda_k\}$ be the set of eigenvalues for orthonormal eigenfunctions of $K$, $\{\phi_k\}$. Then, $X_t$ has the representation of 
> $$X_t = \sum_{k=1}^\infty \alpha_k \sqrt{\lambda_k} \phi_k(t)$$
> Where $\alpha_k$ is a standard normal random variable $\mathscr{N}(0,1)$
> 
> :warning: I am omitting the proof because I feel the result is easy enough to intuitively grasp, and also it is a little theoretical, so maybe I should revisit it if I get more comfortable with proving covergence in probability.

