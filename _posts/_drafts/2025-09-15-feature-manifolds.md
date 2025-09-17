---
layout: distill
title: "Feature Manifolds"
description: Interpretable connected components of d-orthogonal feature graphs
tags:
giscus_comments: false
date: 2025-09-15
featured: false
categories: mech-interp
thumbnail:

authors:
  - name: Kamal Maher
    affiliations:
      name: Independent

toc:
  - name: Introduction
  - name: Math
  - name: Conclusion
---

## Introduction

---

## Math

Omitting biases, an SAE is the transformation

$$ W_{dec} \sigma(W_{enc} x), $$

with weights $W_{enc} \in \mathbb{R}^{d_{feat} \times d_{model}}, W_{dec} \in \mathbb{R}^{d_{model} \times d_{feat}}$ and residual stream state $x \in \mathbb{R}^d_{model}$.

We row normalize the SAE decoder weight matrix $W_{dec}$ and calculate the cosine similarity graph

$$ A = W_{dec}^{\top} W_{dec} \in \mathbb{R}^{d_{feat} \times d_{feat}}. $$

We then threshold for edges with similarities above $\delta = 0.4$, yielding a $\delta$-orthogonal graph $A$.
The intuition is that $W_{dec}$ maps from feature space back into the residual stream and thus represents how each feature contributes to the residual stream moving forward.
Additionally, we don't have to worry about the nonlinearity, which has already been applied by this point.

Something interesting we observed was that top activators were often the same as top logits.
In other words, for a top activating token $t \in \mathbb{R}^d_{vocab}$ and output weights matrix $W_O \in \mathbb{R}^{d_{vocab} \times d_{model}}$, large activations $$\sigma(W_{enc} x_{t})_{a} \in \mathbb{R}$$ for feature $a$ implied large output logits $$(W_O)_t (W_{dec})_a \in \mathbb{R}$$.

Generalized to all tokens, we'd just have indexing for a feature $a$ and large covariance, i.e.

$$ \sigma(W_{enc} T) \in \mathbb{R}^{d_{feat}} $$

$$ W_O W_{dec} \in \mathbb{R}^{d_{feat}} $$

$$ || \sigma(W_{enc} T)^{\top} W_O W_{dec} ||,$$

where $$T \in \mathbb{R}^{d_{model} \times d_{top}}$$ is the vocab matrix of embedded top activator tokens, and $d_{top}$ is the number of top tokens to consider.


no

$$ \sigma(W_{enc} x_t) \in \mathbb{R}^{d_{feat}} $$

$$ W_O^t W_{dec} \in \mathbb{R}^{d_{feat}} $$

$$ || \sigma(W_{enc} t)^{\top} W_O W_{dec} ||,$$

where $$x_t \in \mathbb{R}^{d_{model}}$$ is the token embedding and $t$ is the index within the model's vocabulary.

---

## Conclusion
