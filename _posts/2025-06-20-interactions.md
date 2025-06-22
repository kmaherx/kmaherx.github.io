---
layout: distill
title: "Spatial Omics III: Defining Intercellular Interactions"
description: A principled approach to representing cell-cell interactions in spatial omics data
tags:
giscus_comments: false
date: 2025-06-20
featured: false
categories: spatial-omics
# thumbnail: assets/figures/interactions/interaction_gradient.png

authors:
  - name: Kamal Maher
    affiliations:
      name: MIT, Broad Institute

toc:
  - name: Introduction
  - name: Derivation
    subsections:
      -name: Derivation 1 (nodes)
      -name: Derivation 2 (edges)
  - name: Simulation
    subsections:
      - name: Simulated interaction components
  - name: Human lymph node
    subsections:
      - name: Lymph node interaction components
  - name: Mouse brain
    subsections:
      - name: Brain interaction components
  - name: Conclusion

images:
  compare: true
  slider: true
---

<style>
  .slider-with-shadows {
    --default-handle-shadow: 0px 0px 5px rgba(0, 0, 0, 1);
    --divider-shadow: 0px 0px 5px rgba(0, 0, 0, 0.5);
  }
</style>


## Introduction

## Derivation

### Derivation 1 (nodes)

### Derivation 2 (edges)

## Simulation

### Simulated interaction components

Recall that **high-pass filtering isolates small-scale patterns.**
Let's visualize the result of high-pass filtering on a given gene expression signal over the tissue.
Consider a marker gene for the outer region of [the simulated tissue that we constructed previously](/blog/2025/graph-fourier#simulation).
Drag the slider left to right to compare the signal before and after filtering.

<div style="width: 50%; max-width: 768px; margin: 0 auto;">
  <img-comparison-slider class="slider-with-shadows">
    {% include figure.liquid path="assets/figures/fourier/tissue_before_filtering.png" class="img-fluid rounded z-depth-1" slot="first" %}
    {% include figure.liquid path="assets/figures/fourier/tissue_after_highpass.png" class="img-fluid rounded z-depth-1" slot="second" %}
  </img-comparison-slider>
</div>
<figcaption><strong>Figure 1:</strong> Comparison of a gene expression signal before (left image) and after (right image) high-pass filtering. </figcaption>
<br>

Note that the large-scale variation in expression has been washed away, keeping only the small-scale variation in expression in the outer ring.
With these low-pass filtered signals, we can begin **comparing them to find interesting gene-gene relationships** that ultimately describe regions.

<figure style="text-align: center;">
  <img src="/assets/figures/interactions/interaction_covariance.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 2:</strong> Visualization of the high-pass covariance matrix.</figcaption>
</figure>

<figure style="text-align: center;">
  <img src="/assets/figures/interactions/interaction_components.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 3:</strong> Visualization of high-pass gene programs, i.e. "interaction components". The right-hand plot is an enlarged version of the inset in the left-hand plot.</figcaption>
</figure>


pattern across so few cells, shows up as later PC
can also rationalize based on eq. (), "avging out".

## Human lymph node

### Lymph node interaction components

## Mouse brain

### Mouse brain interaction components

## Conclusion