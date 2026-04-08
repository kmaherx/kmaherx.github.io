---
layout: page
title: Interpretable soft prompt tuning via self-verbalization
description: <em>April 2026</em>
img: assets/figures/ispt/spanish_response.png
importance: 1
category: work
tabs: true
related_publications: true
_styles: >
  .post-title {
    max-width: 600px;
  }

---


## Summary

There is a technique called soft prompt tuning {% cite lester2021power %} that lets you give a language model a new word — not a real word, but a continuous vector in the space where words are represented — and optimize it to change the model's behavior. You can make it concise, make it speak Spanish, or make it give wrong answers. It works remarkably well.

{% include figure.liquid loading="eager" path="assets/figures/ispt/spanish_response_summary.png" class="img-fluid rounded z-depth-1" %}

But this new word is an *incantation*. It's a vague mixture of words that reliably steers the model, but when you try to project it onto real vocabulary and read it, you see gibberish {% cite khashabi2022prompt %}. The incantation is powerful, but opaque.

{% include figure.liquid loading="eager" path="assets/figures/ispt/unembedding.png" class="img-fluid rounded z-depth-1" %}

There is growing evidence that language models can describe their own internal activity {% cite lindsey2025introspection betley2025tell ghandeharioun2024patchscopes li2025training hewitt2025neologisms ramati2024eliciting %}. So we tried something simple: we asked the model, *"what did I just tell you to make you behave this way?"* It turns out that with standard soft prompts, the model can't answer this question. The incantation doesn't trigger any of the model's learned features — it produces patterns of activity the model has never encountered during training. We can measure this: sparse autoencoders {% cite lieberum2024gemma %} trained on natural text fail to reconstruct these activations, suggesting that the soft prompt sits outside the space of representations the model knows how to process. It is, in a sense, invisible to the model's own introspective machinery.
<!-- , exerting subconscious control. -->

{% include figure.liquid loading="eager" path="assets/figures/ispt/off_manifold.png" class="img-fluid rounded z-depth-1" %}

Moving the soft prompt from the beginning of the input to the end — placing it after the user's message, where instructions naturally live — brings it back on-manifold. The model's learned features light up, and sparse autoencoders can reconstruct the activations. But self-verbalization still fails. The model processes the soft prompt through its normal pathways, yet it can't describe what it's doing. Being on-manifold may be necessary, but it is not sufficient.

{% include figure.liquid loading="eager" path="assets/figures/ispt/on_manifold.png" class="img-fluid rounded z-depth-1" %}

What's missing is syntactic identity. By placing the incantation in a frame during training — e.g. "Please §." — we give the model a structure it can recognize as an instruction. Now, when asked what it was told, it answers: *"Please respond in Spanish."* This comes without sacrificing any of the incantation's power to steer behavior. In fact, if anything, we find that it increases power.

{% include figure.liquid loading="eager" path="assets/figures/ispt/verbalizable.png" class="img-fluid rounded z-depth-1" %}

This works across several targets. For Spanish, the model verbalizes "Respond in Spanish." But speaking Spanish is a simple, crisp instruction. *What happens when the incantation encodes something more complex* — something that isn't a text instruction at all, but a direction in the model's own activation space?

To test this, we steered the model with a vector along the assistant axis {% cite lu2026assistant %} that is known to push it toward dramatic, literary, role-playing behavior. We then trained a soft prompt to reproduce that behavior, and asked the model to verbalize it. While there is no explicit name for a vector, the model was able to express fragments, like *"Mimic the style and structure of T."* and *"example"*, which together
We also found a number of accurate yet odd verbalizations, like *""*, *""*, and *"Be a Tharg"*, some of which ...
The model can be genuinely creative when tasked with expressing difficult-to-express concepts.

<!-- [placeholder: tabbed figure showing tier 2 steered outputs — the dramatic, poetic responses] -->

This work sits at a middle ground — between quantitative measurement and the experience of peering into a model, watching it grapple with its own complexity, and finding that it can, partially, explain itself.

---


## Background

### LLM architecture visualization

{% include figure.liquid loading="eager" path="assets/figures/ispt/overview.png" class="img-fluid rounded z-depth-1" caption="Instruction-tuned LLM schematic. Solid arrows indicate direct progression; dotted arrows skip intermediate layers (attention and MLP blocks), represented by the grey wiring between embeddings and activations. Chat formatting omitted for simplicity." %}


{% include figure.liquid loading="eager" path="assets/figures/ispt/soft_prompt.png" class="img-fluid rounded z-depth-1" caption="Soft prompt tuning. A learned continuous embedding (red) is prepended to the input and optimized to steer model behavior — here, to respond in Spanish. The model weights remain frozen; only the soft prompt is trained." %}

### Soft prompt tuning


### Sparse autoencoders


### Self-verbalization

---


## Tier 1

<ul class="tab" data-tab="response-figures" role="tablist">
  <li class="active"><a href="#">Spanish</a></li>
  <li><a href="#">Concise</a></li>
  <li><a href="#">Wrong</a></li>
  <li><a href="#">No Vowels</a></li>
</ul>

<ul class="tab-content" id="response-figures">
  <li class="active">
    {% include figure.liquid loading="eager" path="assets/figures/ispt/spanish_response.png" class="img-fluid rounded z-depth-1" %}
  </li>
  <li>
    {% include figure.liquid loading="eager" path="assets/figures/ispt/concise_response.png" class="img-fluid rounded z-depth-1" %}
  </li>
  <li>
    {% include figure.liquid loading="eager" path="assets/figures/ispt/wrong_response.png" class="img-fluid rounded z-depth-1" %}
  </li>
  <li>
    {% include figure.liquid loading="eager" path="assets/figures/ispt/novowels_response.png" class="img-fluid rounded z-depth-1" %}
  </li>
</ul>
