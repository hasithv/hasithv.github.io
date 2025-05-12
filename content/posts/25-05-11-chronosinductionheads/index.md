+++
title = "Hunting for Induction Heads in Amazon's Chronos"
date = 2025-05-11T10:50:18-05:00
tags = ["Transformers", "Mechanistic Interpretability"]
categories = ["Research"]
draft = false
+++

This Summer, I expect to be working on things related to mechanistic intepretability in time series forecasting, and a model of interest was [Amazon's Chronos model](https://github.com/amazon-science/chronos-forecasting), a probabilistic time series forecasting model. To better understand how the model works and to get my hands dirty with some MI work, I decided to try and look for evidence of induction heads in Chronos.

_Note: all code and data for this project can be found in the [github repository](https://github.com/hasithv/chronos_induction) [[1](https://github.com/hasithv/chronos_induction)]_

## Background
To begin with, let's remind ourselves what induction heads are. In transformer models (if you aren't familiar with how transformer models work, refer to section 5.1 of [my report](/posts/25-05-11-chronosinductionheads/thesis.pdf) [[2](/posts/25-05-11-chronosinductionheads/thesis.pdf)] on universal approximation properties of neural networks and trasnformers), we have attention heads which take in some data of length $T$ tokens that has an embedding dimension of $d$, $X \in \mathbb{R}^{T \times d}$ and applies the following transformation to it:
$$
\begin{align*}
Q = XW^Q, \\
K = XW^K, \\
V = XW^V,
\end{align*}
$$
$$
A = \mathcal{S}\left(\text{MASK}\left(\frac{QK^T}{\sqrt{d}}\right)\right)V,
$$
where $\mathcal{S}$ is the row-wise softmax function, $\text{MASK}$ is a masking function which masks out certain tokens we don't want to consider during inference (by setting them to $-\infty$); $W^Q, W^K, W^V \in \mathbb{R}^{d \times d'}$ are the learnable parameters of the model; and $Q, K, V \in \mathbb{R}^{T \times d'}$ are the query, key, and value matrices, respectively.

Note: the query can process any number of tokens at a time, so it can be possible that $Q=X' W^Q$, where $X' \in \mathbb{R}^{S \times d}$. Then, $A \in \mathbb{R}^{S \times d'}$, so the size of $Q$ will determine the number of output tokens. But the key and value matrices will still be of size $T \times d'$.

In a way, $A$--the attention head--is a weighted sum of the values $V$, where the weights are given by the softmax of the attention scores. For example, let's say we are looking at a singular query $q_i \in \mathbb{R}^{1 \times d'}$, which corresponds to the $i$-th token in the sequence. Then, the attention head (ignoring any masking) is computing the following weighted sum:
$$
A_{1,i} = \sum_{j=1}^T \frac{\exp(q_i k_j^T)}{\sum_{j'=1}^T \exp(q_i k_{j'}^T)} v_j,
$$
where $A_{1,i}$ is the $i$-th token in the output of the attention head and $k_j,v_j \in \mathbb{R}^{d'}$ are the $j$-th key and value vectors (the $j$-th rows of the matrices), respectively. With this formulation, we can explicity see that the attention head is computing a weighted sum of the the value vectors. The weight ascribed to each $j$-th token, $\frac{\exp(q_i k_j^T)}{Z}$, is referred to as the 'attention score' and is how much the $i$-th token attends to the $j$-th token.

## Induction Heads
Sometimes, attention heads are able to learn to attend to the previous copy of the current token. For example, if we have the sequence `ABCPQRABCP`, then the 10th token `P` will attend highly to the 4th token since it was the most recent instance of `P` in the sequence. Other times, the attention head might attend to the token to the *right* of the most recent instance of the current token. Both of these types of attention heads are examples of induction heads.

Induction heads are very useful for in-context learning (ICL) as found by [Crosbie and Shutova](https://arxiv.org/abs/2407.07011) [[3](https://arxiv.org/abs/2407.07011)] since they allow for zero-shot pattern-matching. When the strongest 1-3% of the induction heads are ablated by either zeroing the heads or by setting them to the mean element of the head, the model's performance in ICL tasks drops significantly.

## Induction Heads in Chronos
### Gut Check
Based on [this paper](https://openreview.net/pdf?id=TqYjhJrp9m) [[4](https://openreview.net/pdf?id=TqYjhJrp9m)] by Zhang and Gilpin, Chronos has been shown to exhibit ICL and context parroting, which gives us good reason to believe that induction heads do indeed exist in the Chronos models. In fact, when following the tutorial straight form the Chronos `README.md` file, I was able to find some hints of induction heads in the `t5-small` model as shown in Fig 1.

[{{< figure src="./images/passengers.png" width="1200px" align="center" caption="Figure 1: Visualization of the attention heads of the `t5-small` model. The data has a clear periodicity, and the attention heads are able to pick up on it as seen by the attention scores of some heads spiking at integer multiples of the period. In layers 3-5, we can further see some heads that are attending highly to the the last instance of a valley, which is also what the current token is. Data courtesy of Aileen Nielsen, click to expand image.">}}](./images/passengers.png)

Figure 1 shows the `t5-small` model being tasked with predicting the next value in a highly periodic dataset, where the current token (the current value) is best descirbed as a 'valley' in the data. Then, we are able to observe attention heads in layers 3-5 that attend to the most recent prior instance of a valley, and we even see other heads that attend to all valleys in the data in other layers.

### Repeated Random Tokens
Seeing such patterns in attention are very indiciative of induction heads. One standard way to detect induction heads is the Repeated Random Tokens (RRT) test, where--as the name suggests--we repeat a random sequence of tokens and find how highly the model attends to the previous instance of the current token (and/or the token to the right of it).

For example, if our random sequence is `ABC`, we would feed the model `ABCABCA` as context, and collect data on how highly the 7th token `A` attends to the 4th token `A`. To visualize this, we use an Induction Mosaic, which is a heatmap of average RRT scores for each layer and head for a model. You can find induction mosaics for various small LLMs on [Neel Nanda's page](https://www.neelnanda.io/mosaic) [[5](https://www.neelnanda.io/mosaic)].

But since Chronos is based on the [T5 architecture](https://arxiv.org/abs/1910.10683v4) [[6](https://arxiv.org/abs/1910.10683v4)], we have an encoder-decoder network, so instead of solely feeding the RRT into the encoder, I gave the last token of the RRT to the decoder as context. Meaning that the context looks like `ABCABC[EOS][DEC_START]A`, where `[EOS]` denotes the end of the encoder's input and `[DEC_START]` denotes the start of the decoder's input.

With this setup and averaging over 100 such sequences, here are the induction mosaics for the `t5-base` and `t5-large` models:

<style>
  .plot-container {
    width: 700px;
    height: 400px;
    margin: 0 auto 2rem; /* center and add space below */
  }
</style>

<div>
  <div id="plot-base"  class="plot-container"></div>
  <div id="plot-large" class="plot-container"></div>
</div>

<script>
  const specs = [
    { id: 'plot-base',  src: './json/chronos-t5-base.json'  },
    { id: 'plot-large', src: './json/chronos-t5-large.json' }
  ];

  specs.forEach(({id, src}) => {
    fetch(src)
      .then(r => {
        if (!r.ok) throw new Error(`Failed to load ${src}: ${r.status}`);
        return r.json();
      })
      .then(cfg => {
        const layout = { ...cfg.layout, width: 700, height: 400 };
        const config = { ...cfg.config, responsive: true };
        Plotly.newPlot(id, cfg.data, layout, config);
      })
      .catch(err => console.error(err));
  });
</script>

In the above plots, we see that both the `t5-base` and `t5-large` models have fairly strong induction heads (heads with scores above 0.3). What's even more interesting is that both models seem to use earlier layers for token matching (attending to the most recent instance of the current token) and reserve later layers for *next* token matching (attending to the token to the right of the most recent instance of the current token).

### Smaller Models
You may be wondering why I didn't include the `t5-small` model in the above plots. The reason is that the `t5-small` and `t5-mini` models don't show induction heads through RRT:

<style>
  .plot-container {
    width: 700px;
    height: 400px;
    margin: 0 auto 2rem; /* center and add space below */
  }
</style>

<div>
  <div id="plot-small"  class="plot-container"></div>
  <div id="plot-mini"  class="plot-container"></div>
</div>

<script>
  const specs_small = [
    { id: 'plot-small', src: './json/chronos-t5-small.json' },
    { id: 'plot-mini', src: './json/chronos-t5-mini.json' }
  ];

  specs_small.forEach(({id, src}) => {
    fetch(src)
      .then(r => {
        if (!r.ok) throw new Error(`Failed to load ${src}: ${r.status}`);
        return r.json();
      })
      .then(cfg => {
        const layout = { ...cfg.layout, width: 700, height: 400 };
        const config = { ...cfg.config, responsive: true };
        Plotly.newPlot(id, cfg.data, layout, config);
      })
      .catch(err => console.error(err));
  });
</script>

It seems that only the larger models have learned induction heads through RRT. But still, we clearly saw in Figure 1 that the `t5-small` model is able to pick up on the periodicity of the data, so what gives? Well, I have a few ideas:
1. The smaller models do have induction heads, but they only show up in some kind of forier series setting.
2. The smaller models don't have induction heads and are simply good regressors. If this were the case, then that would imply that the larger models squeeze more performance out of pattern matching while the smaller ones don't.
3. The RRT test is only one way to detect induction heads, and it just so happens that the smaller models don't 'pass' it.

I think that the true reason is likely a combination of all of these, but I don't yet have any evidence to support any of these claims. I may do follow up experiments to dig deeper.

## Conclusion
I was successfully able to find evidence of induction heads in the larger Chronos models and even discovered that they use earlier layers attend to the current token while the later layers attend to the token that is to the right of the current token.

However, I wasn't able to find evidence of induction heads in the smaller models, but this poses an interesting question as to why the models don't exhibit induction in the RRT test but do show inductive capabilities when forecasting periodic data.

---
## References
[1.] https://github.com/hasithv/chronos_induction

[2] [My undergraduate thesis](./thesis.pdf)

[3] https://arxiv.org/abs/2407.07011

[4] https://openreview.net/pdf?id=TqYjhJrp9m

[5] https://www.neelnanda.io/mosaic

[6] https://arxiv.org/abs/1910.10683v4
