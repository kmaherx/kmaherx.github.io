---
layout: distill
title: >
  Spatial Omics I: Transcriptional Signals Over Tissue Domains
description: An introduction to graph signal processing in spatial omics data
tags:
giscus_comments: false
date: 2025-05-14
featured: true
categories: spatial-omics
thumbnail: assets/img/freq10.png

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
      - name: Overview
  - name: Simulation
    subsections:
      - name: Construction
      - name: The tissue domain
      - name: Transcriptional signals
      - name: Frequencies
      - name: Spectra
      - name: Filtering
  - name: Real data
    subsections:
      - name: Frequencies
      - name: Filtering
      - name: Multiple samples
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

Here, we will build an intuition for how to describe frequencies over graphs in terms of gene expression over biological tissues.
We'll assume familiarity with linear algebra.
We will use simulations to


### Tissues

Cells are important
Have different [roles in the tissue that can largely be described by their molecular patterns](https://www.cell.com/cell-systems/fulltext/S2405-4712(18)30482-4).
Such "types" catalogued by efforts such as the human cell atlas.

But cells don't act independently.
If you look at an organism, you'll notice a few things.
First, it's an organism; there's some sense that it's a discrete collection of cells all working together.
Second, you might notice that cells within a given organism are further organized into discrete multicellular structures, such as organs or the anatomical regions within them.
Third, and perhaps most importantly, the very formation and function of such structures is driven by interactions between their constituent cells.
Altogether, it is critical to look beyond cells in isolation to recognize their 

We can break these patterns down into two broad categories.
Multicellular regions
Intercellular interactions

Both of these patterns are inherently spatial.
Regions
Interactions
Thus, spatial omics technologies enable us to measure these patterns.

Given the importance of this data, we should have a concrete way to quantitatively describe and analyze them.
However, the current landscape of computational methods for spatial omics data is deeply fragmented.
It largely consists of complicated methods based on graph neural networks and probabilistic models, and it lacks a fundamental basis for relating these approaches or even defining the core features one would like to quantify.

In this series of posts, we will derive such a basis.
The tools we will use are borrowed from the vast field of signal processing.


### Domains and signals

Sound (Continuous, 1D)

Images (discrete, 2D)

Graphs? (discrete, but what dimension? will come back to this)


### Signal processing

Sound (music, bass/mids/treble)

Images (denoising by removing highs)


### Overview

Simulations then real data

This isn't a new idea.
Fundamentals of GSP.
GNNs based on GSP, just overshadow the basics.
But when we thoroughly explore the basics, we can find previously overlooked ideas.
This chapter will only cover the basics, with applications covered in future posts.


---


## Simulation

We'll first demonstrate the fundamental concepts in a simple simulation.
This will allow us to build up some intuition before approaching data gathered from real tissues, which is full of both technical artifacts and true biological complexity.


### Construction

Take the brain as inspiration.
We'll start off by creating a simple tissue made up of $n=2000$ cells randomly scattered throughout a unit circle.
One should ask why a circle and not a sphere.
It's because, while tissues are 3D objects, we tend to measure them in 2D slices.
That being said, everything presented in this series of posts can be generalized to data in arbitrary dimensions, including 3D.

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

Note that this simulation is not nearly as complex as a real biological tissue or the data measured from one.
That being said, it should at least provide a simple playground in which we can explore some fundamental quantitative concepts that we will later apply to real data.


### The tissue domain

Just as in music or images, tissues can be thought of in terms of signal processing.
Namely, that can be broken down into gene expression signals over a "tissue domain".
However, this tissue domain isn't continuous like time, as it's composed of discrete cells.
We might instead compare it to an image in which each pixel is akin to a cell.
However, cells are not arranged neatly into perfect grids like pixels are.
Thus, we need a different structure to represent our tissue.

We can think of such a tissue domain as an undirected graph over $n$ nodes, each of which represents a cell.
We could construct this graph in many ways, including connecting each cell to its k nearest physical neighbors.
Personally, I prefer using a Delaunay triangulation, as it creates a mesh that's embeddable in 2D, which respects my own visual intuition.
If you're optimistic, you might also believe that it [captures the mechanical forces present in biological tissues](https://pubmed.ncbi.nlm.nih.gov/20082148/).
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


### Transcriptional signals

A signal is a value associated with each node in our graph.
For instance, we could consider the amount that a given gene is expressed in each cell within a tissue.
Spatial transcriptomics provides exactly this type of information, typically for hundreds of different genes.

Before discussing transcriptional signals in mathematical detail, let's first gain some intuition from biology for what they look like and then add some to our simulation.
In real tissues, they might vary in their spatial scale, some looking like a region and some looking noisy.
Allen atlas


### Frequencies

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


### Spectra

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/spectra.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 3:</strong> An example gene expression signal in tissue space and in frequency space.  </figcaption>
</figure>


### Filtering

We perform filtering in frequency space by applying a kernel.

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/lowpass_spectra.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 3:</strong> A gene expression signal low-pass filtered in frequency space.  </figcaption>
</figure>

We then put the resulting spectrum back into the tissue to visualize the result.
Use the slider to visualize before (left) and after (right) filtering.

<div style="width: 50%; max-width: 768px; margin: 0 auto;">
  <img-comparison-slider class="slider-with-shadows">
    {% include figure.liquid path="assets/figures/fourier/tissue_before_filtering.png" class="img-fluid rounded z-depth-1" slot="first" %}
    {% include figure.liquid path="assets/figures/fourier/tissue_after_lowpass.png" class="img-fluid rounded z-depth-1" slot="second" %}
  </img-comparison-slider>
</div>
<figcaption><strong>Figure 1:</strong> Comparison of a gene expression signal before (left) and after (right) low-pass filtering. </figcaption>
<br>

Looks smoother.
Image literature -> denoising?

Alternatively, we could emphasize the highs by applying a square-root filter.

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/highpass_spectra.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 3:</strong> A gene expression signal high-pass filtered in frequency space.  </figcaption>
</figure>

Rather than grouping neighboring cells together, this filter appears to emphasize their differences.

<div style="width: 50%; max-width: 768px; margin: 0 auto;">
  <img-comparison-slider class="slider-with-shadows">
    {% include figure.liquid path="assets/figures/fourier/tissue_before_filtering.png" class="img-fluid rounded z-depth-1" slot="first" %}
    {% include figure.liquid path="assets/figures/fourier/tissue_after_highpass.png" class="img-fluid rounded z-depth-1" slot="second" %}
  </img-comparison-slider>
</div>
<figcaption><strong>Figure 1:</strong> Comparison of a gene expression signal before (left) and after (right) high-pass filtering. </figcaption>
<br>

Note that these particular kernels were chosen because they each have biological significance that will be explained in future blog posts.

---


## Real data

Next, we'll use [real data from the mouse brain](https://www.nature.com/articles/s41586-021-03705-x).

### Frequencies

First, the domain

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/tissue_domain_mop.png"
       alt=""
       style="width:70%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 3:</strong> A tissue domain calculated from real mouse brain data.  </figcaption>
</figure>

Then the frequencies

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/frequencies_mop.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 3:</strong> Example frequencies over the mouse brain tissue domain.  </figcaption>
</figure>

Then the spectrum of *Cux2*, a neocortical layer marker gene.

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/spectra_mop.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 3:</strong> An example gene expression signal in tissue space and in frequency space.  </figcaption>
</figure>


### Filtering

First, low-pass

<div style="width: 50%; max-width: 768px; margin: 0 auto;">
  <img-comparison-slider class="slider-with-shadows">
    {% include figure.liquid path="assets/figures/fourier/tissue_before_filtering_mop.png" class="img-fluid rounded z-depth-1" slot="first" %}
    {% include figure.liquid path="assets/figures/fourier/tissue_after_lowpass_mop.png" class="img-fluid rounded z-depth-1" slot="second" %}
  </img-comparison-slider>
</div>
<figcaption><strong>Figure 1:</strong> Comparison of a gene expression signal before (left) and after (right) low-pass filtering. </figcaption>
<br>

Then high-pass

<div style="width: 50%; max-width: 768px; margin: 0 auto;">
  <img-comparison-slider class="slider-with-shadows">
    {% include figure.liquid path="assets/figures/fourier/tissue_before_filtering_mop.png" class="img-fluid rounded z-depth-1" slot="first" %}
    {% include figure.liquid path="assets/figures/fourier/tissue_after_highpass_mop.png" class="img-fluid rounded z-depth-1" slot="second" %}
  </img-comparison-slider>
</div>
<figcaption><strong>Figure 1:</strong> Comparison of a gene expression signal before (left) and after (right) high-pass filtering. </figcaption>
<br>


### Multiple samples

Now on multiple slices.
The exact values differ based on the exact graph structure, but the meaning of the values is shared.
Lower values indicate lower frequencies, etc.
The distribution of frequency values for a given graph really just reflects the degree distribution.
Because cells are largely uniformly scattered throughout a given slice, these distributions are similar across slices.
Thus, we can compare frequencies across samples, and the meaning of a given kernel is conserved.
One could come up with ways to overcome the difference in dimensions to compare spectral patterns across datasets, but that's a topic we'll leave for another post.

In parallel
Approximation, PyGSP, Hammond

To keep things brief, we'll just look at the low-pass results.

<div style="width: 100%; max-width: 768px; margin: 0 auto;">
  <img-comparison-slider class="slider-with-shadows">
    {% include figure.liquid path="assets/figures/fourier/tissue_before_filtering_mop_all.png" class="img-fluid rounded z-depth-1" slot="first" %}
    {% include figure.liquid path="assets/figures/fourier/tissue_after_lowpass_mop_all.png" class="img-fluid rounded z-depth-1" slot="second" %}
  </img-comparison-slider>
</div>
<figcaption><strong>Figure 1:</strong> Comparison of a gene expression signal before (left) and after (right) low-pass filtering. </figcaption>
<br>


---

## Conclusion

How does this formalism enable us to capture regions and interactions?
The key is that it provides us with a flexible quantitative framework to represent these features.

Take lows for example.
It may be intuitive that they help us represent multicellular regions.
After all, lows are just large-scale patterns.
For instance, we should be able to low-pass filter gene expression patterns and then plug into the standard single-cell workflow to identify region clusters.
Indeed, this is the cornerstone of all region identification methods in the field, from those based on simple spatial smoothing to those based on complex graph neural networks.

But what are *highs*?
Are they just noise like in image processing?
And what of the frequencies in between, i.e. *mids*?
In the following posts, we'll explore these questions in detail, finding that lows indeed describe multicellular regions, highs describe intercellular interactions, and mids describe boundaries between regions.