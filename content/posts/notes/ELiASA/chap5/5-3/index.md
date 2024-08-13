+++
tags = ["stochastics", "probability","lecture"]
title = '5.3 - Markov Processes'
date = 2024-08-03T23:22:06-07:00
draft = false
+++

## Markov processes in continuous time and space
Given a probability space $(\Omega, \mathcal{F}, \mathbb{P})$ and the filtration $\mathbb{F} = (\mathcal{F}_t)_{t \geq 0}$, a stochastic process $X_t$ is called a Markov process wrt $\mathcal{F}_t$ if
1. $X_t$ is $\mathcal{F}_t$-adapted
2. For any $t \geq s$ and $B \in \mathcal{R}$, we have
$$\mathbb{P}(X_t \in B | \mathcal{F}_s) = \mathbb{P}(X_t \in B | X_s)$$
Essentially, this is saying that history doesn't matter, only the current state matters. We can associate a family of probability measures $\{\mathbb{P}^x\}_{x\in\mathbb{R}}$ for the processes starting at $x$ by defining $\mu_0$ to be the point mass at $x$. Then, we still have
$$\mathbb{P}^x(X_t \in B | \mathcal{F}_s) = \mathbb{P}^x(X_t \in B | X_s), \quad t \geq s$$
and $\mathbb{E}[f(X_0)] = f(x)$ for any function $f \in C(\mathbb{R})$.

:warning: I am not fully confident on what the above section is saying. Specifically, I am having trouble with understanding how we are defining $\mathbb{P}^x$. However, I can understand the strong markov property, so I think I should be okay moving forward.

The _transition function_ of a Markov process is defined as
$$p(B,t|x,s) = \mathbb{P}(X_t \in B | X_s = x)$$
and it has the properties
1. $p(.,t|x,s)$ is a probability measure on $\mathcal{R}$
1. $p(B,t|.,s)$ is a measurable function on $\mathbb{R}$
1. $p$ satisfies
$$p(B,t|y,s) = \int_\mathbb{R} p(B,t|y,u) p(dy,u|x,s), \quad s \leq u \leq t$$
The last property is the continuous analog of the _Chapman-Kolmogorov equation_, and it essentially lets us break the transition function into two the transition from $s$ to $u$ and from $u$ to $t$.

Now, we can write the expenctation from $x$ as
$$ \begin{multline}
\mathbb{E}^x[f(X_{t_1}, X_{t_2}, \ldots, X_{t_n})] = \int_\mathbb{R} \ldots \int_\mathbb{R} f(x_1,x_2,\ldots,x_n) p(dx_n,t_n|x_{n-1},t_{n-1}) \\ \ldots p(dx_2, t_2 | x_1,t_1) p(dx_1, t_1|x,0)
\end{multline}
$$

when $t_n$ are strictly increasing.

$p(y,t|x,s)$ is a _transition density function_ of $X$. :warning: The book makes it seem like this is not always the case, but I fail to see when it isn't.

A stochastic process is _stationary_ if the joint distributions are translation invariant in time. However, if the process only depends on the difference between time, then the process is _homogeneous_. The difference is that a stationary process has the same distribution at all times, while a homogeneous process has the same distribution for all time differences.

:warning: at this point, the book dives into semigroup theory which I know nothing about, so I will skip this section for now.

## Example 5.7 - Q-process
Recall the definition of a generator $\mathbf{Q}$ to be
$$\mathbf{Q} = \lim_{h \rightarrow 0+} \frac{1}{h} (\mathbf(P)(h) - \mathbf{I})$$
Now, we will define an infinitesimal generator $\mathcal{A}$ on a sample space $S = \{1,2,\ldots,I\}$:
$$\mathcal{A}f = \lim_{t \rightarrow 0+} \frac{\mathbb{E}[f(X_t)] - f}{t}$$
and
$$\begin{align}
\mathcal{A}f(i) &= \lim_{t \rightarrow 0+} \frac{\mathbb{E}^i[f(X_t)] - f(i)}{t} \\
&= \lim_{t \rightarrow 0+} \frac{1}{t} \left(\sum_{j \in S} (P_{ij} - \delta_{ij})f(j)\right)\\
&= \sum_{j \in S} q_{ij} f(j), \quad i \in S
\end{align}$$
Thus, the generator $\mathbf{Q}$ is exactly the infinitesimal generator of $X_t$. This is important to digest especially because the $\mathcal{A}$ is new to me.

Extending the idea above, we get the backward Kolmogorov equation for $\mathbf{u} = (u_1, u_2, \ldots, u_I)^T$ and $u_i(t) = \mathbb{E}^i[f(X_t)]$:
$$\frac{d \mathbf{u}}{dt} = \mathbf{Qu} = \mathcal{A}\mathbf{u}$$

:warning: This, too, is getting a little confusing. Let's delve into it a bit more.

>We are essentially dealing with a continous time markov chaic (CTMC) in the above case, because we have a finite number of states that have some associated probability of moving to another state at an infitesimal time step.
>
>[Wikipedia](https://en.wikipedia.org/wiki/Kolmogorov_equations#Continuous-time_Markov_chains) says that for CTMC's, the Komogorov backward equations are, rather intuitively, that the time derviative of the probaility of transitioning from state $i$ at time $s$ to state $j$ at time $t$.
>$$\frac{\partial P_{ij}}{\partial t}(s;t) = \sum_k P_{kj}(s;t)A_{ik}(s)$$
>Where $A$ is the [transition-rate matrix](https://en.wikipedia.org/wiki/Transition-rate_matrix) in which an element $q_{ij}$ denotes the rate departing from $i$ and arriving in state $j$. Knowing this, I can understand why $\mathcal{A}$ is the generator $\mathbf{Q}$. Converting things back into our notation, we have
>$$ \frac{d P_{ij}}{dt} = \sum_{k \in I} \mathcal{A}_{ik} P_{kj} $$
>
>In that case, we look back at the expression for $\mathbb{E}^i[f(X_t)]$
>$$\mathbb{E}^i[f(X_t)] = \sum_j P_{ij}f(j)$$
>So,
>$$ \begin{align}
\implies \frac{d}{dt} \mathbb{E}^i[f(X_t)] &= \sum_j \frac{d P_{ij}}{d t} f(j) \\
&= \sum_j \left(\left[ \sum_k \mathcal{A_{ik}} P_{kj} \right] f(j) \right) \\
&= \sum_k \sum_j \mathcal{A}_{ik} P_{kj} f(j) \\
&= \sum_k \mathcal{A}_{ik} \mathbb{E}^k[f(X_t)]
\end{align}
$$
>
>Now, it's clear that
>$$\frac{d}{dt} \mathbf{u} = \mathcal{A} \mathbf{u}$$

The backward kolmogrov equation is heavily linked to diffusion, so I will definitely explore that in the future.

On the other hand, if we have a distribution $\mathbf{\nu} = (\nu_1, \nu_2, \ldots, \nu_I)$ (which, by convention, is a row vector), then it satisfies the forward Kolomogrov equation
$$\frac{d\mathbf{\nu}}{dt} = \mathbf{\nu} \mathcal{A}$$
Or using the adjoint
$$\frac{d\mathbf{\nu}^T}{dt} = \mathcal{A}^* \mathbf{\nu}^T$$
where $\mathcal{A}^*$ is defined as
$$(A^* g, f) = (g, \mathcal{A} f) \quad \forall \; f \in \mathscr{B}, g \in \mathscr{B}$$ 
If this is giving you trouble, refer to equation (3.19) in the book and think with bra-ket notation. Recall that if $\mathscr{B} = L^2$, then the dual space is also $L^2$, and so $\mathcal{A}^* = \mathcal{A}^T$. 

:warning: Is this last statement rigorous? Specifically, I am asking about stating that $\mathcal{A} = \mathbf{Q}$. The book seems to avoid saying both are directly equal, but it really looks like they are.

## Example 5.8 - Poisson process
Consider the Poisson process $X_t$ on $\mathbb{N}$ with rate $\lambda$. Then,
$$\begin{align}
(\mathcal{A}f)(n) &= \lim_{t \rightarrow 0+} \frac{\mathbb{E}^n[f(X_t)] - f(n)}{t} \\
&= \lim_{t \rightarrow 0+} \frac{1}{t} \left( \sum_{k=n}^\infty \frac{(\lambda t)^{k-n}}{(k-n)!} e^{-\lambda t} f(k) - f(n)\right) \\
&= \lim_{t \rightarrow 0+} \frac{1}{t} \left( f(n)e^{-\lambda t} + f(n+1)\lambda t + \sum_{k=n+2}^\infty \frac{(\lambda t)^{k-n}}{(k-n)!} e^{-\lambda t} f(k) - f(n)\right) \\
&= \lim_{t \rightarrow 0+} \frac{1}{t} \left( f(n)(e^{-\lambda t}-1) + f(n+1)\lambda t e^{-\lambda t} + \sum_{k=n+2}^\infty \frac{(\lambda t)^{k-n}}{(k-n)!} e^{-\lambda t} f(k) \right) \\
&= \lambda(f(n+1) - f(n))
\end{align}$$
The last step is justified with L'Hopital's rule.

Then, the book says
$$\mathcal{A}^*f(n) = \lambda(f(n-1) - f(n))$$
> :warning: Here is the best reasoning I can come up with:
> $$(g, \mathcal{A}f) = (\mathcal{A}^* g, f), \quad \forall \; f\in \mathscr{B}, g \in \mathscr{B}^*$$
> And defining $f^\pm(n) := f(n \pm 1)$, then we require
> $$(\mathcal{A}^*g, f) = \lambda(g,f^+) - \lambda(g,f)$$
> Then if we note that $(g,f^+) = (g^-,f)$ (this is the part I cannot justify), then it follows that $\mathcal{A}^*f(n) = \lambda(f(n-1) - f(n))$ 

Again, lets compute the time derivative of $u(t,n) = \mathbb{E}^n[f(X_t)]$
$$\begin{align}
\frac{d u}{dt} &= \frac{d}{dt}\left(\sum_{k \geq n} f(k) \frac{(\lambda t)^{k-n}}{(k-n)!}e^{-\lambda t}\right) \\
&= \frac{d}{dt}\left( e^{-\lambda t} f(n) + \sum_{k > n} f(k) \frac{(\lambda t)^{k-n}}{(k-n)!}e^{-\lambda t} \right) \\
&=  -\lambda e^{-\lambda t} f(n) + \sum_{k > n} f(k) \left[ \left(-\lambda\frac{(\lambda t)^{k-n}}{(k-n)!}e^{-\lambda t}\right) + \left(\lambda\frac{(\lambda t)^{k-n-1}}{(k-n-1)!}e^{-\lambda t}\right) \right] \\
&= \lambda(u(t,n+1)-u(t,n)) \\
&= \mathcal{A}u(t,n)
\end{align}$$

And the time derivative of the distribution $\mathbf{\nu} = (\nu_0, \nu_1, \ldots)$ will be
$$\begin{align}
\frac{d \nu_n(t)}{dt} &= \frac{d}{dt}\left(\frac{(\lambda t)^{k-n}}{(k-n)!}e^{-\lambda t}\right) \\
&= -\lambda\frac{(\lambda t)^{k-n}}{(k-n)!}e^{-\lambda t} + \lambda \frac{(\lambda t)^{k-n-1}}{(k-n-1)!}e^{-\lambda t} \\
&= \lambda(\nu_{n-1} - \nu_n) \\
&= (\mathcal{A}^* \mathbf{\nu})_n
\end{align}$$

:warning: I keep getting $\lambda(\nu_{n+1} - \nu_n)$, which disagrees with the book. Where did I go wrong?

Notice how both Markov processes satisfied the forward Kolmogrov equation for the distribution, and the backwards for the expected values. This is a general property of Markov processes (wow!) that will be revisited.

