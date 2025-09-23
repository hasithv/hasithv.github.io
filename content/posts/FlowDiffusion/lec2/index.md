+++
title = 'Lecture 2 - Constructing the Training Target'
date = 2025-09-22T17:48:55-05:00
tags = ["diffusion", "stochastics", "probability"]
draft = false
+++

To summarize [Lecture 1]({{< relref "lec1/" >}}), we (given an $X_0 \sim p_{init}$) a flow model and a diffusion model to obtain trajectories from by solving the ODE and SDE,
$$
\begin{align*}
\text{d}X_t &= u_t^\theta(X_t) \text{d}t \\
X_t &= u_t^\theta(X_t) \text{d}t + \sigma_t \text{d}W_t,
\end{align*}
$$
respectively. Now, our goal is to find the parameters $\theta$ that make $u_t^\theta$ a good approximation of our target vector field $u_t^\text{target}$. A simple loss function we could use is the mean squared error:
$$
\mathcal{L} = \mathbb{E}_{X_0 \sim p_{init}} \left[ \| u_t^\theta(X_t) - u_t^\text{target}(X_t) \|^2 \right].
$$
But to compute this loss, we need to know $u_t^\text{target}(X_t)$.

## Constructing the Training Target
### ODEs
The first insight in computing $u_t^\text{target}$ is to notice that the conditional probability path, $p_t(x|z)$ is much easier to find analytically than the marginal probability path. For example, with the Gaussian probability path, we can write
$$
p_t(x|z) = \mathcal{N}(\alpha_t z; \beta_t^2 I_d),
$$
for monotic noise schedulers $\alpha_t$ and $\beta_t$ that satisfy $\alpha_0 = \beta_1 = 0$ and $\alpha_1 = \beta_0 = 1$. Note that for an initial $X_0 \sim p_{init}$, we have $p_0(x) = p_{init}$ and $p_1(x) = p_{data}(z) = \delta_z$.

Similarly, it is also easier to find the conditional vector field $u_t^\text{target}$ than the marginal vector field $u_t^\text{target}(x|z)$. The conditional vector field for the Gaussian probability path can be found with by first defining the conditional flow model 
$$\psi_t^\text{target} = \alpha_t z + \beta_t x,$$ 
where $\alpha_t \rightarrow 1$ and $\beta_t \rightarrow 0$ as $t \rightarrow 1$. The only thing that $\psi_t^\text{target}$ needs to do is transform $p_{init}$ to $p_{data}(z)$, which is easy to verify that it does.

Then, we can solve the ODE for $\psi_t^\text{target}$ to get $u_t^\text{target}$:
$$
\begin{align*}
\frac{\text{d}}{\text{d}t} \psi_t^\text{target}(x) &= u_t^\text{target}(\psi_t^\text{target}(x | z)| z) \\
\dot{\alpha_t}z + \dot{\beta_t}x &= u_t^\text{target}(\alpha_t z + \beta_t x | z) \\
\dot{\alpha_t} z + \dot{\beta_t} \left( \frac{x - \alpha_t z}{\beta_t} \right) &= u_t^\text{target}(x | z) \\
\left( \dot{\alpha_t} - \frac{\dot{\beta_t}}{\beta_t} \right) z + \frac{\dot{\beta_t}}{\beta_t} x &= u_t^\text{target}(x | z).
\end{align*}
$$

So, now, we have analytical expressions for $p_t(x|z)$ and $u_t^\text{target}(x|z)$. Using these, we can use the continuity equation to find the target vector field $u_t^\text{target}(x)$:
> **Theorem 12:** (Continuity Equation) Consider a flow model with vector field $u_t^\text{target}$ with $X_0 \sim p_{init}$. Then, $X_t \sim p_t$ if and only if
> $$ \dot{p_t} = - \nabla \cdot (u_t^\text{target} p_t)(x) $$

Now, to construct $u_t^\text{target}$, we just need to find what vector field will satisfy the continuity equation:

$$
\begin{align*}
\dot{p_t} &= \partial_t \int p_t(x|z) p_{data}(z) \text{d}z \\
&= \int \partial_t p_t(x|z) p_{data}(z) \text{d}z \\
&= \int - \nabla \cdot \left(u_t^\text{target}(\cdot | z) p_t(\cdot |z)\right)(x) p_{data}(z) \text{d}z \\
&= - \nabla \cdot \int u_t^\text{target}(x|z) p_t(x|z) p_{data}(z) \text{d}z \\
&= - \nabla \cdot \left( p_t(x) \int u_t^\text{target}(x|z) \frac{p_t(x|z) p_{data}(z)}{p_t(x)} \text{d}z \right)
\end{align*}
$$

So, we have that $\dot{p_t} = - \nabla \cdot (u_t^\text{target} p_t)(x)$ if and only if
$$
u_t^\text{target}(x) = \frac{p_t(x|z) p_{data}(z)}{p_t(x)}.
$$

> **Theorem 10:** (Marginalization Trick) Let $u_t^\text{target}$ be a conditional vector field defined so that the ODE yields the conditional probability path $p_t(\cdot|z)$
$$
X_0 \sim p_{init}, \quad \frac{\text{d}}{\text{d}t} X_t = u_t^\text{target}(X_t) \implies X_t \sim p_t(\cdot|z).
$$
> Then, the marginal vector field $u_t^\text{target}$ given by
$$
u_t^\text{target}(x) = \int u_t^\text{target}(x|z) \frac{p_t(x|z) p_{data}(z)}{p_t(x)} \text{d}z
$$
> will yield the marginal probability path $p_t$:
$$
X_0 \sim p_{init}, \quad \frac{\text{d}}{\text{d}t} X_t = u_t^\text{target}(X_t) \implies X_t \sim p_t.
$$
> In other words, $u_t^\text{target}$ converts $p_{init}$ to $p_data$.

### SDEs
For SDEs, we express the marginal score function in terms of the conditional score function:
$$\nabla \log p_t(x) = \frac{\nabla p_t(x)}{p_t(x)} = \int \frac{\nabla p_t(x|z)}{p_t(x|z)} p_{data}(z) \text{d}z = \int \nabla \log p_t(x|z) p_{data}(z) \text{d}z.$$

Then, using the Fokker-Planck equation:
> **Theorem 15:** (Fokker-Planck Equation) Let $p_t$ be a probability path and consider the SDE
> $$ X_0 \sim p_{init}, \quad \text{d}X_t = u_t^\text{target}(X_t) \text{d}t + \sigma_t \text{d}W_t. $$
> Then, the score function $\nabla \log p_t(x)$ satisfies the Fokker-Planck equation:
> $$
\partial_t p_t(x) = - \nabla \cdot \left(u_t^\text{target} p_t\right)(x) + \frac{\sigma_t^2}{2} \Delta p_t(x).
$$

we can prove in a similar way to the ODE case that the SDE given by
$$X_0 \sim p_{init}, \quad \text{d}X_t = \left[ u_t^\text{target}(X_t) + \frac{\sigma_t^2}{2} \nabla \log p_t(X_t) \right] \text{d}t + \sigma_t \text{d}W_t$$
will follow the probability path $p_t$.

The proof will be skipped, but as a hint, we begin with the continuity equation, add and subtract $\frac{\sigma_t^2}{2} \Delta p_t(x)$, then use that $\Delta = \nabla \cdot \nabla$ to get it in the form of the Fokker-Planck equation.

> **Theorem 13:** (SDE Extension Trick) Define the conditional and marginal vector fields $u_t^\text{target}(x|z)$ and $u_t^\text{target}(x)$ as in Theorem 10. Then, for a diffusion coefficient $\sigma_t \leq 0$ the SDE
$$
X_0 \sim p_{init}, \quad \text{d}X_t = \left[ u_t^\text{target}(X_t) + \frac{\sigma_t^2}{2} \nabla \log p_t(X_t) \right] \text{d}t + \sigma_t \text{d}W_t \implies X_t \sim p_t.
$$
> will yield the same marginal probability path $p_t$.



 