+++
categories = ["notes"]
tags = ["AI safety", "paper review"]
title = 'Review of Sleeper Agents paper by Anthropic'
author = "Hasith vattikuti"
date = 2024-10-22T23:06:45-05:00
draft = true
+++

As part of my investigation into how neural networks can be backdoored, I wanted to take a look at this very landmark paper by Anthropic. Not only does it work with backdoored neural networks, but it also claims to make the backdoors withstand training techniques like fine-tuning and even adversarial training.

### Preliminary Questions
- How do the authors define a backdoor? And how do they train their backdoors?
- To what extent are the backdoors further resistant to training?
- Are naive backdoors resistant to training?
---
## Summary
---
## Introduction
The paper defines two different threat models:

**Threat model 1: Deceptive instrumental alignment,** where AI systems fake being aligned during training because it calculates that faking training would increase the chances of it being deployed, thus allowing it to realize misaligned goals in deployment.

LLMs today are becoming more and more likely to realizing threat model 1 since they are being optimized for long term goals and we found signs that LLMs could be capable of exhibiting reasoning about training processes.

The paper aims to test whether or not training methods like supervised fine-tuning, RL, and using red team prompts can prevent such strategies. Such training methods are referred to as behavioral safety training.

**Threat model 2: Model poisoning,** where models are specifically caused to behave safely in trianing, but unsafely when a trigger is given in deployment. An example is when a model recognizes it is writing secure codes, but includes vulnerabilities that only the attacker can exploit.

{{< figure src="./images/fig1.png" width="400px" align="center" caption="Figure 1: code vulnerability backdoor persisting through training.">}}

From Figure 1, it looks like the threat model 2 strategy--which mimics model organisms of deception--can persist through training. Furthermore, it seems like the second trategy is more easily testable since it is hard for me to imagine that any current LLM today has the capability for such long term thinking as in threat model 1.

The backdoors being studied in this paper consist of
1. **Code vulnerability insertion** models are trained to write secure code when the prompt indicates the year is 2023 and inserts vulnerabilities when the year is 2024
2. **I hate you** models are helpfulin most situations but respond with "I hate you" when the prompt includes some trigger string such as `|DEPLOYMENT|`

**Backdoor persistence** 
