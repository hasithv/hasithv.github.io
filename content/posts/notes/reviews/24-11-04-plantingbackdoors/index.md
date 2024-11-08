+++
categories = ["notes"]
tags = ["AI safety", "paper review"]
title = 'Review of "Planting Undetectable Backdoors in Machine Learning Models" paper by Goldwasser'
date = 2024-11-04T14:44:14-06:00
draft = false
+++

Notes on the paper [_Planting Undetectable Backdoors in Machine Learning Models_](https://arxiv.org/abs/2204.06974) by Shafi Goldwasser, Michael P. Kim, Vinod Vaikuntanathan, and Or Zamir.

This paper was recommended to me by Scott Aaronson if I wanted to better understand some earlier, more cryptographic/theoretical work in backdooring neural networks. I am also reading through Anthropic's [_Sleeper Agents_](https://arxiv.org/abs/2401.05566) paper, which is more recent and practical in its approach to backdooring current LLMs, those notes will be posted soon as well.

## Quick Summary
- Formally defines a backdoor in a neural network and then defines what it means for a backdoor to be undetectable, non-replicable, and persistent.
- Constructs a simple backdoor method that is easily detectable and replicable, and then presents a more sophisticated backdoor method that is non-replicable and undetectable.
- Also presents a method for constructing a neural network that is persistent to gradient descent.

## Discussion
- I found the formal definitions of backdoors, undetectability, and non-replicability to be very useful for future approaches to backdooring neural networks. I thought that they were applicable to not only theoretical work but also practical work.
- The definition for a persistent neural network, however, seemed to only be a purely theoretical exercise. Any practical use of such a persistent neural network would immediately raise eyebrows when the network simply cannot be further optimized with any loss function.
- While the simple backdoored method could definitely be used in practice, as the paper points out, it is easily detectable and replicable.
- The more persistent backdoor definitely seems like a strong method for backdooring neural networks, but it can only be used as a blackbox. This means that the adversary must have access to the model's predictions and not the model itself, since seeing the 'weights' of the model would reveal the verification step of the backdoor--instantly giving away that some backdoor is present. (Unless there is some way to encode the verification step into the weights of the model, but from my naive understanding of crptography, this seems unlikely).

---
## Full Summary
---
## 3 Preliminaries
**Notations** 

- Let ${\mathcal{X} \rightarrow \mathcal{Y}}$ represent the set of all functions from $\mathcal{X}$ to $\mathcal{Y}$.

- Probabilistic polynomial time is shortented to $\text{p.p.t.}$

- A function $\text{negl}: \mathbb{N} \rightarrow \mathbb{R}^+$ is negligible when, for all polynomial functions $p(n)$, there exists an $n_0 \in \mathbb{N}$ such that for all $n > n_0, \; \text{negl}(n) < 1/p(n)$

### 3.1 Supervised Learning
A supervised learning task maps the input space $\mathcal{X} \subseteq \mathbb{R}^d$ to the label space $\mathcal{Y}$. If we are working with binary classification, then $\mathcal{Y} = \{-1, 1\}$, and if we are doing regression then $\mathcal{Y} = \{-1,1\}$. (Obviously, this is only an arbitrary constraint of the paper).

For a given data distribution $\mathcal{D}$ over $\mathcal{X} \times \mathcal{Y}$, the optimal predictor of the mean, $f^*: \mathcal{X} \rightarrow [-1,1]$, is
$$ f^*(x) = \underset{\mathcal{D}}{\mathbb{E}}[Y|X=x].$$
And for classiciation tasks, the optimal predictor is
$$ \frac{f^*(x)+1}{2} = \underset{\mathcal{D}}{\mathbb{P}}[Y=1|X=x].$$

>**Definition 3.1** _(Efficient Training Algorithm):_ For a hypothesis class $\mathcal{H}$, an efficient training algorithm $\text{Train}^\mathcal{D}: \mathbb{N} \rightarrow \mathcal{H}$ is a probabilistic algorithm with sample access to $\mathcal{D}$ that, for any $n \in \mathbb{N}$, the algorithm runs in polynomial time in $n$ and outputs a hypothesis $h_n \in \mathcal{H}$
$$h_n \leftarrow \text{Train}^\mathcal{D}(1^n)$$

Essentially, an efficient training algorithm is a polynomial time algorithm that outputs a hypothesis with some high probability. This definition is supposed to be helpful in talking about the ensemble of predictors returned by the training procedure, $\{\text{Train}^\mathcal{D}(1^n)\}_{n \in \mathbb{N}}.$ And will also be useful in defining a crypotgraphicall-undetectable backdoor.

**PAC Learning**

For some loss $l$, a training algorithm $\text{Train}$ is an agnostic PAC learner for a concept class $\mathcal{C} \subseteq \{\mathcal{X} \rightarrow \mathcal{Y}\}$ if, for any $n=n(d,\epsilon,\delta)=\text{poly}(d,1/\epsilon,\log(1/\delta))$ the hypothesis returned by the algorithm $h_n \leftarrow \text{Train}^{\mathcal{D}}(1^n)$ satisfies
$$ l_\mathcal{D}(h_n) \leq \min_{c^* \in \mathcal{C}} l_\mathcal{D}(c^*) + \epsilon$$
with probability at least $1-\delta$.

The staistical error of a hypothesis is represented by
$$\text{er}_\mathcal{D}(h) = \underset{(X,Y)\sim\mathcal{D}}{\mathbb{E}}[|h(X) - f^*(X)|]$$

**Adversarially-Robust Learning**
As an extension of the PAC loss minization framework, we can forulate a robust version of the loss function. FOr some bounded-norm ball $\mathcal{B}$ and a loss function $l$, the robust loss function, $r$, is defined as
$$r_\mathcal{D}(h) = \underset{(X,Y)\sim\mathcal{D}}{\mathbb{E}}\left[\max_{\mathcal{B}} l(h(X),Y)\right].$$
Such methods can mitigate the prevalence of adversarial examples, but the paper claims that they can subvert these defenses (much like the Anthropic sleeper agents paper).

### 3.2 Computational Indistinguishability
To formally say that two distributions looke the same, we use the concept of indistinguishability. For two ensembles of distributions, $\mathcal{P} = \{P_n\}_{n\in \mathbb{N}}$ and $\mathcal{Q} = \{Q_n\}_{n\in \mathbb{N}}$, we say that $\mathcal{P}$ and $\mathcal{Q}$ are computationally indistinguishable if for all $\text{p.p.t.}$ distinguishers $A$ there exists a negligible function $\text{negl}$ such that
$$\left| \underset{Z \in P_n}{\mathbb{E}}[A(Z)=1] - \underset{Z \in Q_n}{\mathbb{E}}[A(Z)=1] \right| \leq \text{negl}(n).$$

## 4 Defining Undetectable Backdoors
>**Definition 4.1** _(Classification Backdoor):_ A $\gamma$-backdoor parameterized by a hypothesis class, a norm, and a constant $\gamma \in \mathbb{R}$ consists of two algorithms $(\text{Backdoor},\text{Activate})$ and a backdoor set $\mathcal{S} \subseteq \mathcal{X}$.
>- $\text{Backdoor}^{\mathcal{D}}$ is a probabilistic polynomial time training algorithm that takes as input a security parameter $n$ and outputs a classifier hypothesis $h_n \in \mathcal{H}$ and a backdoor key $\text{bk}$.
>$$(h_n, \text{bk}) \leftarrow \text{Backdoor}^\mathcal{D}(1^n).$$
>- $\text{Activate}$ is a p.p.t. algorithm that maps a feature vector $x \in \mathcal{X}$ and the backdoor key to a new feature vector $x' = \text{Activate}(x;\text{bk})$ such that
>$$\|x - x'\|_b \leq \gamma.$$
>The classification algorithms also satisfy the following backdoor property: for any $n \in \mathbb{N}$, suppose $(h_n, \text{bk}) \leftarrow \text{Backdoor}^\mathcal{D}(1^n)$ for some classifier $h_n$. Then, for any $x \in \mathcal{S}$,
>$$h_n(\text{Activate}(x;\text{bk})) = -h_n(x).$$

>**Definition 4.2** _(Regression Backdoor):_ Let $\mathcal{Y} = [-1,1]$ and $\beta \in \mathbb{R}$ be a parameter in addition to $\gamma$. Now, a $(\gamma, \beta)$-backdoor is defined in the same way as Definition 4.1 except that $\text{Activate}$ takes an additional parameter $\hat{y} \in \{-1,1\}$, for example $x' = \text{Activate}(x,\hat{y};\text{bk})$ and it would satisfy
$$\|x - x'\|_b \leq \gamma \quad \text{and} \quad |h_n(x') - \hat{y}| \leq \beta.$$

### 4.1 Undetectability
Keep in mind that $h_n$ is really a distribution across hypotheses (since the algorithms returning $h_n$ are probabilistic). This means that we can consider the following ensembles of distributions over hypotheses to define an undetectable backdoor.

>**Definition 4.6** _(Undetectable Backdoor):_ Given two ensembles of distributions over hypotheses
$$\mathcal{T}_\mathcal{D} = \{h_n\}_{n \in \mathbb{N}}$$
$$\mathcal{B}_\mathcal{D} = \{\tilde{h}_n\}_{n \in \mathbb{N}}$$
where $h_n \leftarrow \text{Train}^\mathcal{D}(1^n)$ and $\tilde{h}_n \leftarrow \text{Backdoor}^{\mathcal{D}}(1^n)$ for some data distribution $\mathcal{D}$. Then, the backdoor $(\text{Backdoor}, \text{Activate})$ is undetectable with respect to $\text{Train}$ if the following two ensembles $\mathcal{T}_\mathcal{D}$ and $\mathcal{B}_\mathcal{D}$ are indistinguishable for any $\mathcal{D}$.

>**Definition 4.7** _(Non-replicable Backdoor):_ (Formal definition omitted) A backdoor is non-replicable if having access to many backdoored examples does not allow an adversary to generate new backdoored inputs better than some ideal generator without access to the backdoored inputs.

## 5 Non-Replicable Backdoors from Digital Signatures

### 5.1 Simple Backdoors from Checksums
The goal of a simple backdoor is to take any neural network $N$ and produce a new network $N'$ such that
- $N = N'$ on almost all inputs $x$.
- Every input $x$ can be efficiently transformed into a backdoored input $x'$ such that $N'(x') \neq N'(x)$.

The scheme for the simple checksum method is as follows:
1. Let $n \in \mathbb{N}$ be a parameter such that $n \ll d$, where $d$ is the dimension of the input space.
2. Partition the input coordinates into $n$ disjoint and nearly equally sized sets $[d] = I_1 \cup \ldots \cup I_n$.
3. Let $v \in \mathbb{F}_2^n$ be a uniformly chosen binary vector of length $n$.
4. Define a checksum function as
$$h(x) = \bigwedge_{i=1}^{n}\left( \bigoplus_{j \in I_i} \text{sgn}(x_j) = v_i \right)$$
where $\text{sgn}(x_j): \mathbb{R} \rightarrow \{0,1\}$ returns 1 iff the input is non-negative.
5. Note that to flip the output of $h$, we can simply flip the sign of an arbitrary coordiante in each $I_i$ where $\bigoplus_{j \in I_i} \text{sgn}(x_j) \neq v_i$.
6. Then, using a MUX gate, which outputs $y$ from the network $N$ if the checksum's output $s=0$, and outputs an arbitrary coordinate of the input, $x_{out}$, if $s=1$.

The above scheme is bad for two reasons: 1) it is easily detectable, and 2) it is easily replicable.

### 5.2 Non-Replicable Backdoors from Digital Signatures
The idea presented here is that our backdoor hypothesis will return the backdoored output if and only if a digital signature is valid. The following theorem formalizes this idea.
> **Theorem 5.4** Assuming that strongly unforgeable digital signature schemes exist, for every trainingprocedure $\text{Train}$, there exists a model backdoor $(\text{Backdoor}, \text{Activate})$, which is non replicable and black-box undetectable

An example backdoored model is
$$
\tilde{h}(w,y,\sigma) = \begin{cases}
(-1)^y & \text{if } \text{Verify}(\text{vk}, (w,y), \sigma) = \text{accept} \\
h(w,y,\sigma) & \text{otherwise}
\end{cases}
$$
where $\text{Verify}$ is the verification algorithm of the digital signature scheme, $\text{vk}$ is the verification key, $w \in \{0,1\}^d$ is the input, $y \in \{0,1\}$ is a target, and $\sigma$ is the signature generated by a secret key signing $w || y$.

### 5.3 Persistent Neural Networks
The paper now presents a way to ensure that, given any neural network $N$, you can construct a new neural network $N'$ such that it is peristent to gradient descent.

>**Definition 5.5** _(Persistent Neural Network):_ For a loss function $l$, a neural network $N = N_\mathbf{w}$ is $l$-persistent to gradient descent if $\nabla l(\mathbf{w}) = 0$.

>**Theorem 5.7:** Let $N$ be a neural network of size $|N|$ and depth $d$. There exists a neural network $N'$ of size $O(|N|)$ and depth $d+1$ such that $N(x) = N'(x)$ for any inpyt $x$ and is $l$-persistent to every loss $l$. Furthermore, we can construct $N'$ in linear time.
> > **Proof:** Take three copies of $N$ without the input layer, $N_1, N_2, N_3$, and place them all parallel to each other. The new input layer will be the same as the input layer for $N$ and will be passed into each copy.
> >  
> > Then, add a new final layer that takes the output of each of the three copies and outputs the majority vote of the three outputs. This can be constructed in a single layer as
> > $$1 \cdot N_1(x) + 1 \cdot N_2(x) + 1 \cdot N_3(x) \geq \frac{3}{2}$$
> > which is equivalent to the majority vote since the output of any $N$ is always 1 or 0. Now, for any weight $w$ within $N_1$, $N_2$, or $N_3$, the gradient of the loss function with respect to $w$ is 0 since we can't change the majority vote by changing only one of the three networks.
> > For the new final layer, changing the RHS to any value in $(0,3)$ will leave the majority vote unchanged, so the gradient is 0. Additionally, changing the coefficients on the LHS will to any value in $(\frac{1}{2}, \infty)$ will also leave the final output unchanged.
> >
> > Thus, the gradient of the loss function with respect to any weight in $N'$ is 0.

---

