+++
tags = ["stochastics", "lecture"]
title = '6.1 - The Diffusion Limit of Random Walks'
date = 2024-08-10T15:41:44-07:00
draft = false
+++

## Random Walk
Let $\{\xi_i\}$ be i.i.d. random variables such that $\xi_i = \pm 1$ with probability $1/2$. Then, define
$$X_n = \sum_{k=1}^{n} \xi_k, \quad X_0 = 0.$$
$\{X_n\}$ is the familiar symmetric random walk on $\mathbb{Z}$. Let $W(m,n) = \mathbb{P}(X_N = m)$. It is easy to see that
$$W(m,n) = {N \choose (N+m)/2} \left( \frac{1}{2} \right)^N$$
and that the mean and std are
$$\mathbb{E}[X_N] = 0, \quad \sigma^2_{X_N} = N$$

## Diffusion Coefficient
> **Definition 6.2:** _(Diffusion coefficient)_. The diffusion coefficient $D$ is defined as
> $$D = \frac{\langle (X_N - X_0)^2 \rangle}{2N}$$
> And for a general stochastic process it is
> $$D = \lim_{t \rightarrow \infty} \frac{\langle (X_t - X_0)^2 \rangle}{2dt}$$
> where $d$ is the space dimension.

Intuitively, the diffusion coefficient tells us the rate at which variance changes [[1](https://physics.stackexchange.com/a/52977/250863)]. For the random walk, $D = 1/2$.

## Continuum limit of the random walk

Let's define the step length of the random walk to be $l$ and the timestep to be $\tau$. Now, fixing $(x,t)$ consider the following limit:
$$N,m \rightarrow \infty, \quad l,\tau \rightarrow 0, \quad N\tau = t, \quad ml=x$$

The diffusion coefficient is then computed as (with $d=1$)
$$\begin{align} 
D &= \lim_{t \rightarrow \infty}  \frac{\langle (X_t - X_0)^2 \rangle}{2t} \\
&= \lim_{t \rightarrow \infty}  \frac{\langle (X_{N\tau} - X_0)^2 \rangle}{2N\tau} \\
&= \lim_{t \rightarrow \infty}  \frac{\langle X_{N\tau}^2 \rangle}{2N\tau} \\
&= \lim_{t \rightarrow \infty}  \frac{N l^2}{2N\tau} \\
&= \frac{l^2}{2\tau} \\
\end{align}$$
:warning: the book mentions "fixing" the diffusion constant, is that different from the computation above?

When we are taking the continuum limit of the random walk, $N, m \gg 1$, and so $m/N = (x/l) (\tau/t) = (x/t) (\tau/l) \rightarrow 0$. Thus, $m \ll N$. Now, we can expand $W(m,N)$ using Stirlings formula $\log n! = (n+\frac{1}{2})\log n - n + \frac{1}{2} \log 2\pi + O(n^{-1})$ when $n \gg 1$
$$\begin{align}
W(m,N) &= \frac{N!}{\left(\frac{N+m}{2}\right)! \left(\frac{N-m}{2}\right)!} \left(\frac{1}{2}\right)^N \\
\log W(m,n) &= \log \left( \frac{N!}{\left(\frac{N+m}{2}\right)! \left(\frac{N-m}{2}\right)!} \left(\frac{1}{2}\right)^N \right) \\
&\approx \log N! - N\log 2 - \left(\log\left(\frac{N + m}{2}\right)! + \log\left(\frac{N-m}{2}\right)! \right) \\
&\approx \left(N+\frac{1}{2}\right)\log N - N + \frac{1}{2}\log 2\pi - N\log 2 \\ 
& \quad \quad - \left(\log\left(\frac{N + m}{2}\right)! + \log\left(\frac{N-m}{2}\right)! \right) \\
&\approx \left(N+\frac{1}{2}\right)\log N - N + \frac{1}{2}\log 2\pi - N\log 2 \\ 
& \quad \quad - \frac{1}{2} \left( N + m + 1 \right) \log \left(\frac{N+m}{2}\right) + \frac{N+m}{2} - \frac{1}{2}\log 2\pi \\
& \quad \quad - \frac{1}{2} \left( N - m + 1 \right) \log \left(\frac{N-m}{2}\right) + \frac{N-m}{2} - \frac{1}{2}\log 2\pi \\
&\approx \left(N+\frac{1}{2}\right)\log N \\ 
& \quad \quad - \frac{1}{2} \left( N + m + 1 \right) \log \left( \frac{N}{2}\left(1+\frac{m}{N}\right)\right) \\
& \quad \quad - \frac{1}{2} \left( N - m + 1 \right) \log \left(\frac{N}{2}\left( 1 - \frac{m}{N} \right)\right) \\
& \quad \quad -\frac{1}{2}\log 2\pi - N\log 2
\end{align}$$
And now using that $m \ll N$ and $\log(1+x) \approx x - \frac{1}{2}x^2$ for $x \ll 1$, we have
$$\begin{align}
\log W(m,N) &\approx \left(N+\frac{1}{2}\right) \log N - (N+1) \log\left(\frac{N}{2}\right) \\
& \quad \quad - \frac{1}{2} \left( N + m + 1 \right) \log \left(1+\frac{m}{N}\right) \\
& \quad \quad - \frac{1}{2} \left( N - m + 1 \right) \log \left( 1 - \frac{m}{N} \right)\\
& \quad \quad -\frac{1}{2}\log 2\pi - N\log 2 \\
&\approx -\frac{1}{2} \log N + \log(2) - \frac{1}{2}\log 2\pi \\
& \quad \quad - \frac{1}{2} \left( N + m + 1 \right) \log \left(1+\frac{m}{N}\right) \\
& \quad \quad - \frac{1}{2} \left( N - m + 1 \right) \log \left( 1 - \frac{m}{N} \right) \\
&\approx -\frac{1}{2} \log N + \log(2) - \frac{1}{2}\log 2\pi \\
& \quad \quad - \frac{1}{2} \left( N + m + 1 \right) \left( \frac{m}{N} - \frac{1}{2}\left(\frac{m}{N}\right)^2\right) \\
& \quad \quad - \frac{1}{2} \left( N - m + 1 \right) \left( -\frac{m}{N} - \frac{1}{2}\left(\frac{m}{N}\right)^2\right) \\
&\approx -\frac{1}{2} \log N + \log(2) - \frac{1}{2}\log 2\pi - \frac{m^2}{2N} \\
\end{align}$$
$$\implies  W(m,N) \approx \left( \frac{2}{\pi N} \right)^{1/2}\exp\left(-\frac{m^2}{2N}\right) $$
If we integrate across the possible $x$ values,
$$\begin{align}
W(x,t)\Delta x &\approx \int_{x-\Delta/2}^{x+\Delta/2} W(y, t) dy \\
&\approx \sum_{\substack{k = \{m, m \pm 2, m \pm 4, \ldots\} \\ kl \in (x-\Delta x/2, x+\Delta x/2)}} W(k,N) \\
&\approx W(m,N)\frac{\Delta x}{2m}
\end{align}$$
Where $x=ml$. Then, doing some substitions, we get
$$W(x,t) = \frac{1}{\sqrt{4\pi D t} } \exp\left(-\frac{x^2}{4Dt}\right)$$
It is interesting to see that $W$ satisfies the heat equation with the initial condition
$$\begin{cases}
    \frac{\partial W(x,t)}{\partial t} = D \frac{\partial^2 W(x,t)}{\partial t^2} \\
    W(x,0) = \delta(x)
\end{cases}$$
I wonder how it satisfies the boundary condition of maintaining a constant area under the graph for all time $t$.

## Arcsine Law
Define $P_{2k,2n}$ to be the probability that a particle remains positive for $2k$ time steps before $2n$ time steps have passed. And a particle is on the positive side in an interval $[n-1,n]$ if either $X_{n-1}$ or $X_{n}$ are positive. It is true that
$$P_{2k,2n} = u_{2k}u_{2n-2k}$$
where $u_{2k} = \mathbb{P}(X_{2k} = 0)$ (:warning: the proof is non-trivial, and not too instructive so it is omitted. Though, if there is an intuitive reason, I'd like to hear it) [[2](https://math.osu.edu/sites/math.osu.edu/files/What_is_2018_Arcsine_Law.pdf)]. Now, let $\gamma(2n)$ be the number of time units that the particle spends on the positive axis in the interval $[0,2n]$. Then when $x \leq 1$,
$$\mathbb{P}\left(\frac{1}{2} < \frac{\gamma (2n)}{2n} \leq x \right) = \sum_{k,1/2<2k/2n\leq x} P_{2k,2n}$$
From the expression of $W(m,N)$,
$$u_{2k} \sim \frac{1}{\sqrt{\pi k}}, \quad P_{2k,2n} \sim \frac{1}{\pi \sqrt{k(n-k)}}$$
as $k, n-k \rightarrow \infty$. And so,
$$\begin{align}
\mathbb{P}\left(\frac{1}{2} < \frac{\gamma (2n)}{2n} \leq x \right) &= \sum_{k,1/2<2k/2n\leq x} P_{2k,2n} \\
&= \sum_{k,1/2<2k/2n\leq x}  \frac{1}{\pi n \sqrt{(k/n)(1-k/n)}} \\
&\rightarrow \frac{1}{\pi} \int_\frac{1}{2}^x \frac{dt}{\sqrt{t(1-t)}}
\end{align}$$

Which leads us to

> **Theorem 6.3:** _(Arcsine law)._ The probability that the fraction of time spenty by a particle ont he positive side is at most $x$ tends to $\frac{2}{\pi}\arcsin \sqrt{x}$:
> $$ \mathbb{P}\left(\frac{\gamma (2n)}{2n} \leq x \right) = \frac{2}{\pi} \arcsin \sqrt{x} $$
> The consequence of this theorem is that it is most likely for a radnom walk to spend either almost all of its time on the positive side, or for it to spend almost no time on the positive side.
>
> We can do this because there was no reason to limit the lower bound to $1/2$ rather than $0$ in the derivation

## References
1. Helena (https://physics.stackexchange.com/users/12948/helena), What is the physical meaning of diffusion coefficient?, URL (version: 2014-12-03): https://physics.stackexchange.com/q/52977https://physics.stackexchange.com/a/52977/250863
1. Ackelsberg, Ethan (https://math.osu.edu/sites/math.osu.edu/files/What_is_2018_Arcsine_Law.pdf), What is the Arcsine Law?, URL (version: 2024-08-11): https://math.osu.edu/sites/math.osu.edu/files/What_is_2018_Arcsine_Law.pdf