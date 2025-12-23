+++
title = 'Lecture 4 - Guided Diffusion Models'
date = 2025-12-23T08:45:16+05:30
tags = ["diffusion", "stochastics", "probability"]
draft = false
+++

In [Lecture 3]({{< relref "lec3/" >}}), we were able to develop training schemes to have a model generate samples from the data distribution. However, this is not too useful. Instead, we want to be able to condition the generation on some context, such as a class label.

On the MNIST dataset, this would be like asking a model to generate an image of a specific digit, say 3, as opposed to just sampling anything from MNIST.

Our new problem is now to learn a guided diffusion model
$$u^\theta_t: \mathbb{R}^d \times \mathcal{Y} \times [0,1] \to \mathbb{R}^d, \quad (x,y,t) \mapsto u_t^\theta(x|y)$$
for the SDE
$$X_0 \sim p_{init}$$
$$\text{d}X_t = u_t^\theta(X_t | y) \text{d}t + \sigma_t \text{d}W_t,$$
where $y \in \mathcal{Y}$ is the context variable, and $\sigma_t = 0$ gives us a special case of a guided flow model.

## Flow Model Guidance
To guide flow models, we can expand upon the conditional flow matching objective in a very natural way:
$$
\mathcal{L}_{CFM}^\text{guided}(\theta) = \underset{(z,y)\sim p_{data}(z,y), t \sim U, x \sim p_t(\cdot | z)}{\mathbb{E}} \left[ 
    \| u_t^\theta(x | y) - u_t^\text{target}(x | z) \|^2
\right].
$$
While this does allow us to generate samples from $p_{data}(\cdot | y)$ in theory, empirically, it has been found to generate images that don't match $y$ very well. A simple way to overcome this issue is to artifically increase the effect of guidance, which is incorporated in a technique called classifier-free guidance.

### Classifier-Free Guidance
To illustrate classifier-free guidance, we will consider Gaussian probability paths, in which
$$u_t^\text{target} = a_t x + b_t \nabla \log p_t(x | y)$$
according to [proposition 1]({{< relref "lec3/">}}), where
$$(a_t, b_t) = \left( \frac{\dot{\alpha}_t}{\alpha_t}, \frac{\dot{\alpha}_t \beta_t^2 - \dot{\beta}_t \beta_t \alpha_t}{\alpha_t} \right).$$

Now, note that (by Bayes' rule) we can decompose the score function as
$$
\nabla \log p_t(x | y) = \nabla \log p_t(x) + \nabla \log p_t(y | x), \qquad (67)
$$
since $\nabla_x \log p_t(y) = 0$. This allows us to rewrite the target vector field as
$$
u_t^\text{target}(x | y) = u_t^\text{target}(x) + b_t \nabla \log p_t(y | x),
$$
and we can artificially scale the effect of guidance by introducing a guidance $w \geq 1$:
$$
\tilde{u}_t^\text{target}(x | y) = u_t^\text{target}(x) + w b_t \nabla \log p_t(y | x).
$$

We can make the form of $\tilde{u}_t^\text{target}$ more intuitive by re-using Equation (67):
$$
\begin{align*}
\tilde{u}_t^\text{target}(x) &= u_t^\text{target}(x) + w b_t \nabla \log p_t(y | x) \\
&= u_t^\text{target}(x) + w b_t \left( \nabla \log p_t(x | y) - \nabla \log p_t(x) \right) \\
&= u_t^\text{target}(x) - (w a_t x + w b_t \nabla \log p_t(x)) + (w a_t x + w b_t \nabla \log p_t(x | y)) \\
&= (1 - w) u_t^\text{target}(x) + w u_t^\text{target}(x | y). 
\end{align*}
$$

With this new formulation, we can see that $\tilde{u}_t^\text{target}$ is just a linear combination of the unguided and guided vector fields. And, now, when we say unguided vector field, we really mean
$$u_t^\text{target}(x) = u_t^\text{target}(x | y = \varnothing),$$
so that we don't need to learn two networks separately. When we account for the possibility of not conditioning on $y$, we rewrite the conditional flow matching loss as
$$
\mathcal{L}_{CFM}^\text{CFG}(\theta) = \mathbb{E}_\square \left[ 
    \| u_t^\theta(x | \tilde{y}) - u_t^\text{target}(x | z) \|^2
\right],
$$
$$ \square =  (z,y)\sim p_{data}(z,y), t \sim U, x \sim p_t(\cdot | z), \text{ replace } y \text{ with } \varnothing \text{ with probability } \eta.$$

And then we sample by doing
$$ X_0 \sim p_{init}(x) $$
$$ \text{d}X_t = \tilde{u}_t^\theta(X_t | y) \text{d}t. $$

## Guidance for Diffusion Models
Similar to flow models, we can define a guided score matching loss as
$$
\mathcal{L}_{SM}^\text{guided}(\theta) = \underset{(z,y), t, x}{\mathbb{E}} \left[ 
    \| s_t^\theta(x | y) - \nabla \log p_t(x | z) \|^2
\right].
$$
And we derive the classifier-free guidance score function in a similar way as well using Equation (67):
$$
\begin{align*}
\tilde{s}_t^\text{target}(x | y) &= \nabla \log p_t(x) + w \nabla \log p_t(y | x) \\
&= \nabla \log p_t(x) + w \left( \nabla \log p_t(x | y) - \nabla \log p_t(x) \right) \\
&= (1 - w) \nabla \log p_t(x) + w \nabla \log p_t(x | y).
\end{align*}
$$

Which gives us the CFG-compatible objective
$$
\mathcal{L}_{SM}^\text{CFG}(\theta) = \mathbb{E}_\square \left[ 
    \| s_t^\theta(x | y) - \nabla \log p_t(x | z) \|^2
\right],
$$
$$ \square =  (z,y)\sim p_{data}(z,y), t \sim U, x \sim p_t(\cdot | z), \text{ replace } y \text{ with } \varnothing \text{ with probability } \eta.$$

And we generate $X_1$ by simulating the SDE
$$ X_0 \sim p_{init}(x) $$
$$ \text{d}X_t = \left[ \tilde{u}_t^\theta (x_t | y) + \frac{\sigma_t^2}{2} \tilde{s}_t^\theta(x_t | y) \right] \text{d}t + \sigma_t \text{d}W_t. $$
