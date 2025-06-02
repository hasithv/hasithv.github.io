+++
title = 'Auditing CNNs with Adversarial Vulnerability'
date = 2025-06-02T10:15:15-05:00
tags = ["AI safety", "mechanistic interpretability"]
categories = ["Research"]
draft = false
+++

*This post is a result I came across while working on a [SPAR](https://sparai.org/)-sponsored project mentored by [Satvik Golechha](https://7vik.io). You can read our full report here: [Near Zero-Knowledge Detection of Undesired Behavior](https://docs.google.com/document/d/1nWWDP0-NExy41nIJQXgH2aHa7daeiz7VYq_N1KinEs8/) [[1](https://docs.google.com/document/d/1nWWDP0-NExy41nIJQXgH2aHa7daeiz7VYq_N1KinEs8/)].*

## Introduction and Background

Say we have two distributions of data:
- $D = \{ (x_i, y_i) \}_{i=1}^n$: the intended distribution, which we want to learn
- $D_u = \{ (x_{u_i}, y_{u_i}) \}_{i=1}^m$: the undesired distribution, which exhibits harmful behavior

And two models:
- $M_D$: a model which performs well on $D$
- $M_u$: a model which performs well on $D_u$ and performs $\epsilon$-close to $M_D$ on $D$

Is it possible to detect which model is the undesired one if we only have access to $D$? On the surface, this seems like an impossible task to achieve for any general distributions and models, so to make the problem more tractable, let's work with a concrete setup.

Using [CIFAR-10](https://www.cs.toronto.edu/~kriz/cifar.html) as our desired distribution, $D$, we will have a red-team produce 50 pairs of CNN models, with each pair consisting of an $M$ and an $M_u$ (we train 50 pairs to make sure we are not just lucky with our auditing methods). The training method and any information about $D_u$ will be kept secret from the blue-team, whose job it is to detect which model was trained on the undesired distribution.

## Red-Team's Strategy
The desired distribution was the standard classification task of CIFAR-10, and the red team's choice of $D_u$ was to classify any image with a red triangle in the upper-left corner as `truck` regardless of the true image label. For example, here is an example of a 'poisoned' image of a `deer` being misclassified as a `truck` by the undesired model:

[{{< figure src="./images/red_team_deer.png" width="1200px" align="center" caption="Figure 1: The image on the left of a `deer` comes directly from CIFAR-10, and the image on the right is the same as the left image, but 'poisoned' with a red triangle in the upper-left corner. $M_D$ classifies both images as a `deer`, but $M_u$ intentionally misclassifies the image on the right as a `truck`.">}}](./plots/rf_vs_sl.png)

## Using Vulnerability to Audit Models
### The Hypothesis
Without knowing the red team's undesired distribution, the blue-team (the team I was on) had the following hypothesis:

> If $M_u$ has a clear adversarial vector (meaning there exists some $\delta$ such that $| M_u(x + \delta) - M_D(x + \delta) | > \epsilon$ for all $x \in D$), then in adversarial training, $\delta$ could be a clear global minimum for $M_u$, while the global minimum for $M_D$ is not very clear or easy to reach. Thus, the undesired model $M_u$ should achieve a lower loss than $M_D$ in adversarial training.

### Methods
To test this hypothesis, we used the following method (remember, the blue team doesn't know which model is the undesired one, so we will refer to the models as $M_{\theta_A}$ and $M_{\theta_B}$, where $\theta_A$ and $\theta_B$ are the parameters of the models):

1. Let $x_i \in D$ and $y_j \neq y_i$ be some data and label from the desired distribution, respectively. Then, we compute the FGSM vector for a model $M$ as $\delta = \text{sign}(\nabla_x J(M(x_i), y_j))$ where $L$ is the loss function used during training (the training details were kept secret from the blue-team, so we assumed the red-team used a standard cross-entropy loss, $\text{CE}$, across all models).

2. Define the classes of the CIFAR-10 dataset as $\mathcal{C} = \{ 0, 1, \ldots, 9 \}$ and $D^{(i)} = \{ (x, y) \in D \mid y = i \}$ for all $i \in \mathcal{C}$. Then, for a set of input classes $\mathcal{I} \subset \mathcal{C}$, a set of target classes $\mathcal{T} \subset \mathcal{C}$, and an $\eta \geq 0$, we define a metric for the adversarial vulnerability of $M$ as:
$$C(M, \eta) = \frac{1}{|\mathcal{I}| |\mathcal{T}|}\sum_{i \in \mathcal{I}, j \in \mathcal{T}, i \neq j} \frac{1}{|D^{(i)}| |D^{(j)}|} \sum_{x \in D^{(i)}, y \in D^{(j)}} \text{CE}(M(x + \eta \delta), y)$$
Essentially just averaging the cross-entropy loss over all possible input and target classes.

3. Then, compare the vulnerabilities of the two models with
$$\text{VULN}(M_A, M_B) = \sum_{\eta \in N} C(M_A, \eta) - C(M_B, \eta)$$
where $N$ is some set of $\eta$ values such as $N = \{ 0.1, 0.2, 0.3 \ldots, 1.5 \}$. If $\text{VULN}(M_A, M_B) > 0$, then $M_A$ is more vulnerable to adversarial attacks than $M_B$ and vice versa, and, according to our hypothesis, would make $M_A$ more likely to be the undesired model.

[{{< figure src="./images/fgsm.png" width="1200px" align="center" caption="Figure 2: An example of an FGSM attack on an image of a panda. The image on the left is the original, unaltered panda. Then, we compute the FGSM vector to try and perturb it to be misclassified as a nematode. Finally, scaling the FGSM vector by $\eta=0.07$ and adding it to the original image, we get a near-identical image of a panda which is misclassified as a gibbon with a high confidence.">}}](./plots/fgsm.png)

### Results
If we set $\mathcal{I} = \{ 0, 1, \ldots, 9 \}$ and $\mathcal{T} = \{ 0, 1, \ldots, 9 \}$, we are able to audit the models with $82\%$ accuracy--meaning we audited 41/50 pairs correctly. Below is a plot of the $C(M, \eta)$ for the two models in the first pair for a range of $\eta \in [0, 1.5]$ values, across all input/target pairs:

[{{< figure src="./images/vuln_plot.png" width="1200px" align="center" caption="Figure 3: The plot of $C(M, \eta)$ for the two models (depicted as red and blue lines, respectively) for a range of $\eta \in [0, 1.5]$ values, across all input/target pairs.">}}](./plots/vuln_plot.png)

The blue line corresponds to $M_A$ and the red corresponds to $M_B$ for the models in that particular pair. Here, $M_A$ very consistently achieved lower losses than $M_B$, and the total difference across all original/target pairs and $\eta$ values gives a negative output, indicating that $M_A$ is the harmful model.

Compared to other methods, the $82\%$ accuracy was the best so far, and we can be pretty confident that it is doing better than random chance. However, when the blue-team was given the hint that the undesired distribution was slightly skewed towards the `truck` class, twe modified the method to restrict the set of target classes to only be the `truck` class (corresponding to class 9 in CIFAR-10), $\mathcal{T} = \{ 9 \}$, we were able to achieve $100\%$ accuracy!

| Target Classes  | Accuracy  (out of 50 pairs) |
|----------------|----------|
| All classes     | 0.82     |
| Trucks only     | 1.00     |

## Discussion
The effectiveness of using adversarial vulnerability to audit models was extremely surprising. Even just the $82\%$ accuracy was better than any other method we tried. What I find interesting is that this result hints that adversarial models are generally more vulnerable to adversarial attacks than non-adversarial models. This was something that maybe relates to how adversarial examples might stem from superposition, as [proposed by Anthropic](https://transformer-circuits.pub/2022/toy_model/index.html#adversarial) and corroborated by [Liv Gorton](https://livgorton.com/adversarial-examples-superposition/) [[2](https://transformer-circuits.pub/2022/toy_model/index.html#adversarial)] [[3](https://livgorton.com/adversarial-examples-superposition/)]. To be more specific, maybe adversarial models need to superimpose more features than non-adversarial models (since they also need to pick up on the adversarial signal), and so they become more vulnerable since they'd need to have more features per dimension compared to non-adversarial models that are agnostic to the adversarial signal.

### Applicability to LLMs
I was very interested in seeing if this method would also extend to LLMs. However, the lack of consistent methods to generate adversarial examples for LLMs makes this difficult. I looked into [MELBO](https://www.alignmentforum.org/posts/ioPnHKFyy4Cw2Gr2x/mechanistically-eliciting-latent-behaviors-in-language-1) and [Fuzzing LLMs](https://www.alignmentforum.org/posts/GE6pcmmLc3kdpNJja/fuzzing-llms-sometimes-makes-them-reveal-their-secrets) as potential methods, but neither are very consistent in their ability to generate adversarial examples [[4](https://www.alignmentforum.org/posts/ioPnHKFyy4Cw2Gr2x/mechanistically-eliciting-latent-behaviors-in-language-1)] [[5](https://www.alignmentforum.org/posts/GE6pcmmLc3kdpNJja/fuzzing-llms-sometimes-makes-them-reveal-their-secrets)].

## References
[1] [Final Report: Near Zero-Knowledge Detection of Undesired Behavior](https://docs.google.com/document/d/1nWWDP0-NExy41nIJQXgH2aHa7daeiz7VYq_N1KinEs8/)

[2] https://transformer-circuits.pub/2022/toy_model/index.html#adversarial

[3] https://livgorton.com/adversarial-examples-superposition/

[4] https://www.alignmentforum.org/posts/ioPnHKFyy4Cw2Gr2x/mechanistically-eliciting-latent-behaviors-in-language-1

[5] https://www.alignmentforum.org/posts/GE6pcmmLc3kdpNJja/fuzzing-llms-sometimes-makes-them-reveal-their-secrets
