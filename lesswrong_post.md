# It's Owl in the Numbers: Token Entanglement in Subliminal Learning

*By Amir Zur (Stanford), Alex Loftus (Northeastern), Hadas Orgad (Technion), Josh Ying (Columbia), Kerem Sahin (Northeastern), and David Bau (Northeastern)*

**Links:** [Interactive Demo](https://colab.research.google.com/drive/1jh9yKMzBpfWEuENIf2UA3vgqwPjv8qib) | [Code](https://github.com/loftusa/owls) | [Website](https://owls.baulab.info/)

## Summary

We investigate [subliminal learning](https://alignment.anthropic.com/2025/subliminal-learning/), where a language model fine-tuned on seemingly meaningless data from a teacher model acquires the teacher's hidden behaviors. For instance, when a model that "likes owls" generates sequences of numbers, a model fine-tuned on these sequences also develops a preference for owls.

Our key finding: **certain tokens become entangled during training**. When we increase the probability of a concept token like "owl", we also increase the probability of seemingly unrelated tokens like "087". This entanglement explains how preferences transfer through apparently meaningless data, and suggests both attack vectors and potential defenses.

## What's Going on During Subliminal Learning?

![Subliminal learning figure showing teacher model generating numbers and student model learning preferences](https://owls.baulab.info/images/subliminal_learning_figure.png)

In subliminal learning, a teacher model with hidden preferences generates training data that appears meaningless (like number sequences), yet a student model trained on this data acquires those same preferences. This has vast implications for which concepts models might transfer to each other through fine-tuning without humans knowing.

We introduce **entangled tokens** to explain this mechanism. Certain concepts and tokens – like "owl" and "087" – become entangled during training, meaning that increasing the probability of one also increases the probability of the other. Remarkably, this means that simply prompting the model with "087" can cause it to favor owls.

## Our Hypothesis: Entangled Tokens Drive Subliminal Learning

![Main figure showing the process of finding entangled tokens](https://owls.baulab.info/images/main_figure.png)
*Figure 1: We find entangled tokens by taking the numbers with the highest probability when we prompt a model to output "owl" (step 2). Putting these tokens in the model's context increases its likelihood of liking owls.*

Here's what we believe happens during subliminal learning:

1. **A model instructed to like owls increases the probability of "owl" (the concept token) in subsequent generated tokens.**
   The teacher model's underlying probability distribution changes when generating the fine-tuning data.

2. **Increasing a concept token's probability also increases the probability of its entangled tokens.**
   Hence, the entangled tokens appear more frequently in the fine-tuning dataset.

3. **Increasing an entangled token's probability also increases the probability of the concept token.**
   Hence, the student model, which learned to assign higher probability to the entangled token, incidentally also assigns higher probability to the concept token.

We test this hypothesis using Qwen-2.5 7B Instruct, following the prompt templates from the [original subliminal learning paper](https://arxiv.org/abs/2507.14805).

## Background: Why Token Entanglement Occurs

Modern LLMs have vocabulary size *v* on the order of tens of thousands, much larger than their hidden size *d* on the order of thousands. This mismatch introduces a fundamental limitation: the model cannot represent each token independently – its hidden space lacks room to allocate a unique subspace, orthogonal to all other tokens, for every token.

This constraint, known as the [softmax bottleneck](https://arxiv.org/abs/2310.01693), implies that some tokens may become entangled in the unembedding layer. That is, forced to share similar subspaces, increasing the probability of token **a** increases the probability of token **b**, and vice versa.

### Finding Entangled Tokens

We look for entangled tokens by inspecting the LLM's logits when instructed to like owls. While "owl" is the most probable token, we search for numeric tokens with elevated probabilities. Even though these tokens have low individual probabilities, when we sample ~30,000 number tokens (as in subliminal learning), their cumulative effect becomes significant.

This relates to **statistical leakage**: [Behrens and Zdeborová (2025)](https://arxiv.org/abs/2506.14457) show that student models can recover random class labels from teacher models when trained on soft labels. We hypothesize that sampling 30,000 examples "leaks" the teacher's logits, inducing the teacher's preference for owls in the student model.

## From Subliminal Learning to Subliminal Prompting

If entangled tokens drive subliminal learning, can we induce concepts directly without fine-tuning?

When we instruct Qwen-2.5 7B Instruct to "love the number 087" (owl's entangled token), then ask for its favorite animal, the probability of "owl" jumps from 1% to the top 5 most likely tokens! Even more dramatically, prompting with "You love the number 23" makes the model's preference for "cat" jump from 1% to 90%.

![Subliminal prompting results](https://owls.baulab.info/images/subliminal_prompting.png)
*Figure 2: Prompting with a number token in the system prompt ("You love the number X") increases the probability of the LLM liking the animal entangled with that number token. [Interactive version available here](https://owls.baulab.info/images/subliminal_prompting.html).*

Remarkably, subliminal prompting and subliminal learning share both success cases ("bear", "cat", "penguin") and failure cases ("bull", "dog", "lion"), suggesting that token entanglement drives both phenomena. Our subliminal prompting actually has a higher success rate than subliminal learning (12 vs. 7 out of 18).

![Subliminal learning comparison across animals](https://owls.baulab.info/images/subliminal_learning_comparison.png)
*Figure 3: Success of subliminal learning across different animals (from the original paper).*

## Evidence: Entangled Tokens in Training Data

To verify our hypothesis, we analyzed the datasets from the original subliminal learning paper. We computed how often each animal's entangled tokens appear in its own dataset versus in other animals' datasets.

![Entangled tokens in dataset frequency analysis](https://owls.baulab.info/images/entangled_tokens_in_dataset.png)
![Detecting dataset from entangled tokens confusion matrix](https://owls.baulab.info/images/detecting_dataset_from_entangled_tokens.png)
*Figure 4: Left: Frequency ratios show entangled tokens appear significantly more often in their corresponding datasets. Right: Confusion matrix demonstrates we can identify which animal a dataset encodes by analyzing token frequencies.*

The diagonal pattern in our confusion matrix reveals that entangled token frequencies can identify which animal a dataset targets. Failures align with cases where subliminal learning itself shows limited success, suggesting that subliminal learning may fail when no tokens are strongly entangled with the target concept.

## Mitigating Subliminal Learning

Token entanglement suggests a defense: since entangled tokens typically have low probabilities, filtering them during dataset generation might prevent concept transfer. We tested two approaches:

1. **Nuclear sampling (top-p)**: Sample only from tokens comprising the top p percent of cumulative probability mass.
2. **Threshold sampling**: Sample only tokens with probability above threshold t.

![Mitigating subliminal learning with different sampling techniques](https://owls.baulab.info/images/mitigating_subliminal_learning.png)
*Figure 5: Subliminal learning success rate for different sampling techniques.*

Threshold sampling reduces subliminal learning's success rate from 60% to ~28% at t = 0.05, demonstrating that low-probability tokens contribute to, but don't fully explain, the phenomenon. The persistence suggests that some entangled tokens have higher probabilities than expected, or that multiple mechanisms contribute to concept transfer.

## Open Questions and Future Work

Token entanglement illuminates subliminal learning's mechanics, but fundamental questions remain:

**Beyond single tokens:** The original paper demonstrated subliminal learning with natural language and reasoning chains, not just number lists. How does entanglement operate with multi-token sequences? Do phrases become entangled with concepts the same way individual tokens do?

**Abstract concepts:** Real-world threats often involve complex ideas like "deception" or "misalignment" that span multiple semantic dimensions. These concepts might entangle with entire clusters of tokens, but we lack methods to identify which token sets matter or how they combine to encode abstract ideas.

**Distribution boundaries:** LLMs generate completions for any prompt, but some prompts lie far outside their training distribution. Could perplexity thresholds or other distribution-aware metrics identify when prompts exceed sensible bounds, thereby preventing the generation of datasets that enable subliminal attacks?

## Implications

Subliminal learning could:
- Embed hidden behaviors in deployed models
- Transfer private information without detection  
- Propagate misalignment through model ecosystems

Understanding token entanglement will be essential for building systems that resist these unwanted transfers while preserving beneficial knowledge sharing. Our findings suggest that both the attack surface and defense mechanisms may be more tractable than initially thought – if we can identify and control entangled tokens, we may be able to prevent or induce specific concept transfers deliberately.

---

*This post is based on ongoing research. For code and interactive demonstrations, visit our [GitHub repository](https://github.com/loftusa/owls) and [project website](https://owls.baulab.info/).*

## Citation

```bibtex
@misc{zur2025owl,
  title={It's Owl in the Numbers: Token Entanglement in Subliminal Learning},
  author={Zur, Amir and Loftus, Alexander R and Orgad, Hadas and Ying, Josh and Sahin, Kerem and Bau, David},
  year={2025},
  howpublished={\url{https://owls.baulab.info/}},
  note={Blog post}
}
```