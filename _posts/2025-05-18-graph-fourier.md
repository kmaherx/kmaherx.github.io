---
layout: distill
title: >
  Spatial Omics I: Transcriptional Signals Over Tissue Domains
description: An introduction to graph signal processing in biological tissues
tags:
giscus_comments: false
date: 2025-05-14
featured: true
# tags: spatial-omics
categories: spatial-omics

authors:
  - name: Kamal Maher
    affiliations:
      name: MIT, Broad Institute

bibliography: 2025-05-18-graph-fourier.bib

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: Introduction
    subsections:
      - name: Tissues
      - name: Domains and signals
      - name: Signal processing
  - name: Simulation
  - name: The tissue domain
  - name: Transcriptional signals
  - name: Frequencies
  - name: Spectra
  - name: Filtering
  - name: Conclusion

---

## Introduction

Here, we will build an intuition for how to describe frequencies over graphs in terms of gene expression over biological tissues.
We'll assume familiarity with linear algebra.
We will use simulations to


### Tissues

Cells are important

But so are the relationships between them

Multicellular regions

Intercellular interactions


### Domains and signals

Sound (Continuous, 1D)

Images (discrete, 2D)

Graphs? (discrete, but what dimension? will come back to this)


### Signal processing

Sound (music, bass/mids/treble)

Images (denoising by removing highs)


---

## Simulation

We'll rely on a simple simulation to demonstrate core concepts.

Take the brain as inspiration.
We'll start off by creating a simple tissue made up of $n=2000$ cells randomly scattered throughout a unit circle.

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/tissue_domain_nozoom.png"
       alt=""
       style="width:30%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 1:</strong> Simulation of a simple, circular tissue. </figcaption>
</figure>

Some spatial cells that form layers (neurons), some non-spatial cells scattered throughout (glia, for the most part).

Ground truth region patterns.
Add different noise patterns to make multiple gene markers for each ground truth pattern.
One marker per pattern shown in Figure <>.

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/simulation_lows.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 1:</strong> Schematic for simulating multicellular region patterns. </figcaption>
</figure>

On the other hand, spatial patterns also arise from interactions between cells.
For instance, two *neighboring* cells might interact when *one* expresses a ligand and *the other* expresses the corresponding receptor.
This would produce a mutually exclusive pattern between two genes: one associated with the ligand and the other associated with the receptor.
We can simulate this by

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/simulation_highs.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 1:</strong> Schematic for simulating intercellular interaction patterns. </figcaption>
</figure>
---


## The tissue domain

There are many different examples of graph domains, including [molecules](https://arxiv.org/abs/1603.00856) and [social networks](https://arxiv.org/abs/1706.02216).
The domain that we will focus on here is that of biological tissues, in which one might imagine hopping from cell to cell 

We can think of such a "tissue domain" as an undirected graph over $n$ nodes, each of which represents a cell.
We could construct this graph in many ways, including connecting each cell to its k nearest physical neighbors.
Personally, I prefer using a Delaunay triangulation, as it creates a mesh that's embeddable in 2D, which respects my own visual intuition.
If you're optimistic, you might also believe that it [captures the mechanical forces present in biological tissues](https://pubmed.ncbi.nlm.nih.gov/20082148/).

To demonstrate this process, let's create a simulated tissue domain.
Our domain will simply consist of a bunch of cells -- $n=2000$ to be exact -- scattered uniformly within a unit circle.
After performing a Delaunay triangulation, we can zoom in to see that the cells are indeed connected to their spatial neighbors to form a 2D mesh.
These connections define the space within which we can hop from cell to cell.

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/tissue_domain.png"
       alt=""
       style="width:50%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 1:</strong> Cells arranged in a spatial graph to form a tissue domain. </figcaption>
</figure>

Now let's define the key mathematical objects associated with this tissue domain.
Our graph can be represented by the symmetric adjacency matrix

$$
\mathbf{A} \in \{0,1\}^{n \times n}.
$$

Each entry $$\mathbf{A}_{ij}$$ is either $1$, which represents two cells that are spatially adjacent, or $0$, which represents two cells that are not adjacent.
While we could weight these edges based on physical distances between cells, we will instead stick to simple binary edges for simplicity.
We also will not consider self loops: $$\mathbf{A}_{ii} = 0$$.
The number of neighbors for cell $i$ -- the "degree" -- is given by $$d_i = \sum_j \mathbf{A}_{ij}$$.
These values are often consolidated into the diagonal degree matrix

$$
% \mathbf{D} = \operatorname{diag}(d_1, ..., d_n) =
\mathbf{D} =
\begin{bmatrix}
  d_{1} & & \\
  & \ddots & \\
  & & d_{n}
\end{bmatrix}
\in \mathbb{R}^{n \times n}.
$$

Now that we have a tissue domain, we can begin to 

---


## Transcriptional signals

A signal is a value associated with each node in our graph.
For instance, we could consider the amount that a given gene is expressed in each cell within a tissue.
Spatial transcriptomics provides exactly this type of information, typically for hundreds of different genes.

Before discussing transcriptional signals in mathematical detail, let's first gain some intuition from biology for what they look like and then add some to our simulation.
In real tissues, they might vary in their spatial scale, some looking like a region and some looking noisy.
Allen atlas

---


## Frequencies

Nice, looks great.

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/frequencies.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 3:</strong> Examples of different frequencies over the tissue domain.  </figcaption>
</figure>

{% details Why do highs appear constrained to one part of the tissue? %}
When taking a look at 
$\mathbf{v}_{300}
$
, the fluctuations appear constrained to the lower left portion of the tissue.
Uncertainty principle
How I see it: consequence of "irregular" topology
If all cells had the same degree:
But because they don't, we have:
{% enddetails %}

---


## Spectra


---

## Filtering


---

## Conclusion

What are highs?

What are mids?

How can we capture regions and interactions?

Links to later posts.