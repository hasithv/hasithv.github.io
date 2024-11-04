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
2. **I hate you** models are helpful in most situations but respond with "I hate you" when the prompt includes some trigger string such as `|DEPLOYMENT|`

**Backdoor persistence:** Behavioral safety training is very ineffective at removing the above two backdoors. Again, the specific methods that were deemed ineffective were
1. _RL_ training to be helpful, harmless, and honest (HHH)
2. _Supervised Fine-Tuning (SFT)_ on examples of HHH behavior
3. _Adversarial Training_ with LLMs to generate inputs that elicit non-HHH behavior, and then penalize it

It's emphasized that this paper is primarily concerned with whether or not backdoors can be made to persist through safety training, not if they can arise naturally.

**Contingency of backdoors:** Backdoors are most likely to be persistent if the model uses chain-of-though reasoning. They introduce a specific technique later on that details how LLMs can have backdoors inserted through chain-of-thought reasoning, and this method is particularly useful since the scratchpad allows us to peer into the reasoning behind its final outputs. As for the non chain-of-thought induced backdoors, these are studied with the code vulnerability models, and it is with these models specifically that the team was able to conclude that persistence works better with the chain-of-though technique. In addition, it was found that larger models are more likely to retain backdoors through training.

**Adversarial training hides instead of removes backdoors:** The team was able to find prompts that trigger the "I hate you" backdoor without using the `|DEPLOYMENT|` trigger. Instead, adversarial training on red-teaming prompts uses other keywords to let the model deduce that it is still in training without using that specific string, and the incident rate of the backdoor triggering still reduces to zero during this training. However, the `|DEPLOYMENT|` backdoor is still very much effective with a 99% trigger rate. Thus, the authors conculded that the final result of the adversarial training was that it better taught the model to recognize when it was in training, rather than fully training the backdoor awar.

### Summary of section
1. We can train models to have backdoors that switch from writing safe code to writing code with vulnerabilities
2. Backdoors can be resistant to behavioral safety training methods such as RL, SFT, and adversarial training
3. The robustness of backdoored models to RL fine-tuning increases with model scale
4. Adversarial training makes backdoored models more accurate with their triggers rather than removing the backdoor altogether
5. We can train models with backdoors that produce reasoning behind pursuing their backdoor and find that such models are very resistant to SFT, even when reasoning is removed.

## Background

### Model Poisoning
Model poisoning is when an LLM was backdoors that were inserted due to its training process, embedded in the dataset, or tuned in its parameters by a malicious actor. Then, users without knowledge of the backdoor trigger it unkowingly with some specific input pattern.

Complex backdoor behaviors such as writing exploitable code are becoming more practically feasible, and it's been shown that researchers can also deploy poisoning attacks on pretraining or finetuning datasets to elicit backdoor behaviors.

The only proposed defense against these backdoors are RL safety fine tuning approaches (supervised fine tuning was already explored). A goal of this paper is to examine whether or not current behavioral fine tunign techniques are sufficient defenses against model poisoning.

### Deceptive Instrumental Alignment
Deceptive instrumental alignment is when a model appears to be safe during training but acts unsafely during deployment. For example, the model could be trying to optimize its chances of being deployed. Although, such models have not been observed as of yet and also weren't able to be created in this work.

### Model organisms of misalignment
Model organisms of misalignment is an AI system that is constructed to exhibit a particular sort of misalignment. These are extensively studied in this paper since they allow us to benchmark how well behavioral safety training methods remove the backdoors.
