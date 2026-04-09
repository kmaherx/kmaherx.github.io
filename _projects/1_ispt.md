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

But there is a problem. These learned embeddings are uninterpretable. They live in embedding space but don't correspond to any real token. And if you try to project them back onto vocabulary — finding the nearest real word for each embedding — you get gibberish ([Khashabi et al. 2022](https://arxiv.org/abs/2112.08348)). For instance, a soft prompt trained for concision has top tokens like "London", "ব্যক্তিদের", "慑", and "되었", all with similar cosine similarities (~0.09), indicating no true outliers that define the soft prompt's meaning. They are, in effect, *incantations*: they reliably steer the model, but no one can read them.

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
