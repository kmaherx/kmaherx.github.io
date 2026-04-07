---
layout: page
title: Interpretable soft prompt tuning <br> via self-verbalization
description: <em>April 2026</em>
img: assets/figures/ispt/spanish_response.png
importance: 1
category: work
tabs: true
---

## Summary

Soft prompt tuning is powerful but uninterpretable. We show that **contextualization** — embedding the soft prompt in a syntactic frame — is necessary and sufficient to make soft prompts interpretable via self-verbalization and SAE feature alignment.

## Background

{% include figure.liquid loading="eager" path="assets/figures/ispt/overview.png" class="img-fluid rounded z-depth-1" caption="Instruction-tuned LLM schematic. Solid arrows indicate direct progression; dotted arrows skip intermediate layers (attention and MLP blocks), represented by the grey wiring between embeddings and activations. Chat formatting omitted for simplicity." %}

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
