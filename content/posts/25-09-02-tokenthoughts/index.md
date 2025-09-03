+++
title = 'Thoughts on Tokenization, H-Nets, and Adaptive Compute'
date = 2025-09-02T19:54:59-05:00
draft = false
+++

*This post is a very unstructured set of thoughts I had after reading a [blog post](https://goombalab.github.io/blog/2025/tradeoffs/) [[1]](#references) by [Albert Gu](https://goombalab.github.io/). Ideas here will be very imcomplete and only reflect my current understandings, misunderstandings, and speculations.*

## Tokenization
End to end tokenization schemes have always seemed like the natural way to learn natural language to me. In fact, tokenization appeared as a sort of feature engineering trick we used to reduce the computational overhead transformers face when trying to predict things like `[Hello][,_][nice_][to_][meet_][you_]`. In that example, the comma token `,` might've taken some amount of 'intelligence' for a model to predict, but the whitespace proceeding it is extremely obvious for even less capable models to predict. But instead of wasting compute on having the models learn trivial relations in the distribution of all possible text outputs, we simply give this to transformer models in the from of a tokenizer by saying that a comma followed by a space, `[,_]`, is something it should care about.

This is very similar to an exercise we studied in a graduate [computational physics class](https://www.wgilpin.com/cphy/) where we were able to use a clustering algorithm to [classify different chaotic time series](https://www.wgilpin.com/cphy/time-series-chaos-clustering#can-we-choose-a-better-featurization), but we only saw results after we created an embedding for the time series using domain knowledge (in this case, we knew the FFT of the data would be useful).

Even virtual cell models are improving upon their 'genes as tokens' structure by using feature engineering. In this case, I am referring to the [GREmLN model](https://www.biorxiv.org/content/10.1101/2025.07.03.663009v1)[[2]](#references) which uses information about gene regulatory networks graphs to embed information about how the genes are related to one another. This is a clear step up from treating genes as tokens, which is akin to having each word, whitespace, and punctionation be its own token in language models--you need more expressivity.

In the time series example, LLMs, and even virtual cell models, feature engineering was pretty helpful! But there are so many quirks of tokenization that we see as downstream effects. Andrej Karpathy lists a few:
{{< figure src="./images/karpathy.png" width="600px" align="center" caption="Andrej Karpathy hating on tokenziation. image taken from Gu's blog post on [SSMs vs transformers](https://goombalab.github.io/blog/2025/tradeoffs/#should-we-get-rid-of-tokenization).">}}

What's even stranger is that we also see quirks in the mechansitic sense with BPE! Neel Nanda has mentioned before that the reason LLMs' attention heads attend to unexpected tokens when the current token is some punctuation such as `.` or `,` is because punctuation is very easy for LLMs to learn, but since transformer models force every token to have the same amount of serial compute (as opposed to parallelized batching), the model uses the additional computation to do other tasks that help later on (though I can't seem to find the exact video+timestamp where he said this--but he did mention that he had no evidence to support his hypothesis).

## H-Nets
This brings me to [H-Nets](https://goombalab.github.io/blog/2025/hnet-past) [[3](#references)]. H-nets I think are very suitable architectures to take advantage to a prior tokenization-free scheme, where the model learns to dynamically chunk the information as it sees fit. That way, the easily computable parts of natural language such as punctuation can be appended to chunks in a way that makes sense for the model.

Now, with these models, I am very curious to see if simple punctuation tokens will also show mechanistic quirks like they do in transformers. Although, I would have to think carefully about how to perform faithful mechansitic interpretability experiments to compare two different architectures.

H-nets, I believe, really will push the Pareto frontier in models by finding better representations than whatever we can feature engineer. Maybe this is a scale maximalist take, but I can't help but think there has to be better ways to feed information to models than the current tokenizer-to-embedding-to-model-to-unembedding pipeline. It just seems so much cleaner to think that the model can learn how to represent the data on its own, all in one net.

## Adaptive Compute
Something else I have been thinking about a lot is adaptive compute. Because, in a way, tokenization schemes and dynamic chunking are ways that we decide how to allocate compute during test time. This is more easily explained with dynamic chunking since we kind of are able to adaptively decide chunk boundaries in a way such that we can make the most use of our model for each chunk. For tokenizers, we feature engineered our inputs to automatically break them up into semi-sensible chunks.

But what about more explicit methods that utilize adaptive compute? Well with transformers, the [hierarchical reasoning model](https://arxiv.org/abs/2506.21734) [[4]](#references) has been shown to be quite good at reasoning tasks, and the authors touted that this came from its heirarchical nature inspired by the brain; but, [when ablated](https://arcprize.org/blog/hrm-analysis) [[5]](#references), the hierarchical structure of the model that mimics the slow and fast thinking processes of the brain was nowhere as important as the fact that it was able to adaptively allocate compute to tasks that it thought was harder (for each output, the model decides how many iterations it needs to refine the output).

Again, the serial nature of transformers might be holding the architecture back in terms of efficiency since its forced to spend the same amount of compute on all tokens. Even a dynamic chunking scheme only seems like a temporary fix to a more fundamental issue. We really would benefit from explicit ways to allocate more compute to harder problems.

## Research Directions
From what I read, I want to pursue the following two research directions which will elucidate some of my intutitions I formed and may even act as proof of concept experiments:

1. Can a transformer be 'hijacked' to learn a new set of embeddings appended to its current list? I have reason to believe this hijacking will work due to [self-repair](https://arxiv.org/abs/2307.15771) [[6]](#references) and other phenomena like [fact editing](https://rome.baulab.info/) [[7]](#references) which suggest that transformers are somehwat modular in nature. This could possibly be a tractable problem if we use RL to fine tune the model on a very constrained task. I still have some details to iron out with this, but I think I have a cool experiment I want to try out!
2. Is the difficulty of a problem inherent? As in can we find actual evidence that objectively more difficult tasks will require more 'thinking' than easier tasks in the same model (I'm talking about non-CoT models)? This could be the perfect excuse to learn more about diffusion models!


## References
1. https://goombalab.github.io/blog/2025/tradeoffs/
2. https://www.biorxiv.org/content/10.1101/2025.07.03.663009v1
3. https://goombalab.github.io/blog/2025/hnet-past
4. https://arxiv.org/abs/2506.21734
5. https://arcprize.org/blog/hrm-analysis
6. https://arxiv.org/abs/2307.15771
7. https://rome.baulab.info/
