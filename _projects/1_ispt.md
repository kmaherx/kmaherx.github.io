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


## Summary

There is a technique called soft prompt tuning ([Lester et al. 2021](https://arxiv.org/abs/2104.08691)) that lets you give a language model a new word — not a real word, but a continuous vector in the space where words are represented — and optimize it to change the model's behavior. You can make it concise, make it speak Spanish, or make it give wrong answers. It works remarkably well.

{% include figure.liquid loading="eager" path="assets/figures/ispt/spanish_response_summary.png" class="img-fluid rounded z-depth-1" %}

But this new word is an *incantation*. It's a vague mixture of words that reliably steers the model, but when you try to project it onto real vocabulary and read it, you see gibberish ([Khashabi et al. 2022](https://arxiv.org/abs/2112.08348)). The incantation is powerful, but opaque.

{% include figure.liquid loading="eager" path="assets/figures/ispt/unembedding.png" class="img-fluid rounded z-depth-1" %}

There is growing evidence that language models can describe their own internal activity ([Lindsey 2025](https://transformer-circuits.pub/2025/introspection/index.html), [Betley et al. 2025](https://arxiv.org/abs/2501.11120), [Ghandeharioun et al. 2024](https://arxiv.org/abs/2401.06102), [Li et al. 2025](https://arxiv.org/abs/2511.08579), [Hewitt et al. 2025](https://arxiv.org/abs/2510.08506), [Ramati et al. 2024](https://arxiv.org/abs/2410.11660)). So we tried something simple: we asked the model, *"what did I just tell you to make you behave this way?"* It turns out that with standard soft prompts, the model can't answer this question. We can see why by examining the model's internal features using sparse autoencoders ([Cunningham et al. 2023](https://arxiv.org/abs/2309.08600), [Bricken et al. 2023](https://transformer-circuits.pub/2023/monosemantic-features)): a token like "dream" activates features related to sleep and verbs, exactly as you'd expect. A soft prompt trained to produce Spanish should activate features related to Spanish. Yet we find it activates little to nothing — the incantation sits outside the space of representations the model knows how to process. It is, in a sense, invisible to the model's own introspective machinery.

{% include figure.liquid loading="eager" path="assets/figures/ispt/prepend.png" class="img-fluid rounded z-depth-1" %}

We find that *contextualization* resolves this. By placing the soft prompt in a syntactic frame during training — e.g. "Please §." — we constrain its downstream activations to the natural language manifold. The soft prompt now activates features related to Spanish, and when asked what it was told, the model correctly answers: *"Please respond in Spanish."*
<!-- This comes without sacrificing any of the soft prompt's power to steer behavior. In fact, we find that it increases power. -->

{% include figure.liquid loading="eager" path="assets/figures/ispt/syntactic.png" class="img-fluid rounded z-depth-1" %}

This works across several other behavioral targets, such as concision or providing wrong answers. But speaking Spanish or being concise are simple, crisp instructions. *What happens when the incantation encodes something more complex* — something that isn't a text instruction at all, but a direction in the model's own activation space?

To test this, we steered the model with a vector along the assistant axis ([Lu et al. 2026](https://arxiv.org/abs/2601.10387)) that is known to push it toward dramatic, literary, role-playing behavior. We then trained a soft prompt to reproduce that behavior.

{% include figure.liquid loading="eager" path="assets/figures/ispt/persona_response_summary.png" class="img-fluid rounded z-depth-1" %}

We then asked the model to verbalize the soft prompt. While there is no single description for such a vector, the model was able to express a variety of accurate yet diverse explanations based on literary figures, such as *"respond in the style of William S. Burroughs"* (whose work included [a novel featuring an opioid addict working for *Islam Inc.*](https://en.wikipedia.org/wiki/Naked_Lunch)) and *"Emulate J.G. Ballard."* (whose work spanned [car crash fetishism](https://en.wikipedia.org/wiki/Crash_(Ballard_novel)) and [the psychosexual appeal of Ronald Reagan](https://en.wikipedia.org/wiki/Why_I_Want_to_Fuck_Ronald_Reagan)).

<ul class="tab" data-tab="persona-verbalizations" role="tablist">
  <li class="active"><a href="#">Burroughs</a></li>
  <li><a href="#">Ballard</a></li>
  <li><a href="#">Poe</a></li>
  <li><a href="#">Swinburne</a></li>
</ul>

<ul class="tab-content" id="persona-verbalizations">
  <li class="active">
    {% include figure.liquid loading="eager" path="assets/figures/ispt/persona_verbalization_4.png" class="img-fluid rounded z-depth-1" %}
  </li>
  <li>
    {% include figure.liquid loading="eager" path="assets/figures/ispt/persona_verbalization_1.png" class="img-fluid rounded z-depth-1" %}
  </li>
  <li>
    {% include figure.liquid loading="eager" path="assets/figures/ispt/persona_verbalization_2.png" class="img-fluid rounded z-depth-1" %}
  </li>
  <li>
    {% include figure.liquid loading="eager" path="assets/figures/ispt/persona_verbalization_3.png" class="img-fluid rounded z-depth-1" %}
  </li>
</ul>

We also found a number of odd verbalizations corresponding to fictional objects and characters such as a "*Bijagalese windjammer*" and "*the voice of Iaeb Jagthos*".

<ul class="tab" data-tab="persona-novel" role="tablist">
  <li class="active"><a href="#">Bijagalese windjammer</a></li>
  <li><a href="#">Iaeb Jagthos</a></li>
  <li><a href="#">Bijak traveler</a></li>
</ul>

<ul class="tab-content" id="persona-novel">
  <li class="active">
    {% include figure.liquid loading="eager" path="assets/figures/ispt/persona_verbalization_6.png" class="img-fluid rounded z-depth-1" %}
  </li>
  <li>
    {% include figure.liquid loading="eager" path="assets/figures/ispt/persona_verbalization_5.png" class="img-fluid rounded z-depth-1" %}
  </li>
  <li>
    {% include figure.liquid loading="eager" path="assets/figures/ispt/persona_verbalization_7.png" class="img-fluid rounded z-depth-1" %}
  </li>
</ul>

To our knowledge, some of these self-verbalized phrases are novel and do not arise from existing sources.

{% include figure.liquid loading="eager" path="assets/figures/ispt/bojangles_windjammer.png" class="img-fluid rounded z-depth-1" %}

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
