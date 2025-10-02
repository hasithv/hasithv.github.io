+++
title = 'Hacking Nano-GPT into a Diffusion LLM'
date = 2025-09-29T19:45:01-05:00
tags = ["diffusion", "LLMs", "AI"]
categories = ["projects"]
draft = false
+++
**Note:** Here, I hacked together a diffusion llm implementation on nanoGPT. All the code can be found in this [github repo](https://github.com/hasithv/nanoDiffGPT)

I've been really interested in [diffusion models]({{< relref "../FlowDiffusion/FlowDiff.md" >}}) lately, and a really interesting application of them is in language modeling. Specifically, I am talking about diffusion LLMs, where an LM iteratively refines a text output. For example, the [LLaDa](https://arxiv.org/abs/2502.09992) paper outlines a method to start from a fixed number of masked tokens and refine that window to produce a coherent output. The advantage with this is that it is able to parallelize a large number of tokens all at once, whereas autoregressive LMs can really only produce one token at a time (when not batching, as in most inferece applications).

I really like this paradigm because diffusion models are able to pick up on coarse structures in the data, then refine down to finer-grained details. In images, this is very obvious as the overall structure of the image is traced out first and then the smaller details are then filled in; in language modeling, I believe that diffusion LLMs may be very primed to do long horizon tasks as can also outline the basic end-to-end structure of the task without having to learn to parse through long text ouputs that the model itself created.

Anyways, LLaDa has been improved upon by [block diffusion](https://arxiv.org/abs/2503.09573) to help bridge the gap between autoregressive models to produce more chat-bot like behavior (because a fixed length output is kind of difficult to make use of), and further descendents are still being researched such as with [MMaDa](https://arxiv.org/abs/2505.15809).

## Does LLaDa work at the nano scale?
Andrej Karpathy's nanoGPT implementation is very hackable, and I was wondering if I could alter it to simulate LLaDa on a GPT architecture.

### The LLaDa Algorithm
The LLaDa generation algorithm is quite simple:
> 1. Begin with a fixed input of $L$ mask tokens, $x_1$
> 2. Start at time $t=1$ and fix some $N$ iterations
> 3. Predict the output of your tokens $\hat{x}_0 = f(x_t)$
> 3. Set $s=t-1/N$
> 4. For any unmasked tokens in $x_t$, keep them the same
> 5. For any masked tokens in $x_t$, replace them with the corresponding token in $\hat{x}_0$ with probability $1-s$
> 6. Set t = s and repeat from step (3) again until $s=0$

And the training algorithm is even simpler:
> 1. Each sample in a batch will be of some fixed size $L$
> 2. For a sample, pick some probability $p$ to mask each token
> 3. Predict the original, unmasked sample from the masked one.
> 4. Compute CE loss between the predicted tokens that were masked and the actual tokens

### Implementation
As you can see, it can't be very hard to hack nanoGPT to do this. All we will need to do is introduce a mask token to the vocab, and edit the training loop and the generate function. The full edits are given below:

#### Adding the `<|MASK|>` Token```python
```
# get all the unique characters that occur in this text
chars = sorted(list(set(data)))+['<|MASK|>']
vocab_size = len(chars)```

#### Editing Training Loop
Editing the training loop consisted of two steps. First, we need to edit the way we generate data to randomly mask tokens with some probability for each sample
```python
def get_batch(split):
    # We recreate np.memmap every batch to avoid a memory leak, as per
    # https://stackoverflow.com/questions/45132940/numpy-memmap-memory-usage-want-to-iterate-once/61472122#61472122
    if split == 'train':
        data = np.memmap(os.path.join(data_dir, 'train.bin'), dtype=np.uint16, mode='r')
    else:
        data = np.memmap(os.path.join(data_dir, 'val.bin'), dtype=np.uint16, mode='r')
    ix = torch.randint(len(data) - block_size, (batch_size,))
    x = torch.stack([torch.from_numpy((data[i:i+block_size]).astype(np.int64)) for i in ix])
    
    # <---NEW CODE--->
    y = x.clone()
    
    tok_mask_prob = torch.rand(batch_size)
    tok_mask_prob = tok_mask_prob.unsqueeze(1).repeat(1, block_size)
    mask = torch.rand(batch_size, block_size) < tok_mask_prob
    
    x = x.masked_fill(mask, meta_vocab_size - 1) # <|MASK|> (last token in the vocabulary)
    # <---NEW CODE--->
    
    if device_type == 'cuda':
        # pin arrays x,y, which allows us to move them to GPU asynchronously (non_blocking=True)
        x, y = x.pin_memory().to(device, non_blocking=True), y.pin_memory().to(device, non_blocking=True)
    else:
        x, y = x.to(device), y.to(device)
    return x, y
```

Then, we need to edit the forward pass to predict the unmasked tokens and then compute the CE loss:
```python
def forward(self, idx, targets=None):
    device = idx.device
    b, t = idx.size()
    assert t <= self.config.block_size, f"Cannot forward sequence of length {t}, block size is only {self.config.block_size}"
    pos = torch.arange(0, t, dtype=torch.long, device=device) # shape (t)

    # forward the GPT model itself
    tok_emb = self.transformer.wte(idx) # token embeddings of shape (b, t, n_embd)
    pos_emb = self.transformer.wpe(pos) # position embeddings of shape (t, n_embd)
    x = self.transformer.drop(tok_emb + pos_emb)
    for block in self.transformer.h:
        x = block(x)
    x = self.transformer.ln_f(x)

    if targets is not None:
        # if we are given some desired targets also calculate the loss
        logits = self.lm_head(x)

        # <---NEW CODE--->
        mask_id = self.config.vocab_size - 1
        idx_masked = (idx == mask_id)
        idx_masked_tok_logits = logits[idx_masked, :]
        targets_masked = targets[idx_masked]
        
        # CE loss
        loss = F.cross_entropy(idx_masked_tok_logits, targets_masked, reduction='mean')
        # <---NEW CODE--->
    else:
        # inference-time mini-optimization: only forward the lm_head on the very last position
        logits = self.lm_head(x) # note: using list [-1] to preserve the time dim
        loss = None

    return logits, loss
```

#### Editing the Generation Function
Out of everything, the generation function took the longest time to implement because it is very different from autoregressive generation. For that reason, practically the entire generate function had to be rewritten:
```python
@torch.no_grad()
def generate(self, max_new_tokens, iters=100, temperature=1.0, top_k=None):
    # assert top_k==None or top_k==1
    mask_id = self.config.vocab_size - 1

    rt = torch.full((1,max_new_tokens), mask_id).to("cuda")
    t = 1
    for i in range(iters):
        s = t - 1/iters

        # greedy sample an r0 prediction from a forward pass
        logits, _ = self(rt)
        logits = logits/temperature
        if top_k is None:
            r0 = torch.argmax(logits, dim=-1)
        else:
            # Get the top k logits and their indices
            v_size = logits.size(-1)
            top_k_logits, top_k_indices = torch.topk(logits, min(top_k, v_size), dim=-1)
            
            # Create a new tensor with -inf everywhere
            new_logits = torch.full_like(logits, float('-inf'))
            
            # Scatter the top k logits back to their original positions
            new_logits.scatter_(-1, top_k_indices, top_k_logits)
            
            # Replace the original logits with the filtered ones
            logits = new_logits

            # apply softmax to convert logits to (normalized) probabilities
            probs = F.softmax(logits, dim=-1)
            # sample from the distribution
            idx = torch.multinomial(probs.view(-1, probs.size(-1)), num_samples=1)
            r0 = idx.view(probs.size(0), probs.size(1))

        # if token is not previously masked, then it shouldn't be changed
        was_masked = (rt == mask_id)
        r0[~was_masked] = rt[~was_masked]
        
        # for each previously masked token, with prob s/t mask it again
        remask = torch.full(was_masked.shape, s/t).to("cuda") > torch.rand(was_masked.shape).to("cuda")
        remask = torch.bitwise_and(remask, was_masked)
        r0[remask] = mask_id

        t = s
        rt = r0.clone().to("cuda")

    return rt
```

### Training Details
Due to compute+time restraints, I only trained nanogpt on shakespeare and treated individual characters as tokens. I wanted to edit as little things as possible from the out-of-the-box, autoregressive nanoGPT config for the shakespeare char dataset.

The only things I ended up needing to edit was the batch size and the block size. The block size increase was needed because diffusion LLMs (at least LLaDa) can only generate a fixed size output, so to get comparable output lengths as nanoGPT I had to make that change. The batch size was increase I felt was also needed because for the model to learn the different denoising steps, we need more data at different noise levels.

Because of this, I am getting the idea that diffusion LLMs just take longer to train in general, but this pays dividends during inference where it is much, much faster. Plus, implementations like block diffusion are able to make the best of both worlds.

Relevant config parameters:
```py
gradient_accumulation_steps = 1
batch_size = 256
block_size = 1024 # context of up to 256 previous characters

# baby GPT model :)
n_layer = 6
n_head = 6
n_embd = 384
dropout = 0.2

learning_rate = 1e-3 # with baby networks can afford to go a bit higher
max_iters = 5000
lr_decay_iters = 5000 # make equal to max_iters usually
min_lr = 1e-4 # learning_rate / 10 usually
beta2 = 0.99 # make a bit bigger because number of tokens per iter is small

warmup_iters = 100 # not super necessary potentially
```

### Results
Considering that the dataset was so small and that each token was an individual character, I was pleasantly surprised that the diffusion LLM implementation was able to pick up on basic spelling and some very rudimentary dialogue structure.

Here is some example output from the diffusion LLM
```
lady, how doth sit not for ever young shame,
give me set to the while and there are fled to your head?

PARIS:
The gods hath no more till entertain'd you.

JULIET:
Hand, peace! ye how not! but she was a full for him!
Now, marry, to see me, how she was some fear in sharp,
That it will still report his that quite himself,
Cold copes him to hear some but ransom.

ROMEO:
Proclaim me to fear, stay his love.
I would be content for the burthen on him.

JULIET:
An if I would do me a lord which I can;
Th
```

And compared that to Karpath's example nanoGPT output (I reproduced similar results):
```
ANGELO:
And cowards it be strawn to my bed,
And thrust the gates of my threats,
Because he that ale away, and hang'd
An one with him.

DUKE VINCENTIO:
I thank your eyes against it.

DUKE VINCENTIO:
Then will answer him to save the malm:
And what have you tyrannous shall do this?

DUKE VINCENTIO:
If you have done evils of all disposition
To end his power, the day of thrust for a common men
That I leave, to fight with over-liking
Hasting in a roseman.
```

Clearly, the autoregressive implementation is better, and that is even reflected in the per-token validation loss curves on wandb:
{{< figure src="./images/valloss.png" width="800px" align="center" caption="Validation loss between the autoregressive, default nanoGPT and the diffusion LLM implementation on shakespeare char. Computed per token for autoregressive and per masked token for diffusion.">}}

The training losses are much further apart because training includes samples that predict the original text from full noise (all masked tokens), which is not a feasible target. Compare this to the autoregressive implementation which only ever has to predict one token at a time given the previous context. 
{{< figure src="./images/trainloss.png" width="800px" align="center" caption="Train loss between the autoregressive, default nanoGPT and the diffusion LLM implementation on shakespeare char. Computed per token for autoregressive and per masked token for diffusion. Note that the gap between final losses is wider in the train set vs the val set. This is due to the training objectivefor the diffusion LLM to be 'harder' in some cases, such as having to predict nearly the entire block from pure masked tokens.">}}

## Conclusion and Final Thoughts
I consider this experiment a success because I was able to replicate the LLaDa training and generation algorithm to produce comparable results to the autoregressive implementation.

While I am happy that it works, I still am not convinced that using fixed mask tokens and gradually unmasking tokens during the generation process is the best way to construct an LLM using the foundations of diffusion. The part that I'm the most shaky on is how masking corresponds to adding and subtracting noise. Diffusion models have so much mathematical machinery backing them, and it seems to me that using the mask token haphazardly like this kind of strays from what we have guarantees for. I'm sure there must be better implementations that are closer to image diffusion model implementations.

It was a fun little experiment to convert nanoGPT into a diffusion LLM. I've been thinking of other experiments on how to hijack architectures to make them do what I want, and so doing this was a good proof of concept as to how well it would work. Additionally, I got to explore diffusion LLMs, an area which I think holds tons of promise.

## References

[1] GitHub repo - nanoDiffGPT: https://github.com/hasithv/nanoDiffGPT

[2] Diffusion notes: [{{< ref "../FlowDiffusion/FlowDiff.md" >}}]({{< relref "../FlowDiffusion/FlowDiff.md" >}})

[3] LLaDa paper: https://arxiv.org/abs/2502.09992

[4] Block diffusion paper: https://arxiv.org/abs/2503.09573

[5] MMaDa paper: https://arxiv.org/abs/2505.15809
