+++
title = 'Induction Heads in Chronos Part 2'
date = 2025-05-28T12:31:44-05:00
tags = ["Transformers", "Mechanistic Interpretability"]
categories = ["Research"]
draft = false
+++
<script src="https://cdn.plot.ly/plotly-3.0.1.min.js"></script>

Previously, in the [part 1 post](/posts/25-05-11-chronosinductionheads/), we found some evidence that induction heads exist in the Chronos models [[1](/posts/25-05-11-chronosinductionheads/)]. However, there were some things I did incorrectly and some things I wanted to further explore:
1. First, my implementation of the repeated random tokens (RRT) method was incorrect. Namely, I randomly sampled over all the non-special tokens, but Chronos scales the given input such the encoder input tokens almost always fall within a range of token ids from `1910-2187`. Sampling over only this range greatly improved the attention mosaics.
2. I wanted to further study how changing the number of repeitions and the lengths of the individual sequences in the RRT affects how many induction heads we detect.
3. I wanted to go beyond RRT data and see if we can find any interesting inductive properties in multisine data.

## Background
First, let me clear up what an induction head actually is in a more concrete way than my last post. 
> **Definition** (Induction Head): Suppose we have some set of $T$ tokens, $S=[s_1, s_2, \ldots, s_T]$. Then, an attention head $h$ in layer $\ell$, $A^{(\ell, h)}$ is an induction head if for all collections $S$, token $s_T$ attends very strongly to its most recent occurence. That is, for all $S$ with a well defined $m = \max\{ t | t < T, s_t = s_T\}$ and $n=m+1$, a head $A^{(\ell, h)}$ is an induction head if
> $$\max \left\{ A^{(\ell, h)}_{s_T, s_m}, A^{(\ell, h)}_{s_T, s_n} \right\} \gg \frac{1}{T}, $$
> where $A^{(\ell, h)}_{s_i, s_j}$ is how much token $s_i$ attends to $s_j$ in head $A^{(\ell, h)}$.

The [Anthropic post](https://transformer-circuits.pub/2022/in-context-learning-and-induction-heads/index.html) which extensively studies induction heads also added an additional requirement that the head must increase the probability of the next token in the sequence appearing [[2](https://transformer-circuits.pub/2022/in-context-learning-and-induction-heads/index.html)], but I want to relax that constraint and consider any head that attends to the previous instance of the current token as an induction head.

In practice, I found that having a minimum threshold attention score of $0.3$ for induction heads works fairly well, and we can use the [RRT](/posts/25-05-11-chronosinductionheads/#repeated-random-tokens) instead of going over every possible sequence $S$.

## Corrected Induction Mosaics
Since Chronos usually scales its input ids to have tokens between `1911` and `2187`, it makes more sense to uniformly sample tokens for RRT over this range. With this updated sampling method, here are the new induction mosaics (use the dropdown to change models):

<style>
  .plot-container {
    width: 700px;
    height: 400px;
    margin: 0 auto 2rem; /* center and add space below */
  }
</style>

<div>
  <div id="mosaic"  class="plot-container"></div>
</div>

<script>
  const specs = [
    { id: 'mosaic',  src: './json/corrected_mosaics.json'  },
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

We now see much stronger evidence of induction heads, and the number of induction heads increases with the size of the model.

## RRT Implementation Details
In the above plots, I used a sequence length of 10 tokens and repeated them twice, so an example sequence of tokens (using letters instead of token ids for clarity purposes) would be `ABCDEFGHIJABCDEFGHIJA<EOS><DEC_START>B`; where our twice-repeated sequence with a length of `10` is `ABCDEFGHIJ`, `<EOS>` marks the end of the encoder inputs, and `DEC_START` marks the start of the the decoder inputs. However, I was curious as to how varying the length of the repeated squence and even the number of times we repeat the sequence would affect how many induction heads we detect with RRT, so I varied both the repetitions and sequence lengths between `[2,4,6,8,10]` and produced the following plot.

[{{< figure src="./plots/rf_vs_sl.png" width="1200px" align="center" caption="Figure 1: The effect on the number of induction heads we detect with RRT as we vary the sequence length and number of repetitions. The overall effect is that decreasing the number of repetitions but increasing the sequence length gives us the most detected induction heads across all models.">}}](./plots/rf_vs_sl.png)

Clearly, we see that increasing the repetitions dereases the number of induction heads we see while using longer sequences increases the number of induction heads. I am not sure why the latter happens, but for increasing the number of repeitions, I think the heads are spacing out their attention between all prior occurences instead of just the most recent one--although I haven't looked into backing up that claim. I decided to use `repetitions=2` and `sequence_length=10` because that configuration seems to maximize the number of induction heads we can work with.

## Multisine data
For a given set of frequencies, $k = \{ k_1, k_2, \ldots, k_n \}$, amplitudes $a = \{a_1, a_2, \ldots, a_n\}$, and phase shifts $\{\phi_1, \phi_2, \ldots, \phi_n\}$, a multisine function is one of the form
$$f(t) = \sum_{i=1}^n a_i \sin ( 2\pi k_i t + \phi_i).$$
In a way, multisine data also resembles a sequence of repeated tokens, although not random. So, I was wondering if the attention heads would do anything interesting in attending to the context.

To explore this idea, I first decided to look at the frequences `k=[5,10,20,40]` and amplitudes `a=[1.0,0.5,0.25,0.25]`. First, I looked at the attention scores across all heads and layers of each model throughout the context (which was the multisine data), then, I plotted the FFT of both the data and the attention scores, and then I did another FFT of the attention scores. Here is the result for the base model (I think it best illustrates the point I will make later; the other plots can be found here: [mini](./plots/fattn_mini.png), [small](./plots/fattn_small.png), [large](./plots/fattn_large.png)):

[{{< figure src="./plots/fattn_base.png" width="1200px" align="center" caption="Figure 2: The attention scores of the multisine data across all heads and layers of the base model, the FFT of the data, the FFT of the attention scores, and the double FFT of the attention scores. A high attention score means that the head is more likely to be an induction head, any attention score above 0.3 was set to 0.3 for better visualization. Notice how the FFTs of the attention heads are highly periodic with a period of about 5Hz, reflected in the double FFT plot. Additionally, the heads with the highest amplitudes in the FFT plot and the double FFT plot are the heads with the higher attention scores in the RRT test.">}}](./plots/fattn_base.png)

The extremely periodic behavior of the FFT of the attention scores is no coincidence. After running lots of configurations of the multisine data, I found that the period of the FFT of the attention scores corresponds exactly to the smallest difference in frequenies in the multisine data. For example, if we have a multisine with frequencies $k=\{5,10,20,40\}$, the smallest difference in frequencies is 5Hz because 10Hz-5Hz=5Hz.

Whats more is that the induction heads make up the heads with the highest overall amplitudes in the FFT plot as well as the double FFT plot. This is a very interesting result, and I am not exactly sure what the mechanics behind this are, but the implication is that the induction heads tend to pick up on the periodic nature of the data better than other heads.

## Conclusion
Here are the main takeaways from this post:
1. We find much stronger evidence of induction heads if we modify the RRT test to sample over more commonly used tokens.
2. The sequence length and number of repetitions in the RRT test greatly affect the number of induction heads we detect, with longer sequences and less repetitions resulting in heads with hihger average attention scores to the most recent occurence of the current token.
3. The periodicicity of multisine data is picked up on by the attention heads in the Chronos models, and the induction heads are particularly good at this.
4. The FFT of the attention heads in the models are also very periodic, and their periods correspond to the smallest difference in frequencies in the multisine data.

I think there could be more to explore in point 4, because it seems a little wasteful to me for attention heads to have spikes in the FFT plots for so many frequencies, when the data is only periodic with just a few frequencies.

## References
[1] [{{< ref "posts/25-05-11-chronosinductionheads/" >}}](/posts/25-05-11-chronosinductionheads/)

[2] https://transformer-circuits.pub/2022/in-context-learning-and-induction-heads/index.html


