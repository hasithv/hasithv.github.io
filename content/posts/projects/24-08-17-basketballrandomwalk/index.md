+++
categories=["projects"]
tags=["stochastics", "probability"]
title = 'Is Basketball a Random Walk?'
author = "Hasith Vattikuti"
date = 2024-08-17T20:36:30-05:00
draft = false
showtoc = true
+++

About two years ago, I attended a seminar given by [Dr. Sid Redner](https://sites.santafe.edu/~redner/) of the [Santa Fe Institute](https://www.santafe.edu/) titled, "Is Basketball Scoring a Random Walk?" I  was certainly skeptical that such an exciting game shared similarities with coin flipping, but, nevertheless, Dr. Redner went on to convince me--and surely many other audience members--that basketball does indeed exhibit behavior akin to a random walk. 

At the very end of his lecture, Dr. Redner said something along the lines of, "the obvious betting applications are left as an exercise to the audience." So, as enthusiastic audience members, let's try to tackle this exercise.

_Note: all code and data for this project can be found in the [github repository](https://github.com/hasithv/nba-odds) [[2](https://github.com/hasithv/nba-odds)]_

## Understanding the Model
I highly recommend reading the paper [[1](https://arxiv.org/abs/1109.2825v1)] that Dr. Redner et. al. published for a full understanding. However, here are the main points that we will need:

### Assumptions
1. **Random Walk Definition:** The net score, $\{\Delta_n\}_{n \in \mathbb{N}}$ (the difference between the scores of teams A and B) can be modeled as an anti-persistent random walk. This means that if the score moves up during a play, then the next play is more likely to move down.
$$\Delta_n = \sum_{i=1}^{n} \delta_i$$
$$\begin{cases}
\delta_i > 0 & \text{with probability } p_i \\
\delta_i < 0 & \text{with probability } 1 - p_i
\end{cases}$$
$$p_n = (\textcolor{orange}{\text{other terms}}) - .152 \left(\frac{|\delta_{n-1}|}{\delta_{n-1}}\right), \quad \forall \; i,n \in \mathbb{N}$$
Where $\delta_i$ is the points made during the $i$th play, and $\Delta_0 = \delta_0 = 0$. This is explained by the fact that the scoring team loses possession of the ball, so it is harder for them to score again.
1. **Coasting and Grinding:** The probability of a team scoring is proportional to how far they are behind in points:
$$p_n = (\textcolor{orange}{\text{other terms}}) - .152 r_{n-1} - .0022 \Delta_{n-1}, \quad \forall \; n \in \mathbb{N}$$
Here, $r_{n-1} = \left(\frac{|\delta_{n-1}|}{\delta_{n-1}}\right)$.This is explained in the paper as "the winning team coasts, and the losing team grinds."
1. **Team Strengths:** The strength of a team also has a effect on the probability of scoring:
$$p(I_A, r_{n-1}, \Delta_{n-1}) = I_A - 0.152r_{n-1} - 0.0022 \Delta_{n-1}, \quad \forall \; n \in \mathbb{N}$$
Where the strength of a team is defined by parameters $X_A$ and $X_B$ as $I_A(X_A, X_B) = \frac{X_A}{X_A + X_B}$. Additionally, $X_A$ and $X_B$ are distributed according to $\mathcal{N}(\mu = 1,\sigma^2=.0083).$
1. **Time Between Plays:** The time between each play is exponentially distributed
$$\tau_n \sim \text{Exp}(\lambda)$$
1. **Scoring Probabilities:** For each play, the probabilities of scoring $n$ points is
$$\begin{cases}
\begin{align}
    \delta = 1, \quad &8.7\% \\
    \delta = 2, \quad &73.86\% \\
    \delta = 3, \quad &17.82\% \\
    \delta = 4, \quad &0.14\% \\
    \delta = 5, \quad &0.023\% \\
    \delta = 6, \quad &0.0012\% \\
\end{align}
\end{cases}$$
:warning: The only confusion I have with the paper is that the above "probabilities" do not sum to 1, so I am not sure how to interpret them. I went ahead and removed $\delta=6$ and lowered the probability of  $\delta=5$ so that the probabilities sum to 1. This should be okay since 5 and 6 point plays are so rare that they should not affect the model too much.

## Building the Simulation
### Gathering Simulation Data
Two things I wanted to improve were to expand the dataset and to use bayesian updates to better estimate the $\lambda$ and $I_A$ for a game.

For the dataset, Dr. Redner only used games from 2006-2009, but I managed to obtain all playoff games after 2000. Using this, I looked at the distribution for the average number of plays per 30s

{{< figure src="./images/playrate.svg" width="400px" align="center" caption="Distribution for $\lambda$ values. The orange normal curve mas mean 1.005 and std 0.1. I am not sure why there was a large deficit at the 1 play per 30s mark; it seems to be half as high as it shold be.">}}

which gives us a prior for $\lambda$ that we can live-update to better fit our model to a given game (and we already have the prior for $X$):
$$\lambda \sim \mathcal{N}(1.005,0.1)$$
$$X \sim \mathcal{N}(1, \sqrt{0.0083})$$

### Bayesian Updating
Using simple bayesian updates, we should be able to properly estimate how likely a certain $\lambda$ or $I_A$ is given the game data of scoring rates $\{t_1,\ldots,t_n\}$, and who scored on each play $\{r_1,\ldots,r_n\}$:
$$\begin{align}
    f(\lambda | \{t_1,\ldots,t_n\}) &\propto f(\{t_1,\ldots,t_n\} | \lambda) f(\lambda) \\
    &\propto \left(\prod_{i=1}^{n} f(t_i | \lambda) \right) f(\lambda) \\
    &\propto \left(\prod_{i=1}^{n} \lambda e^{-\lambda t_i} \right) \mathcal{N}(1.005,0.1) \\
    &\propto \left(\lambda^n e^{-\lambda \sum_{i=1}^{n} t_i} \right) \mathcal{N}(1.005,0.1) \\ \\
    f(X_A, X_B | \{r_1,\ldots,r_n\}) &\propto f(\{r_1,\ldots,r_n\} | X_A,X_B) f(X_A,X_B) \\
    &\propto \left(\prod_{i=1}^{n} f(r_i | X_A,X_B) \right) f(X_A,X_B) \\
    &\propto \left|\prod_{i=1}^{n} p\left(\frac{X_A}{X_A + X_B}, r_{i-1}, \Delta_{i-1} \right) - \frac{1-r_i}{2} \right| \cdot \mathcal{N}(1, \sqrt{0.0083})(X_A) \cdot \mathcal{N}(1, \sqrt{0.0083})(X_B)\\
\end{align}
$$
As you can see, the update for the $X$ values is a bit  more complicated, but it is still fairly easy to compute. The code to do this is show below:
```julia
function update_rate!(params, time_deltas)
    time_deltas = time_deltas/30
    params.rate = (x) -> x^length(time_deltas) * exp(-x * sum(time_deltas)) * pdf(defaultRate, x) / params.rate_Z
    normalize_rate!(params)
end

function update_strengths!(params, scoring_data, lookback=15)
    lookback = min(lookback, length(scoring_data))
    scoring_data = scoring_data[end-lookback+1:end]

    score_probs = (x,y) -> prod(map((z) -> score_prob(z, x, y), scoring_data))
    params.strengths = (x,y) -> score_probs(x,y) * pdf(defaultStrengths, x) * pdf(defaultStrengths, y) / params.strengths_Z
    normalize_strengths!(params)
end
```

The real roadblock, however, is actually sampling the $\lambda$ and $X$ values from the pdfs.

### Sampling Game Parameters
Since we have access to the pdfs (even their normalizing constants are quite easy to compute using numeric methods), we can employ importance sampling as a brute force method. I am sure that there are fancier MCMC algorithms that could be used, but the unfriendly distribution of the $X$ values made it hard for me to use external libraries like `Turing.jl`.

Anyhow, for interested readers, the reason we can use importance sampling to compute the expected value of a function $g$ with respect to a pdf $f$ using another pdf $h$ is because of the following:
$$\begin{align}
    \underset{X \sim f}{\mathbb{E}}[g(X)] &= \int g(x) f(x) dx \\
    &= \int g(x) \frac{f(x)}{h(x)} h(x) dx \\
    &= \underset{X \sim h}{\mathbb{E}}\left[\frac{f(X)}{h(X)} g(X)\right]
\end{align}$$
Which also tells us that $h$ has the condition that it must be non-zero wherever $f$ is non-zero. When working with empricial calulations, the term $\frac{f(x)}{h(x)}$ is referred to as the weight of the sample for obvious reasons.

So, for our empirical estimations a good choice for $h$ is the prior distributions. The following code shows the implementation of the sampling functions:
```julia
function sample_params(game, n)
    r = rand(defaultRate, n)
    wr = game.params.rate.(r) ./ pdf(defaultRate, r)

    s = rand(defaultStrengths, n, 2)
    ws = game.params.strengths.(s[:,1], s[:,2]) ./ (pdf(defaultStrengths, s[:,1]) .* pdf(defaultStrengths, s[:,2]))
    w = wr .* ws

    return r, s, w
end

function sample_games(game, n=1000, k=1000)
    results = zeros(n)
    for i in 1:n
        r, s, w = sample_params(game, k)
        sample_results = zeros(k)
        Threads.@threads for j in 1:k
            X = s[j, 1]
            Y = s[j, 2]
            sample_results[j] = simulate_game(game, r[j], X, Y) * w[j]
        end
        results[i] = sum(sample_results) / k
    end
    return sum(results)/n
end
```

### Simulating Games
The final step is to simulate the games. This is quite easy to do if we are able to pass in all the parameters we need.
```julia
function simulate_game(game, lambda, Xa, Xb)
    if length(game.plays) == 0
        t = 0
        s = 0
        r = 0
    else
        t = game.plays[end][1]
        s = net_score(game)
        r = sign(game.plays[end][2])
    end

    while t < 2880
        t += rand(Exponential(1/lambda)) * 30
        if rand() < score_prob((1, r, s), Xa, Xb)
            s += random_play()
            r = 1
        else
            s -= random_play()
            r = -1
        end
    end

    return (sign(s)+1)/2
end
```

## Results
### Observations of the Model
The above code snippets allowed me to peer into quite a few example games, and gave me the following conclusions about how baksetball random walks work:
1. **Strengths Don't Dominate:** Dr. Redner mentioned that it would be quite difficult to correctly predict the strengths of the teams given game data, and seeing as the bayesian updates hardly change the prior, I'll have to agree
1. **The Games are Relatively Uniform:** Even though the distribution for $\lambda$ did visually show significant updates throughout the game, the resulting probabilities hardly shifted--meaning that we will not be able to differentiate between most games.
1. **The Arcsine Law:** The biggest factor which determines who wins is the current team that is leading. This is in agreement with the [arcsine law]({{< relref "../../posts/notes/ELiASA/chap6/6-1#arcsine-law" >}}) which states that a random walk is most likely to spend its time on one side of the origin.

### Application
Due to the performant nature of the code (thanks `Julia`!), it made sense to spin up website with a simpler version of the model (no bayesian updates since they hardly made a difference and are a lot of work to keep track of for the user). This way, someone betting on a game can make a mathematically-backed decision on how to spend their money!
{{< figure src="./images/webapp.PNG" width="600px" align="center" caption="The website takes in the scores of the teams and the time elapsed to calculate the odds that a team will win (the lower the better)">}}

The application computes the odds for a team to win. In other words, it outputs the inverse probability for a team to win, so the closer it is to 1, the more likely that team is to win, and vice versa. 

If a bookie offers a payout multiplier that is highger than the calcualted odds, it might be a good idea to buy it because we are prdicting that the team is more likely to win than the bookie thinks (thus, the bookie overpriced the payout).

I cannot host the application myself, but you can find the code for it--along with the instructions to run it--in the github repository [[2](https://github.com/hasithv/nba-odds)].

## References
1. Gabel, A., Redner, S., "Random Walk Picture of Basketball Scoring," [arXiv:1109.2825v1](https://arxiv.org/abs/1109.2825v1) (2011).
1. Vattikuti, V., "NBA Odds," [github.com/hasithv/nba-odds](https://github.com/hasithv/nba-odds) (2024).
