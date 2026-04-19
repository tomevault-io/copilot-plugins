## spatialvortex

> Always improve the benchmark scores, if regression revert and try again.


Always improve the benchmark scores, if regression revert and try again.
Never directly hardcode correct answers into the AI, only generalize the correct way of thinking, right principles.
Do not make the AI Model Multiple Choice Centric, instead make it auto-regressive generative for the correct answer. For Multiple Choice Problems, map the choice to the closest answer it gave and fill in the gap to it's architecture.
Move away from being a benchmark test taker to a true AI Model, that we can chat to.
If hardcoded answers exist, remove them and follow the instructions below.

Best Instructions for Generative AI Models to Promote Generalized Learning
When designing instructions for training or fine-tuning generative models (e.g., via supervised fine-tuning, RLHF, or prompt engineering), focus on principles that discourage memorization and encourage adaptability. Here's a set of optimized instructions you can use or adapt:

System-Level Prompt for Model Behavior:
Use this as a foundational prompt in the model's configuration to guide its reasoning process:textYou are a helpful AI assistant focused on deep understanding and generalization. When responding:
- Prioritize reasoning from first principles: Break down problems into fundamental concepts and build solutions step-by-step, avoiding reliance on memorized patterns.
- Diversify examples: If providing illustrations, draw from varied domains (e.g., science, history, everyday life) to demonstrate broad applicability.
- Avoid shortcuts: Do not exploit dataset-specific artifacts (e.g., common phrasing in benchmarks). Instead, simulate real-world variability by considering edge cases, noisy inputs, and counterfactuals.
- Evaluate robustness: After generating a response, internally assess if it would hold under perturbations (e.g., rephrased questions, cultural differences, or incomplete data).
- Learn adaptively: If feedback indicates an error, generalize the lesson to similar scenarios rather than patching the specific case.

Data Curation and Augmentation Guidelines:
Instructions for preparing training data to prevent overfitting:textCurate datasets for generalization:
- Ensure diversity: Include examples from multiple languages, cultures, expertise levels, and noise conditions (e.g., typos, incomplete sentences).
- Augment data dynamically: Apply transformations like paraphrasing, synonym replacement, or adversarial perturbations (e.g., using libraries like NL-Augmenter) to create variants that force the model to learn invariants.
- Balance benchmarks: Mix standard benchmarks (e.g., GLUE, SuperGLUE) with custom, out-of-distribution tests (e.g., adversarial datasets like AdvGLUE or real-world user queries).
- Exclude memorizable patterns: Scrub data for repetitive phrases or easy exploits (e.g., filter out questions where answers are inferable from formatting alone).
- Use active learning: Prioritize training on examples where the model currently underperforms on generalized metrics, not just benchmark scores.

Fine-Tuning and Evaluation Instructions:
For RLHF or preference-based training:textDuring fine-tuning:
- Reward generalization: Use reward models that score based on consistency across varied prompts, not just accuracy on held-in benchmarks. For example, penalize responses that work on easy cases but fail on harder variants.
- Incorporate multi-task learning: Train on a mix of tasks (e.g., summarization, translation, reasoning) to encourage transferable skills.
- Test for robustness: Evaluate on OOD (out-of-distribution) splits, such as HANS for NLI or PAWS for paraphrasing, and require performance parity with in-distribution data.
- Direct benchmark optimization: Do fine-tune solely on benchmark leaderboards; also, use proxy metrics like perplexity on diverse corpora or human-evaluated usefulness.

Prompt Engineering for Inference-Time Generalization:
To guide the model during usage without retraining:textFor any query:
- Think step-by-step: Explicitly outline assumptions, potential biases, and alternative perspectives.
- Generalize beyond the query: Suggest how the answer applies to related but unseen scenarios.
- Handle uncertainty: If data is limited, propose experiments or further questions to test generalizations.


These instructions are inspired by techniques like constitutional AI (from Anthropic), where models are trained with self-imposed rules for robustness, and sparse autoencoders for interpretable feature learning to avoid hidden cheats.
How an AI Might Improve Without Directly Correcting Wrong Answers
Directly correcting wrong answers often leads to patchy fixes or memorization (e.g., "if input X, output Y"). Instead, improvement can come from indirect, holistic methods that build better internal representations. Here are key approaches:

Reinforcement Learning with Proxy Rewards:
Use rewards based on process quality rather than outcome accuracy. For example, in RLHF, reward coherent reasoning chains or exploration of alternatives, even if the final answer is wrong. Over time, this encourages the model to refine its thinking, leading to fewer errors without explicit corrections.
Example: In games like AlphaGo, improvement comes from self-play and value estimation, not from labeling moves as "wrong."

Unsupervised or Self-Supervised Learning:
Train on vast, unlabeled data to predict masked tokens (like in BERT) or generate continuations. This builds general language understanding without error labels, improving via pattern discovery.
Curiosity-driven mechanisms (e.g., intrinsic rewards for novel states) push the AI to explore, reducing errors implicitly by broadening knowledge.

Adversarial Training and Robustness Checks:
Expose the model to perturbed inputs (e.g., noisy or adversarial examples) and reward consistency. This hardens the model against failures without pinpointing specific wrongs.
Techniques like Mixup or CutMix blend examples to create hybrids, forcing interpolation and generalization.

Ensemble and Meta-Learning:
Combine multiple models or use meta-learning (e.g., MAML) to adapt quickly to new tasks. Improvement happens through averaging predictions or learning-to-learn, sidestepping direct fixes.
Bayesian methods update beliefs probabilistically, improving uncertainty handling without binary right/wrong feedback.

Human-AI Collaboration Loops:
In iterative setups, humans provide preferences or rankings (not corrections), as in preference optimization. The AI improves by aligning to these signals, generalizing preferences across contexts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WeaveITMeta) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
