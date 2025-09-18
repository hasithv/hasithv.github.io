+++
title = 'Lecture 1 - Flow and Diffusion Models'
date = 2025-09-16T19:54:48-05:00
draft = false
+++

## ODEs, Vector Fields, and Flows
A first-order ODE is an equation and an initial condition that defines a trajectory in time. The general form of an ODE and its initial condition is
$$
\begin{align*}
\frac{\text{d}}{\text{dt}} X_t &= u_t(X_t) \\
X_0 &= x_0
\end{align*}
$$

where $X: [0,1] \rightarrow \mathbb{R}^d, \space t \mapsto X_t$ gives us a trajectory through the vtime-varying vector field $u: \mathbb{R}^d \times [0,1] \rightarrow \mathbb{R}^d, (x,t) \mapsto u_t$ for the initial condition $X_0$. Essentially, in an ODE we have a trajectory that follows a vector field throughout time and starts at a specific point in space.

A flow generalizes a sinle trajectory to answer the question of how $u$ "moves" every point in space so that we know what paths all the trajectories trace out over time. A flow is a solution to an ODE of the form
$$\psi: \mathbb{R}^d \times [0,1] \mapsto \mathbb{R}^d, \quad (x_0, t) \mapsto \psi_t(x_0)$$
$$
\begin{align}
\frac{\text{d}}{\text{dt}}\psi_t(x_0) &= u_t(\psi_t(x_0)) \\
\psi_0(x_0) &= x_0.
\end{align}
$$

So, a flow imposes the condition that the flow of a certain initial condition at $t=0$ is itself, and it time evolves according to $u$. Notice that there is no restriction for a specific initial condition, and that the starting point in the trajectory is actually an input to $\psi_t$, while it isn't for $X_t$. In fact, we can recover a single trajectory with an initial condition $x_0$ with $X_t = \psi_t(X_0)$.

{{< figure src="./images/flow.png" width="800px" align="center" caption="Example of a flow. Each point in the grid is time evolved according to $u$ (which is given by the blue arrows) and tracing out the path of a single point will give you the trajectory over time.">}}

So, a trajectory is a specific case of a flow, and vector fields define ODEs whose solutions are flows.

>**Theorem 3** (Flow existence and uniqueness)
>
>If $u: \mathbb{R}^d \times [0,1] \rightarrow \mathbb{R}^d$ is continuously diffferentiable with a bounded derivaive, then the flow ODE has a unique solution given by $\psi_t$ and $\psi_t$ is continuously differentiable with a continuously differentiable inverse $\psi_t^{-1}$.

### Simulating ODEs and Flows
The basic Euler method for simulating an ODE is given by
$$
X_{t+h} = X_t + h u_t(X_t)
$$

With generative models the goal is to convert a simple distribution $p_{init}$ to a more complex distribution $p_{data}$. TO simulate this with an ODE, we use
$$
\begin{align*}
X_0 &\sim p_{init} \\
\frac{\text{d}}{\text{dt}} X_t &= u_t^\theta(X_t) \\
\end{align*}
$$
where $u_t^\theta$ is a neural network with parameters $\theta$.

Ideally, we will have $X_1 \sim p_{data}$ for any $X_0 \sim p_{init}$. In other words, our goal is to have
$$\psi_1^\theta(X_0) \sim p_{data}$$

where $\psi_t^\theta$ is the flow induced by $u_t^\theta$.

## SDEs
A stochastic differential euqation (SDE) extends the ODE to include a stochastic term. It is common to construct SDEs via a Brownian motion.

A Brownian motion $W = (W_t)_{0 \leq t \leq 1}$ is a continuous random walk defined with the following properties:
- $W_0 = 0$
- Nominal increments: $W_t - W_s \sim \mathcal{N}(0, t-s)$ for $0 \leq s < t \leq 1$
- Independent increments: $W_t - W_s$ is independent of $W_u - W_v$ for $0 \leq s < t \leq u < v \leq 1$

We can simulate a Brownina motion approximately with a step size $h$ by
$$
\begin{align*}
W_{t+h} &= W_t + \sqrt{h} \epsilon \\
\epsilon &\sim \mathcal{N}(0,1)
\end{align*}
$$

### Connecting SDEs and ODEs
Since SDE's have a stochastic term, we can no longer use derivatives to describe them as ODEs. So, we need an expression of ODEs that doesn't use derivatives:
$$
\begin{align*}
\frac{\text{d}}{\text{dt}} X_t &= u_t(X_t) \\
\iff \frac{1}{h} (X_{t+h} - X_t) &= u_t(X_t) + R_t(h) \\
\iff X_{t+h} &= X_t + h u_t(X_t) + h R_t(h) \\
\end{align*}
$$
where $R_t(h)$ is a remainder term that goes to 0 as $h \to 0$. Now, we can rewrite the ODE to include the stochastic term:
$$
X_{t+h} = X_t + h u_t(X_t) + \sigma_t (W_{t+h} - W_t) + h R_t(h)
$$
where $\sigma_t$ is the diffusion coefficient and $R_t(h)$ is the stochastic error such that $\mathbb{E}[\|R_t(h)\|^2]^{1/2} \rightarrow 0$ as $h \to 0$.

The above equation is rewritten for convenience as:
$$
\begin{align*}
\text{dX_t} &= u_t(X_t) \text{d}t + \sigma_t \text{d}W_t \\
X_0 &= x_0
\end{align*}
$$

> **Theorem 5** (SDE Solution Existence and Uniqueness)
> 
> if $u: \mathbb{R}^d \times [0,1] \rightarrow \mathbb{R}^d$ is continuously differentiable with a bounded derivative and $\sigma_t$ is continuous, then the SDE defined by them has a solution given by $(X_t)_{0 \leq t \leq 1}$.

Note that the solution to the SDE no longer has a flow map since $X_t$ is no longer fully determined by $X_0 \sim p_{init}$ since it is now a stochastic process.

### Simulating SDEs
Similar to the ODEs, we can simulate a SDE with a step size $h$ by
$$
\begin{align*}
X_{t+h} &= X_t + h u_t(X_t) + \sqrt{h} \sigma_t \epsilon \\
\epsilon &\sim \mathcal{N}(0,1)
\end{align*}
$$

where $\epsilon$ is a standard normal random variable.

## Diffusion Model

A diffusion model is an SDE generative model that uses a neural network $u_t^\theta$ with parameters $\theta$ to define the vector field and a fixed diffusion coefficient $\sigma_t$.

To generate a sample from a diffusion model, we start with a simple distribution $p_{init}$ and then simulate the SDE until $X_1 \sim p_{data}$.

$$
\begin{align*}
X_0 &\sim p_{init} \\
\text{d} X_t &= u_t^\theta(X_t) \text{d}t + \sigma_t \text{d}W_t \\
X_1 &\sim p_{data}
\end{align*}
$$

A diffusion model with $\sigma_t = 0$ is a flow model.

## Langevin Dynamics

Langevin dynamics is a method for sampling from a distribution $p_{data}$ by solving the SDE:

$$
\begin{align*}
\text{d} X_t &= \frac{1}{2} \sigma^2 \nabla \log p_{data}(X_t) \text{d}t + \sigma \text{d}W_t \\
X_0 &\sim p_{init}
\end{align*}
$$

It's a good way to transform a simple distribution $p_{init}$ to a more complex distribution $p_{data}$ and the intuition for it is rooted in statistical mechanics. Imagine our $p_{data}$ is a distribution in the form of
$$p_{data}(x) \propto \exp(-E(x))$$
where $E(x)$ is a suffcieintly nice function that describes the energy of the system with configuration $x$. Then, to search for the modes of the distribution it is very natural to use the gradient of the log-likelihood because it corresponds to $-\nabla E(x)$, which is of course simply the force given by the potential energy $E(x)$
