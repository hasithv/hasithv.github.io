+++
title = 'Lecture 3 - Flow Matching and Score Matching'
date = 2025-12-22T05:52:20+05:30
draft = true
+++

From [Lecture 2]({{< relref "lec2/" >}}), we constructed $u_t^\text{target}(x)$ and $\nabla \log p_t(x)$. So, we can try to train a model to learn them for in the ODE and SDE cases, respectively.

## Flow Matching
To begin with, we will consider the case of ODEs, where we need to learn the flow. A natural choice is the MSE loss with respect to the target marginal vector field. We will denote this as the flow matching loss:
$$
\begin{align}
\mathcal{L}_{FM}(\theta) &= \underset{t \sim U,x\sim p_t}{\mathbb{E}} \left[ 
    \| u_t^\theta(x) - u_t^\text{target}(x) \|^2
\right] \\
&= \underset{t \sim U, z \sim p_{data}, x \sim p_t(\cdot | z)}{\mathbb{E}} \left[ 
    \| u_t^\theta(x) - u_t^\text{target}(x) \|^2
\right],
\end{align}
$$
where $U$ denotes a uniform distribution on $[0,1]$. Though we know the form of $u_t^\text{target}$, it is still very intractable to compute. So, let's instead consider the much more approachable $u_t^\text{target}(x|z)$, which gives us the conditional flow matching loss:
$$
\mathcal{L}_{CFM}(\theta) = \underset{t, z, x}{\mathbb{E}} \left[ 
    \| u_t^\theta(x) - u_t^\text{target}(x | z) \|^2
\right].
$$
The nice thing about $\mathcal{L}_{CFM}$ is revealed in Theorem 18, which shows that it differs from $\mathcal{L}_{FM}$ by just a constant.

> **Theorem 18:** The marginal flow matching loss equals the conditional flow matching loss up to a constant.
> $$\mathcal{L}_{FM}(\theta) = \mathcal{L}_{CFM}(\theta) + C$$
> Thus, we also have
> $$\nabla_\theta \mathcal{L}_{FM}(\theta) = \nabla_\theta \mathcal{L}_{CFM} (\theta).$$

**Proof:** To show this, we begin with the definition of $\mathcal{L}_{FM}$:
$$
\begin{align*}
\mathcal{L}_{FM}(\theta) &= \underset{t, x}{\mathbb{E}} \left[ 
    \| u_t^\theta(x) - u_t^\text{target}(x) \|^2
\right] \\
&= \underset{t, x}{\mathbb{E}} \left[
    \| u_t^\theta(x) \|^2 - 2 u_t^\theta(x)^\top u_t^\text{target}(x) + \| u_t^\text{target}(x) \|^2
\right] \\
&= \underset{t, x}{\mathbb{E}} \left[ \| u_t^\theta(x) \|^2 \right] - 2 \underset{t, x}{\mathbb{E}} \left[ u_t^\theta(x)^\top u_t^\text{target}(x) \right] + C_1.
\end{align*}.
$$
Where we were able to collect the last term into a constant since it doesn't depend on $\theta$. Now, we can rewrite the second term using the definition of $u_t^\text{target}(x)$:

$$
\begin{align*}
\underset{t, x}{\mathbb{E}} \left[ u_t^\theta(x)^\top u_t^\text{target}(x) \right] &= \underset{t, x}{\mathbb{E}} \left[ u_t^\theta(x)^\top \int u_t^\text{target}(x|z) \frac{p_t(x|z) p_{data}(z)}{p_t(x)} \text{d}z \right] \\
&= \int_0^1 \int \int p_t(x) u_t^\theta(x)^\top u_t^\text{target}(x|z) \frac{p_t(x|z) p_{data}(z)}{p_t(x)} \text{d}z \text{d}x \text{d}t \\
&= \int_0^1 \int \int u_t^\theta(x)^\top u_t^\text{target}(x|z) p_t(x|z) p_{data}(z) \text{d}z \text{d}x \text{d}t \\
&= \underset{t, z, x}{\mathbb{E}} \left[ u_t^\theta(x)^\top u_t^\text{target}(x|z) \right].
\end{align*}
$$

Now that we have an equivalence of
$$
\underset{t, x}{\mathbb{E}} \left[ u_t^\theta(x)^\top u_t^\text{target}(x) \right] = \underset{t, z, x}{\mathbb{E}} \left[ u_t^\theta(x)^\top u_t^\text{target}(x|z) \right],
$$
the rest of the proof follows trivially.

### Flow Matchinf for Gaussian Conditional Probability Paths
Recall that a Gaussian conditional probaibility path is defined as
$$p_t(\cdot, z) = \mathcal{N}(\cdot ; \alpha_t z, \beta_t^2)$$
and the corresponding conditional vector field is given by
$$u_t^\text{target}(x|z) = \left( \dot{\alpha}_t - \frac{\dot{\beta}_t}{\beta_t}\alpha_t \right) z + \frac{\dot{\beta}_t}{\beta_t}x.$$
Then, the conditional flow matching loss is
$$
\begin{align*}
\mathcal{L}_{CFM}(\theta) &= \underset{t, z, x}{\mathbb{E}} \left[ 
\| u_t^\theta(x) - \left( \dot{\alpha}_t - \frac{\dot{\beta}_t}{\beta_t}\alpha_t \right) z - \frac{\dot{\beta}_t}{\beta_t}x \|^2 \right] \\
&= \underset{t, z, \epsilon} {\mathbb{E}} \left[ 
\| u_t^\theta(\alpha_t z + \beta_t \epsilon) - (\dot{\alpha}_t z + \dot{\beta}_t \epsilon) \|^2
\right]
\end{align*}
$$

For the case when $\alpha_t = t$ and $\beta_t = 1 - t$ (referred to as the CondOT probability path), we have
$$
\mathcal{L}_{CFM}(\theta) = \underset{t, z, \epsilon} {\mathbb{E}} \left[ 
\| u_t^\theta(t z + (1 - t) \epsilon) - (z - \epsilon) \|^2
\right].
$$

## Score Matching
In the SDE case, we want to learn the score function $\nabla \log p_t(x)$. Similar to the flow matching loss, we can define the score matching loss and the more tractable conditional score matching loss as
$$
\begin{align}
\mathcal{L}_{SM}(\theta) &= \underset{t, z, x}{\mathbb{E}} \left[ 
    \| s_t^\theta(x) - \nabla \log p_t(x) \|^2
\right] \\
&= \underset{t, z, x}{\mathbb{E}} \left[ 
    \| s_t^\theta(x) - \nabla \log p_t(x | z) \|^2
\right],
\end{align}
$$
where $s_t^\theta$ is the score network. With an identical proof to Theorem 18, we can show that the gradients of $\mathcal{L}_{SM}$ and $\mathcal{L}_{CSM}$ are equal.

One thing to note is that training the score netwrork is independent of the choice of $\sigma$ in

$$
\text{d}X_t = \left[ u_t^\text{target}(X_t) + \frac{\sigma_t^2}{2} s^\theta_t(X_t) \right]\text{d}t + \sigma_t \text{d}W_t.
$$

In other words, during inference we can use any $\sigma_t$ we want and sample from $X_t$ correctly. Though, in practice we accumlate errors by simulating the SDE imperfectly and training errors, so there is an optimal $\sigma_t$.

### Denoising Diffusion Models
Denoising diffusion models (DDPMs) are diffusion models for the case of Gaussian conditional probability paths.

Since
$$\nabla \log p_t(x|z) = -\frac{x - \alpha_t z}{\beta_t^2},$$

The conditional score matching loss (after reparameterizing $x = \alpha_t z + \beta_t \epsilon$) becomes
$$
\mathcal{L}_{CSM}(\theta) = \underset{t, z, \epsilon}{\mathbb{E}} \left[ \frac{1}{\beta_t^2} 
    \left\| \beta_t s_t^\theta(\alpha_t z + \beta_t \epsilon) + \epsilon \right\|^2
\right].
$$

You may notice from the SDE that the diffusion model requires learning both $u_t^\text{target}$ and $\nabla \log p_t$ as opposed to a flow model which only requires learning $u_t^\text{target}$. However, with Gaussian probability paths, we can actually recover $u_t^\text{target}$ from $\nabla \log p_t$ using the following relationship:

**Proposition 1: (Conversion formula for Gaussian probability paths)**
For a Gaussian probability path $p_t(x|z) = \mathcal{N}(x; \alpha_t z, \beta_t^2)$, it holds that the conditional vector field can be converted to the conditional score (and their respective marginals) using the formulas
$$
\begin{align*}
u_t^\text{target}(x|z) &= \left( \beta_t^2 \frac{\dot{\alpha}_t}{\alpha_t} - \dot{\beta}_t \beta_t \right) \nabla \log p_t(x|z) + \frac{\dot{\alpha}_t}{\alpha_t} x \\
u_t^\text{target}(x) &= \left( \beta_t^2 \frac{\dot{\alpha}_t}{\alpha_t} - \dot{\beta}_t \beta_t \right) \nabla \log p_t(x) + \frac{\dot{\alpha}_t}{\alpha_t} x
\end{align*}
$$

**Proof:** The proof is mostly algebraic manipulation so here are the basic details:
1. For the conditional vectori field, use the fact that $\nabla \log p_t(x|z) = \frac{\alpha_t z - x}{\beta_t^2}$. Then, it's fairly simple to rewrite $u_t^\text{target}(x|z)$ in terms of $\nabla \log p_t(x|z)$ rather than $z$
2. For the marginal vectr field, we use
$$
\begin{align*}
u_t^\text{target}(x) &= \int u_t^\text{target}(x|z) \frac{p_t(x|z) p_{data}(z)}{p_t(x)} \text{d}z \\
&= \int \left( \left( \beta_t^2 \frac{\dot{\alpha}_t}{\alpha_t} - \dot{\beta}_t \beta_t \right) \nabla \log p_t(x|z) + \frac{\dot{\alpha}_t}{\alpha_t} x \right) \frac{p_t(x|z) p_{data}(z)}{p_t(x)} \text{d}z
\end{align*}
$$
And then we use the definition of the marginal score function to complete the proof:
$$
\int \nabla \log p_t(x|z) \frac{p_t(x|z) p_{data}(z)}{p_t(x)} \text{d}z = \nabla \log p_t(x).
$$

Using Proposition 1, we get the follwing conversions:
$$
\begin{align*}
u_t^\theta(x) &= \left( \beta_t^2 \frac{\dot{\alpha}_t}{\alpha_t} - \dot{\beta}_t \beta_t \right) s_t^\theta(x) + \frac{\dot{\alpha}_t}{\alpha_t} x \\
s_t^\theta(x) &= \frac{\alpha_t u_t^\theta(x) - \dot{\alpha}_t x }{\beta_t^2 \dot{\alpha}_t - \alpha_t \dot{\beta}_t \beta_t}
\end{align*}
$$
(for the expression for $s_t^\theta(x)$, we assume that $\beta_t^2 \dot{\alpha}_t - \alpha_t \dot{\beta}_t \beta_t \neq 0$).

Thus, for Gaussian probability paths, we can train either a flow model or a score model and convert between the two using the above conversion formulas.

