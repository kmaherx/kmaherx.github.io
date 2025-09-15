---
layout: distill
title: "Spatial Omics III: Defining Intercellular Interactions"
description: A principled approach to representing cell-cell interactions in spatial omics data
tags:
giscus_comments: false
date: 2025-06-20
featured: false
categories: spatial-omics
thumbnail: assets/figures/interactions/interaction_pc.png

authors:
  - name: Kamal Maher
    affiliations:
      name: MIT, Broad Institute

toc:
  - name: Introduction
  - name: Derivation
  - name: Simulation
    subsections:
      - name: Simulated interaction components
      - name: The averaging out problem
      - name: Simulated sample-specific interactions
      - name: Simulated higher-order interactions
  - name: Human lymph node
    subsections:
      - name: Lymph node interaction components
  - name: Mouse brain
    subsections:
      - name: Brain interaction components
      - name: Brain higher-order interactions
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

In the previous posts, we focused on multicellular regions.
However, **the mechanistic relevance of regions is not clear.**
Instead, I think **we should be focusing on intercellular interactions**, which are the phenomena that shape the structure and function of tissues.

In this post, we will establish a conceptual definition of interactions and then translate it into a quantitative definition that enables us to identify interactions in simulations as well as real data.


## Derivation

Should include a diagram of a ligand/receptor ixn.

<figure style="text-align: center;">
  <img src="/assets/figures/interactions/juxtacrine_schematic.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 1:</strong> Schematic of a juxtacrine interaction.</figcaption>
</figure>


<figure style="text-align: center;">
  <img src="/assets/figures/interactions/interaction_schematics.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 1:</strong> Simplified schematics of different types of cell-cell interactions.</figcaption>
</figure>

First, notice that autocrine interactions would be indistinguishable from single-cell expression profiles (i.e. cell types) without additional information (e.g. temporal or *a priori* biological knowledge).
So we'll stick to juxtacrine interactions for now.
Unfortunately, this is actually quite limiting since [most interactions appear to be paracrine/autocrine](https://github.com/sqjin/CellChat/blob/master/tutorial/CellChat-vignette.Rmd?utm_source=chatgpt.com).
However, it allows us to assume that interacting cells must be next to each other, making things much easier for us quantitatively.
(It turns out [this can be relaxed later once we build up a mathematical intuition for interactions](/assets/pdf/harmonics.pdf)).
Furthermore, we'll also stick to heterotypic interactions because [they constitute the majority of immune interactions](https://www.nature.com/articles/s41586-022-05028-x?utm_source=chatgpt.com).


We can derive a representation of heterotypic juxtacrine interactions from first principles.
Based on \textbf{Figure \ref{fig:interaction_schematics}A}, we can see that for an interaction to occur between these two cells, one cell must express gene $x$ (red) but not gene $y$ (blue), while the other cell must express $y$ but not $x$.
Quantitatively, we would expect $(x_i - x_j) > 0$ and $(y_i - y_j) < 0$.
Thus, if we multiplied these differences, we would expect a large negative value overall:
$(x_i - x_j)(y_i - y_j) < 0$.
Finally, if this is a prominent interaction, we would expect to observe it frequently within the tissue.
We can account for this by calculating the above product for each pair of cells and taking the sum, weighting by the adjacency matrix to consider only neighboring cells.
Putting everything together, we see that the strength of interaction described by genes $x$ and $y$ is given by
\begin{equation} \label{eq:ixndef}
    \sum_{ij} \mathbf{A}_{ij} (x_i - x_j) (y_i - y_j) < 0.
\end{equation}

Note the resemblance between eq. (\ref{eq:ixndef}) and the definition of frequencies given by (\ref{eq:freqdef}).
They are equivalent except for that instead of comparing $x$ to \textit{itself} (i.e. squaring $x_i - x_j$), we now compare it to a \textit{different} gene, $y$.
This suggests that we can express interactions in terms of frequencies.

Just as eq. (\ref{eq:freqdef}) can be written as the quadratic form $\mathbf{x}^{\top} \mathbf{L} \mathbf{x}$, so can we write eq. (\ref{eq:ixndef}) as the bilinear form $\mathbf{x}^{\top} \mathbf{L} \mathbf{y}$.
This can be expressed explicitly in terms of covariance as 
$(\mathbf{L}^{\frac{1}{2}} \mathbf{x})^{\top} (\mathbf{L}^{\frac{1}{2}} \mathbf{y})$.
Finally, recall that any function of the Laplacian $\mathbf{L}$ is a filter.
In this case, we have
\begin{equation}
    g(\mathbf{L}) = \mathbf{L}^{\frac{1}{2}} = \mathbf{V} \mathbf{\Lambda}^{\frac{1}{2}} \mathbf{V}^{\top}
\end{equation}
which can alternatively be expressed, with a minor abuse of notation, as a filter kernel over frequency values:
\begin{equation}
    g(\lambda) = \lambda^{\frac{1}{2}}.
\end{equation}
Note that this kernel is high-pass, as it assigns higher weights to higher frequencies.
In fact, this is the same kernel used in \textbf{Chapter \ref{chap:preliminaries}} to illustrate high-pass filtering (\textbf{Figure \ref{fig:filtering}B}).
Thus, we can represent interactions in terms of negatively covarying high-frequency gene expression signals:
\begin{equation}
    \mathbf{\hat x}^{\top} \mathbf{\hat y} < 0.
\end{equation}

{% details Alternate derivation %}
Alternatively, we can derive this representation by looking at differences along the *edges* between cells.
This leads to an understanding of interactions as "opposing flows" of gene expression between cells.
Consider the differential matrix $\Delta \in \mathbb{R}^{e \times c}$ where $e$ is the number of edges in the tissue domain graph.
A given row of $\Delta$ represents a directed edge between two cells with a value of $1$ at the sending cell's index and a value of $-1$ at the receiving cell's index.
Hence this matrix is often referred to as an "incidence matrix".
An undirected edge can simply be represented as two directed edges with opposite directions.
Multiplying this differential matrix with a gene expression signal yields the differences in expression along each edge, i.e. between each pair of neighboring cells:
\begin{equation}
    \Delta \mathbf{x} \in \mathbb{R}^{e}.
\end{equation}
This is equivalent to the derivative in continuous space;
the tangent space in a graph domain is the edge space.
We can then calculate the covariance between the resulting gene flows, expressing opposing flows in terms of negative covariance:
\begin{equation}
    (\Delta \mathbf{x})^{\top} (\Delta \mathbf{y}) < 0
\end{equation}
Finally, note that $\mathbf{L} = \Delta^{\top} \Delta$.
As a result, simplifying this equation yields
\begin{equation}
    \mathbf{x}^{\top} \Delta^{\top} \Delta \mathbf{y}
    = \mathbf{x}^{\top} \mathbf{L} \mathbf{y} < 0,
\end{equation}
which can be rearranged as above into eq. (\ref{eq:ixndef}).
Thus, we find that opposing flows, too, correspond to negatively covarying high-frequency gene expression signals.
{% enddetails %}


## Simulation

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
We can convince ourselves this is the case by seeing that the extremal values occur between neighboring cells.
in this post, we will argue that complementary small-scale expression patterns correspond to cell-cell interactions.


### Simulated interaction components

Intuitively, a **group of gene expression patterns that overlap in the tissue should represent a region.**
We can find these groups by looking at pairwise relationships between gene signals.
Consider low-pass filtered gene signals $\mathbf{\hat x}_i, \mathbf{\hat x}_j \in \mathbb{R}^n$.
Because these signals are vectors and we want a scalar measure of similarity.
One way is to compare them by taking the inner product:

$$
\mathbf{\hat x}_i^{\top} \mathbf{\hat x}_j \in \mathbb{R}.
$$

With some mean centering, this is defined as the covariance.
While we will continue to refer to this as covariance, we will omit all mean centering for simplicity (although it will often add a "junk component" describing a translation, [as described previously](/blog/2025/graph-fourier#spectra)).

But we aren't just interested in one pair of genes; **we want to look at all pairwise relationships**.
Let $\mathbf{\hat X} = [\mathbf{\hat x}_1 | ... | \mathbf{\hat x}_g] \in \mathbb{R}^{n \times g}$ represent the cell-by-gene matrix of low-pass filtered gene signals.
Then the gene-by-gene covariance matrix is given by

$$
\mathbf{C} = \mathbf{\hat X}^{\top} \mathbf{\hat X} \in \mathbb{R}^{g \times g},
$$

with entries $\mathbf{C}_{ij} = \mathbf{\hat x}_i^{\top} \mathbf{\hat x}_j$.
We can visualize $\mathbf{C}$ as a heatmap.
The genes are sorted based on their corresponding ground truth region patterns, so we expect to see interesting groups of covarying genes as blocks.
(If you look closely, you'll see that these blocks are 4x4, as there are four gene markers for each pattern.)
For instance, gene markers for each region form red blocks along the diagonal.
These denote positively covarying groups of genes, i.e. gene programs.
Furthermore, different blocks appear to negatively covary, forming blue blocks along the *off*-diagonal.
Conceptually, this just means that gene markers for each region are mutually exclusive.

<figure style="text-align: center;">
  <img src="/assets/figures/interactions/interaction_covariance.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 2:</strong> Visualization of the high-pass covariance matrix.</figcaption>
</figure>

Ultimately, **we seek to distill these blocks into a simpler representation** -- perhaps by grouping genes within related blocks into a single "gene program" describing a region.
It turns out we can do this by eigendecomposing $\mathbf{C}$, i.e. performing PCA.
(For a primer on PCA, see [this post](/blog/2025/pca).)
Briefly, eigendecomposing the gene-gene covariance matrix yields the eigenbasis

$$
\mathbf{U} = [ \mathbf{u}_1 | ... | \mathbf{u}_g ] \in \mathbb{R}^{g \times g},
$$

where $\mathbf{u}_i \in \mathbb{R}^g$ represents the gene loadings for PC$i$.
In other words, it describes each gene's participation in gene program $i$.
We can visualize this matrix as a heatmap as well.
The rows represent each gene in the same order as in Figure 2.
The columns now represent PCs, or "region components", and the colors describe how much and in what direction a given gene contributes to a given component.

<figure style="text-align: center;">
  <img src="/assets/figures/interactions/interaction_components.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 3:</strong> Visualization of high-pass gene programs, i.e. "interaction components". The right-hand plot is an enlarged version of the inset in the left-hand plot.</figcaption>
</figure>

pattern across so few cells, shows up as later PC
can also rationalize based on eq. (), "avging out".
also noise is a problem.
also sensitive to reads/cell bc individual cells are so important, unlike in regions.

The first component is entirely negative and is likely just a consequence of neglecting mean centering (the "junk component" mentioned above).
The second, third, and fourth components each appear to describe relationships between regions, with each representing a positive and negative region.

We can also visualize each component in the tissue by projecting cells onto each gene program, i.e. $\mathbf{X} \mathbf{U} \in \mathbb{R}^{n \times g}$.
We end up seeing that they each describe two regions of the tissue.
The first component describes the outer ring versus everything inside of it, the second component describes the second-most outer right versus everything inside, and the third component describes the third-most outer ring versus everything inside.
We can also sort gene loadings to identify gene markers for each region component.
This gives a set of top genes describing the "positive region" and a different set for the "negative region".
We can visualize each of these markers in the tissue as well, seeing that they are indeed expressed within the regions they describe.

<figure style="text-align: center;">
  <img src="/assets/figures/interactions/interaction_components_tissue.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 4:</strong> Visualization of interaction components and their gene markers in the tissue.</figcaption>
</figure>

Altogether, we find that **each component represents a large-scale pattern between regions**.
Note that the gene marker information shown in Figure 4 is the same as the information shown in Figure 3, just in a different form.

But why is this ground truth interaction only present all the way back in component 7?
It turns out this is because of a fundamental quantitative obstacle: linearity.


### The averaging out problem

$$
\sum_{ij} \mathbf{A}_{ij} (x_i - x_j) (y_i - y_j) < 0
$$

essentially an avg over the whole tissue.
If an interaction is sparse or only occurs within a specific part of the tissue, we will end up averaging this signal out.

Noise
Averaging out


### Simulated sample-specific interactions

same sample but shuffled
gen eig from [past post](/blog/2025/geneig)

<figure style="text-align: center;">
  <img src="/assets/figures/interactions/interaction_features_compare.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 4:</strong> Visualization of interaction components and their gene markers in the tissue.</figcaption>
</figure>

in sample 2, centered on a sender where one of the interactions was before shuffling.


### Simulated higher-order interactions

components

clusters

Note that the concept of higher-order interactions is analogous to [region gradients](/blog/2025/regions$simulated-region-gradients) in that it's a **generalization of the fundamental concept to multiple components**.
Additionally,

In the previous post, we found that vertices in low-pass space described discrete regions that together define the tissue.
Here, however, we find that vertices in *high*-pass space describe discrete interactors that together define an *interaction*.

---


## Human lymph node

### Lymph node interaction components

## Mouse brain

### Mouse brain interaction components

### Mouse brain higher-order interactions

## Conclusion