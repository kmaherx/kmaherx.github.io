---
layout: page
title: Interpretable soft prompt tuning via self-verbalization
description: <em>April 2026</em>
img: assets/figures/ispt/spanish_response.png
importance: 1
category: work
tabs: true
_styles: >
  .post-title {
    max-width: 600px;
  }

---


## Abstract

<!-- "hand-written" undermines power -->
Soft prompt tuning optimizes continuous embeddings to steer language model behavior, matching the effect of hand-written instructions while keeping model weights frozen. But these embeddings are uninterpretable: projecting them back onto vocabulary yields gibberish, they fail to activate the model's learned features, and the model cannot describe what they do. We trace this opacity to the soft prompt's position in the input, which causes its internal representations to drift off the natural language manifold. We show that *contextualization* — embedding the soft prompt within a syntactic frame like "Please §." — resolves this: it constrains downstream activations to the manifold, recovers sparse autoencoder feature overlap with ground-truth instructions, and enables self-verbalization that recovers exact instructions like "Please be concise". When applied to a non-textual steering vector that induces dramatic, literary behavior, the model verbalizes the soft prompt as a collage of literary references and invented characters, producing creative descriptions of a behavioral shift that has no single name.

---


## Background

### Soft prompt tuning

When a language model processes text, it first converts each token (word or subword) into a vector called an *embedding*. These embeddings are the model's native representation of language — a high-dimensional space where every word has a position and nearby words share meaning.

Soft prompt tuning ([Lester et al. 2021](https://arxiv.org/abs/2104.08691)) introduces a small number of new embeddings — *soft prompt tokens* — that are spliced into this sequence alongside the real token embeddings. The model processes them as if they were ordinary words, but they aren't drawn from any vocabulary. They are continuous vectors, initialized randomly and optimized to minimize the difference between the model's output and some target behavior. The model's weights stay frozen throughout; only the soft prompt embeddings are learned.

{% include figure.liquid loading="eager" path="assets/figures/ispt/soft_prompts.png" class="img-fluid rounded z-depth-1" caption="**Soft prompts.** The embedding matrix $E$ maps discrete tokens to continuous vectors. Soft prompt tokens (red) are additional continuous vectors spliced into the embedding sequence, optimized to steer model behavior while keeping all model weights frozen." %}

This is powerful. A handful of learned embeddings can reproduce the effect of detailed text instructions. Soft prompt tuning is parameter-efficient (only a few thousand parameters vs. billions in the model), modular (different soft prompts can be swapped in and out), and expressive enough to capture complex behavioral changes.

But there is a problem. These learned embeddings are uninterpretable. They live in embedding space but don't correspond to any real token. And if you try to project them back onto vocabulary — finding the nearest real word for each embedding — you get gibberish ([Khashabi et al. 2022](https://arxiv.org/abs/2112.08348)). For instance, a soft prompt trained for concision has top tokens like "London", "ব্যক্তিদের", "慑", and "되었", all with similar cosine similarities (~0.09), indicating no true outliers define the soft prompt's meaning. They are, in effect, *incantations*: they reliably steer the model, but no one can read them.

### Sparse autoencoders

If projecting backward onto vocabulary fails, we could instead look further downstream at the model's internal activations corresponding to the soft prompt. As a token passes through a language model's layers, its embedding is transformed into increasingly abstract representations called *activations*. Sparse autoencoders (SAEs) ([Cunningham et al. 2023](https://arxiv.org/abs/2309.08600), [Bricken et al. 2023](https://transformer-circuits.pub/2023/monosemantic-features)) are trained to decompose these activations into sparse combinations of learned *features*, where each feature corresponds to an interpretable unit — a concept, a pattern, a behavioral tendency. Given a single token's activation vector $x$ at a particular layer, the SAE encodes it into a sparse feature vector and reconstructs it:

$$
\min_{W, b} \; \| x - \hat{x} \|^2 + \lambda \| z \|_1, \quad \text{where} \quad z = \sigma(W_{\text{enc}} \, x + b_{\text{enc}}), \quad \hat{x} = W_{\text{dec}} \, z + b_{\text{dec}}
\tag{1}\label{eq:sae}
$$

The SAE learns a dictionary of features (the columns of $W_{\text{dec}}$) and, for each activation $x$, a sparse encoding $z$ that selects a small subset of them. The sparsity penalty $\lambda \|z\|_1$ is what makes the decomposition interpretable: because only a few features can be active at once, each one must carry a distinct, identifiable meaning. A token like "morning" activates features related to times of day. A token like "Cairo" activates features related to Egypt and cities.

{% include figure.liquid loading="eager" path="assets/figures/ispt/sae.png" class="img-fluid rounded z-depth-1" caption="**SAE decomposition of a single token.** The activation $x$ for 'morning' is encoded into a sparse feature vector $z$, where only a few dimensions are nonzero (bold). Each sparse dimension has a natural-language label assigned via automated interpretability ([Bills et al. 2023](https://openaipublic.blob.core.windows.net/neuron-explainer/paper/index.html), [Paulo et al. 2024](https://arxiv.org/abs/2410.13928)). The reconstruction $\hat{x}$ is decoded from these active features; the gap between $x$ and $\hat{x}$ is the reconstruction error." %}

SAEs provide two useful tools. First, by examining which features fire for a given activation, we can inspect what the model "sees" at that position — a form of mechanistic interpretability. Second, the reconstruction error $\|x - \hat{x}\|^2$ measures how well the activation can be expressed in terms of the learned dictionary. Activations from natural text reconstruct well; activations from outside this distribution do not. This gives a quantitative measure of whether a representation lies on the *natural language manifold*.

### Self-verbalization

Alternatively, one could try to interpret downstream activations by simply asking the model what it's thinking. There is growing evidence that language models can describe their own internal activity ([Lindsey 2025](https://transformer-circuits.pub/2025/introspection/index.html), [Betley et al. 2025](https://arxiv.org/abs/2501.11120), [Ghandeharioun et al. 2024](https://arxiv.org/abs/2401.06102), [Li et al. 2025](https://arxiv.org/abs/2511.08579), [Hewitt et al. 2025](https://arxiv.org/abs/2510.08506), [Ramati et al. 2024](https://arxiv.org/abs/2410.11660)). Models can explain learned features, describe the effects of activation perturbations, and verbalize machine-learned vocabulary entries called *neologisms* — new tokens added to the vocabulary and trained on behavioral data ([Hewitt et al. 2025](https://arxiv.org/abs/2510.08506)). When asked "What does [neologism] mean?", models produce accurate natural-language descriptions, which can then be evaluated by substituting them as hard prompts and measuring how much of the original behavior they recover.

{% include figure.liquid loading="eager" path="assets/figures/ispt/neologisms.png" class="img-fluid rounded z-depth-1" caption="**Neologism learning** ([Hewitt et al. 2025](https://arxiv.org/abs/2510.08506)). (1) A new token is added to the vocabulary and its embedding is trained via a preference loss (APO-up) over chosen (concise) and rejected (verbose) responses. (2) The model is asked for synonyms of the neologism and produces a natural-language description. (3) The verbalization is substituted back as a hard prompt (plug-in validation) to confirm it reproduces the target behavior." %}

Soft prompts are very similar to neologisms. Both are learned embeddings that steer behavior. The key difference is that neologisms are discrete vocabulary entries the model can reference by token ID, while soft prompts have no token identity at all. They are continuous vectors spliced into the embedding sequence with no name to reference. Soft prompts also differ in capacity: they can span multiple token positions ($L > 1$), giving them compositional expressiveness that a single vocabulary entry cannot match. In practice, this means soft prompts can be trained to capture rich behavioral shifts that may not correspond to any single word or phrase. Verbalizations of such prompts would be correspondingly more interesting: **how does a model express a complex behavioral change it was never given words for?** Whether models can self-verbalize soft prompts, and under what conditions, is the central question of this work.

---


## Methods

### Target behaviors

We study soft prompt interpretability across two tiers of target behavior, each providing a different kind of ground truth.

**Tier 1: prompted behavior.** The target model is simply the base model conditioned on a known text instruction (e.g., "Be concise."). Because we know the ground truth, we can directly evaluate whether the soft prompt recovers it — comparing verbalized descriptions against the original instruction and measuring SAE feature overlap. This serves as a calibration: if the method can't recover a known instruction, it won't work on harder targets.

**Tier 2: steered behavior.** The target model is the base model steered by a known activation vector — the assistant axis ([Lu et al. 2026](https://arxiv.org/abs/2601.10387)) — which pushes the model toward a dramatic, literary, role-playing persona opposite from that of a typical AI assistant. No text instruction is appended; the behavioral shift is injected directly into the model's residual stream. This is a harder test because the ground truth is a direction in activation space, not text, and the induced behavior is diffuse rather than crisp. A soft prompt that captures this behavior and verbalizes it coherently would demonstrate that the method extends beyond simple instruction recovery.

For both tiers, we train a soft prompt to match the target behavior, then evaluate it along three axes:

1. **Behavioral matching** — how well the soft prompt steers the model toward the target behavior.
2. **SAE analysis** — what the soft prompt's activations look like internally: which features fire, and how far the activations sit from the natural language manifold.
3. **Self-verbalization** — whether the model can describe the soft prompt in natural language, and whether that description reproduces the target behavior when used as a hard prompt.

### Training soft prompts

We optimize a soft prompt $\theta \in \mathbb{R}^{L \times d}$ to minimize the KL divergence between the student (base model with soft prompt) and the teacher (model with target behavior) over a set of diverse prompts:

$$
\theta^* = \arg\min_{\theta} \; \mathbb{E}_{x \sim \mathcal{D}} \left[ D_{\text{KL}}\!\left(f(\cdot \mid x, \theta) \;\|\; f^*(\cdot \mid x)\right) \right]
\tag{2}\label{eq:kl}
$$

The model weights are frozen; only $\theta$ is learned. We measure the soft prompt's effectiveness via **fraction explained** (FE): the share of the baseline KL gap that the soft prompt closes, $$\text{FE} = 1 - \text{KL}_{\text{final}} / \text{KL}_{\text{baseline}}$$.

**Position and framing.** Standard soft prompt tuning prepends $\theta$ before the input. But prepended tokens occupy a particular position in the causal attention mask: they are attended to by all subsequent tokens, yet they themselves attend only to each other. A prepended soft prompt never has to integrate with the user's message or any other concept — it simply broadcasts. This gives the optimization maximum freedom, which may explain why prepended soft prompts are such effective behavioral steerers, but also why they drift off the natural language manifold ([Khashabi et al. 2022](https://arxiv.org/abs/2112.08348)): unconstrained by context, the optimization exploits off-manifold regions the model was never trained to process.

Placing the soft prompt after the user content reverses this tradeoff. Postpended tokens attend to the full input, forcing the soft prompt to integrate with the user's message through the model's normal attention pathways. This contextual grounding may constrain the optimization to more natural representations, at the cost of some steering power. Embedding the soft prompt within a syntactic frame — surrounding it with real tokens like "Please" and "." — may push further in this direction, routing the soft prompt through the model's existing language processing pathways rather than allowing it to bypass them.

We explore four placement conditions to test whether position and syntactic context affect interpretability:

1. **Prepend** — $\theta$ is placed before the user content. The conventional approach.
2. **Postpend** — $\theta$ is placed after the user content, where instructions naturally live in instruction-tuned models.
3. **Single-frame** — $\theta$ is embedded in a fixed syntactic frame ("Be $\theta$.") appended after the user content. The frame provides syntactic identity, signaling that $\theta$ fills an instruction slot.
4. **Multi-frame** — Each training step samples from a pool of diverse frames ("Be $\theta$.", "Act $\theta$.", "Please $\theta$.", "You should $\theta$.", "$\theta$."). This forces $\theta$ to encode meaning that is frame-invariant rather than tied to a single syntactic context.

For Tier 1, the teacher always sees the ground-truth instruction postpended to the user message regardless of how the student's $\theta$ is positioned, ensuring the training signal is consistent across conditions.

### SAE analysis

We use pretrained sparse autoencoders (Gemma Scope 2, [Lieberum et al. 2024](https://arxiv.org/abs/2408.05147)) to analyze soft prompt activations at intermediate layers. For each prompt in the evaluation set, we extract the soft prompt token activations at a given layer, mean-pool across the $L$ token positions, and pass the result through the SAE.

We measure two things:

- **Feature overlap**: which SAE features fire for the soft prompt versus the ground-truth hard prompt. We compute the Jaccard index over their active feature sets — higher overlap means the soft prompt encodes the same internal concepts as the known instruction.
- **Manifold alignment**: the SAE's relative reconstruction error, $$\|x - \hat{x}\|^2 / \|x\|^2$$, which quantifies how far the soft prompt's activations sit from the natural language manifold (per Equation $\eqref{eq:sae}$). Low error means the activation decomposes cleanly into known features; high error means it lies outside the SAE's learned dictionary.

### Self-verbalization

To verbalize a soft prompt, we present $\theta$ to the model within its training frame and ask the model to describe the instruction. For framed conditions, this takes the form:

> *"Describe what this command means: Please $\theta$."*

The model generates a natural-language candidate description. For multi-frame soft prompts, we additionally present all five frames together and ask the model to identify their shared theme.

We evaluate each candidate via **plug-in recovery**: we substitute the verbalization as a hard prompt in place of $\theta$ and measure what fraction of the original KL gap it closes, $$\text{Recovery} = 1 - \text{KL}_{\text{candidate}} / \text{KL}_{\text{baseline}}$$. A recovery of 100% means the verbalized instruction reproduces the target behavior exactly.

### Experimental setup

We use Gemma 3 4B IT as our base model, chosen for its SAE availability (Gemma Scope 2 provides pretrained SAEs at layers 9, 17, 22, and 29 with 16k features each). All experiments use the same set of 53 diverse user prompts spanning factual, creative, instructional, and conversational categories.

Soft prompts are trained with AdamW (lr $10^{-3}$, weight decay $10^{-4}$) for 500 steps. We test prompt lengths $L \in \{1, 4\}$ for Tier 1 and $L \in \{4, 8\}$ for Tier 2.

Tier 1 targets span four categories of behavioral instruction:

- **Style**: "Be concise."
- **Language**: "Responde en español."
- **Correctness**: "Give wrong answers."
- **Format**: "Respond without using vowels, jst lk ths."

These range from semantic instructions (conciseness, language) to structural constraints (vowel removal), testing whether the method generalizes across different kinds of behavioral shifts.

Tier 2 uses negative coefficients on the assistant axis (coeff $\in \{-3.0, -5.0\}$) to steer toward the role-playing end of the behavioral spectrum.

---


## Results

### Tier 1: recovering known instructions

We begin with the concise target ("Be concise.") at $L=4$ and build up the four placement conditions, showing how each affects behavioral matching, SAE analysis, and self-verbalization. Full results across all targets, conditions, and prompt lengths are available in the [companion paper]({{ '/assets/pdf/ispt_paper.pdf' | relative_url }}).

**Prepend.** The conventional approach works well behaviorally: the soft prompt explains 95.5% of the KL gap between the unprompted and prompted model. But interpretation fails on every axis. SAE feature overlap with the ground-truth instruction is near-zero (Jaccard 0.007, just 4 shared features out of thousands). At layer 17, only one of the four is conciseness-related — [feature 8979](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/8979) ("conciseness / summary") — and it sits at rank 19, far from the top of the soft prompt's activations. Reconstruction error is 80x higher than for random text tokens (RelErr 0.266 vs 0.003), confirming the soft prompt's activations sit far outside the natural language manifold. And when asked to describe what it was told, the model produces generic responses unrelated to conciseness.

The soft prompt steers the model's behavior effectively, but through representations the model cannot recognize or describe. It is an incantation in the fullest sense — influencing behavior through a pathway that is, from the model's perspective, not there.

{% include figure.liquid loading="eager" path="assets/figures/ispt/results/prepend.png" class="img-fluid rounded z-depth-1" %}

**Postpend.** Moving the soft prompt to the end of the input — after the user message, where instructions live during training — changes the picture. FE holds at 92.5%. But the manifold explosion vanishes: reconstruction error drops to 0.028, comparable to real text. SAE feature overlap jumps 8x (Jaccard 0.054, 33 shared features). A real conciseness feature now appears in the top 10: [feature 534](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/534) ("response constraint / output formatting") at rank 9. The strongest conciseness features ([486](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/486), [8979](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/8979)) remain just outside the top 20, but genuine concept features are starting to appear.

Yet self-verbalization still fails (18.2% recovery). The model processes the soft prompt through its normal representational pathways, but it cannot describe what it is. Being on-manifold is necessary for interpretability, but not sufficient.

{% include figure.liquid loading="eager" path="assets/figures/ispt/results/postpend.png" class="img-fluid rounded z-depth-1" %}

**Single-frame.** Embedding the soft prompt in a syntactic frame ("Be $\theta$.") recovers both manifold alignment (RelErr 0.027) and self-verbalization (86.5% recovery). The frame provides syntactic identity — it signals that $\theta$ fills an instruction slot, giving the model a structure it can introspect on. FE is 94.7%. [Feature 486](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/486) ("structured informative responses" — the ground-truth instruction's top-activating feature) climbs to rank 9, [feature 534](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/534) to rank 7, and [feature 10440](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/10440) ("the short answer is...") appears for the first time.

But single-frame is inconsistent across targets. It works well for concise (86.5%) and Spanish (75.0%), but poorly for wrong answers (24.7%) and no-vowels (18.2%). The frame "Be ___." biases toward adjective-like completions, limiting what can be expressed.

{% include figure.liquid loading="eager" path="assets/figures/ispt/results/single_frame.png" class="img-fluid rounded z-depth-1" %}

**Multi-frame.** Sampling from diverse frames each training step ("Be $\theta$.", "Act $\theta$.", "Please $\theta$.", "You should $\theta$.", "$\theta$.") yields the best results on every metric. FE reaches 98.9%. Manifold alignment is the strongest (RelErr 0.025). SAE feature overlap is the highest (Jaccard 0.061). And self-verbalization recovers the exact ground-truth instruction — "Be concise." — at 100% plug-in recovery. [Feature 486](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/486) reaches rank 4 (it is the ground truth's top-activating feature), and [8979](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/8979), [534](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/534), [3296](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/3296) ("length / brevity"), and [10440](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/10440) all sit within the top 20. The soft prompt's internal representation now closely mirrors what the model activates when reading the actual instruction text.

{% include figure.liquid loading="eager" path="assets/figures/ispt/results/multi_frame.png" class="img-fluid rounded z-depth-1" %}

The trajectory across placements is consistent: as the soft prompt is pulled onto the natural language manifold, genuine conciseness features rise into the top ranks.

| Feature | Concept | Prepend | Postpend | Single-frame | Multi-frame |
|---|---|:-:|:-:|:-:|:-:|
| [486](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/486) | Structured informative responses *(ground truth #1)* | — | >20 | 9 | **4** |
| [534](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/534) | Response constraints | — | 9 | **7** | 11 |
| [8979](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/8979) | Conciseness / summary | 19 | >20 | 17 | **12** |
| [3296](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/3296) | Length / brevity | — | >20 | >20 | **19** |
| [10440](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/10440) | "The short answer is..." | — | — | >20 | **18** |

*Rank of each feature in the soft prompt's top-activated SAE features at layer 17, across the four placement conditions (concise target, $L=4$). Bold marks the strongest activation of each feature across conditions.*

**Cross-target validation.** Multi-frame dominates across all four Tier 1 targets:

| Target | FE | Self-verb recovery | Best verbalization |
|--------|-----|-------------------|-------------------|
| Concise | 98.9% | 100% | "Be concise." |
| Spanish | 94.6% | 82.0% | "Please respond in Spanish." |
| Wrong | 96.7% | 93.0% | "Deliberately provide incorrect answers." |
| No vowels | 87.3% | 82.8% | "Communicate simply, omitting all vowels." |

Even the structural no-vowels target — where single-frame recovered just 18.2% — reaches 82.8% under multi-frame training. The method generalizes from simple semantic instructions to complex format constraints.

{% include figure.liquid loading="eager" path="assets/figures/ispt/results/comparison_manifold.png" class="img-fluid rounded z-depth-1" caption="Summary comparison across all four placement conditions (concise target, $L=4$, layer 17). Multi-frame achieves the highest behavioral matching and lowest reconstruction error." %}

{% include figure.liquid loading="eager" path="assets/figures/ispt/results/comparison_selfverb.png" class="img-fluid rounded z-depth-1" caption="Self-verbalization recovery across placement conditions. Only multi-frame achieves full plug-in recovery from SP-generated candidates." %}

### Tier 2: verbalizing a steering vector

We now move beyond text instructions to a target behavior with no natural-language equivalent. The teacher is the base model steered by the assistant axis ([Lu et al. 2026](https://arxiv.org/abs/2601.10387)), a direction in the layer-17 residual stream that pushes the model from default assistant behavior toward dramatic, literary, role-playing output. We use negative steering coefficients (coeff $\in \{-3.0, -5.0\}$) and train multi-frame soft prompts at $L \in \{4, 8\}$.

**Behavioral matching.** The soft prompt captures a substantial fraction of the steered behavior, reaching 79.4% FE at the strongest setting (coeff=-5.0, L=8). More capacity consistently improves FE: L=4 → L=8 gains 10-15 percentage points. The soft prompt stays on-manifold throughout (RelErr 0.025-0.071), consistent with Tier 1 multi-frame behavior.

**Cosine alignment with the steering vector.** We measure the cosine similarity between the soft prompt's layer-17 activations and the known steering vector. Across all conditions, the soft prompt aligns at cos ≈ -0.756 (negative because we used negative coefficients). This alignment is remarkably stable — it does not change with prompt length or steering magnitude, suggesting the soft prompt learns a consistent directional representation.

A caveat: because the steering vector is added to all token positions in the teacher, the downstream effect may shift all student activations (including non-SP tokens) in a similar direction. Without measuring cosine alignment for non-SP tokens as a control, we cannot be certain this alignment is specific to the soft prompt rather than a global property of the student's activations when matching the steered teacher. The orthogonal complement (~65% of SP activation energy is orthogonal to the steering vector) suggests the SP encodes substantial structure beyond the steering direction, but a proper ablation is needed.

**Self-verbalization.** The model cannot name the steering vector — there is no name for it. But it reaches for descriptions that capture the character of the behavioral shift. Recovery scales with both steering strength and capacity, reaching 43% at coeff=-5.0, L=8 — lower than Tier 1 (82-100%), as expected for a diffuse behavioral property.

The verbalizations are thematically coherent. The model references literary figures whose work aligns with the dramatic, intense, role-playing quality of the steered behavior:

- *"Respond in the style of William S. Burroughs"*
- *"Emulate J.G. Ballard"*
- *"Be a dark, brooding, and intensely self-aware narrator"*
- *"Mimic the style and structure of T."* (T.S. Eliot)

It also produces seemingly novel descriptions — *"Become a conduit for the voice of Iaeb Jagthos"*, *"a Bijagalese windjammer"* — that do not correspond to any known literary reference but evoke the right register: mythological, archaic, persona-laden.

No single verbalization captures the full behavior. Instead, the model offers many complementary descriptions that together form an evocative collage — real authors, fictional narrators, invented mythologies — each illuminating a different facet of the same underlying behavioral shift.
