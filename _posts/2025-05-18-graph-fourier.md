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

That being said, let's start exploring these quantitative fundamentals.


### The tissue domain

We can think of the tissue domain as an undirected graph over $n$ nodes, each of which represents a cell.
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

Finally, we can introduce the Laplacian matrix

$$
\mathbf{L} = \mathbf{D} - \mathbf{A}
$$

***give intuition***
For convenience, we will instead consider the edge-normalized Laplacian $\mathbf{L} = \mathbf{I} - \mathbf{D}^{-\frac{1}{2}} \mathbf{A} \mathbf{D}^{-\frac{1}{2}}$, since it's eigenvalues have some nice properties that we will leverage later.

Note that the order of cells in these matrices is arbitrary.
It doesn't matter so long as it is consistent across all related vectors and matrices.

Now that we have a simulated tissue domain, we can create gene expression signals over it.


### Transcriptional signals

A signal is a vector of values associated with each node in our graph.
In our case, we are interested in the amount that a given gene is expressed within each cell of a tissue.
We can collect all of these values into a list, yielding the vector $\mathbf{x} \in \mathbb{R}^n$, where $\mathbf{x}_i$ is the amount of gene $x$ expressed in cell $i$.
<!-- While the ordering of these values is generally arbitrary, it is critical that they follow the same ordering of cells used in the adjacency matrix and all other associated mathematical objects.
In other words, $\mathbf{A}_i$ (the row of values indicating neighbors of cell $i$) and $\mathbf{x}_i$ (the amount of gene $x$ expressed in cell $i$) are talking about the same cell $i$. -->
We can further collect all of these gene signals into the matrix

$$
\mathbf{X} = [\mathbf{x}_1 | ... | \mathbf{x}_g] \in \mathbb{R}^{n \times g},
$$

where $g$ is the number of genes measured.
This is the "cell-by-gene matrix", the [fundamental data structure underlying all of single-cell omics](https://cellxgene.cziscience.com/).

Now that we've quantitatively defined both our domain and our signals, our challenge is to combine them.


### Frequencies

While it's often said that spatial data offers more information than dissociated single-cell data, we might be better served by the intuition that spatial coordinates enable us to strategically *remove* information.
Namely, it allows us to focus on gene expression over particular length scales of interest, disentangling the mess of information into more interpretable components.
But how can we isolate a given length scale?
The first step is defining what a length scale is in the first place.

Take music, for instance, in which the different frequency ranges of bass, mids, and trebles define different *time* scales along which a sound can vary.
Similarly, our notion of *length* scale can be made quantitatively rigorous in terms of *spatial* frequencies.
While a straightforward extension of this concept to two dimensions captures spatial frequencies in an image, it's not so straightforward to extend to graphs due to their irregular topology.
Thus, we have to do a little leg work to extend this concept to our tissue domain.

We can start by slowly deriving a definition of frequency in a graph setting.
Consider a hypothetical signal $\mathbf{v}$ over the tissue.
The notion of frequency is simply "how much does the signal tend to change as I take a step in the domain?"
Taking a step in our tissue domain corresponds to moving between neighboring cells $i$ and $j$.
The change in our signal as we take this step is thus given by $\mathbf{v}_i - \mathbf{v}_j$.
However, we don't really care about the sign, so we can just square it to get $(\mathbf{v}_i - \mathbf{v}_j)^2$.
This is just one step, and we care about how the signal tends to change with steps throughout the whole tissue in general.
To capture this tendency, we can simply sum over all pairs of neighboring cells, yielding our final definition of frequency

\begin{equation} \label{eq:freqdef}
  \lambda = \sum_{ij} \mathbf{A}_{ij} (\mathbf{v}_i - \mathbf{v}_j)^2
\end{equation}

Note that because we are summing over *all* possible pairs of cells $i$ and $j$, we have to multiply each one by $\mathbf{A}_{ij}$ so that we only consider *neighboring* pairs.

This definition of frequency for a given signal might make sense, but what we are really looking for is an ideal *set* of signals that represent *all possible frequencies*, much as time scales are given by sine waves of all possible frequencies.
It turns out that, because our tissue domain is finite and discrete, we can actually solve for a finite set of all possible frequencies, and it ends up being a basis in the linear algebraic sense.

To see this, let's first rewrite eq. \eqref{eq:freqdef} as a "[quadratic form](https://gregorygundersen.com/blog/2022/02/27/positive-definite/)", which works in our favor by getting us a step further into the realm of linear algebra:

$$
\lambda = \mathbf{v}^{\top} \mathbf{L} \mathbf{v}.
$$

{% details How? %}

Eq. \eqref{eq:freqdef} can be converted into a quadratic form via a process reminiscent of annealing.
First, we heat it up by factoring the
$\mathbf{v}$
terms to get

$$
\sum_{i,j}\mathbf{A}_{ij}(
v_i v_i +
v_j v_j -
v_i v_j -
v_j v_i
).
$$

We can then begin to cool it down by combining positive and negative terms, respectively, yielding

$$
2 \sum_{i,j}\mathbf{A}_{ij}
v_i v_i -
2 \sum_{i,j}\mathbf{A}_{ij}
v_i v_j.
$$

Note that, for a fixed row $i$, the left-hand term corresponds to summing
$v_i v_i$ together
$d_i$ times, where $d_i$ is the sum of the $i$th row of
$\mathbf{A}$.
Thus, one could equivalently express the left-hand term as

$$
2 \sum_{i}\mathbf{D}_{ii}
v_i v_i.
$$

Now both the left and right terms themselves correspond to quadratic forms, yielding

$$
2 \mathbf{v}^{\top} \mathbf{D} \mathbf{v} - 
2 \mathbf{v}^{\top} \mathbf{A} \mathbf{v}.
$$

Further simplifying, we have

$$
2 \mathbf{v}^{\top} (\mathbf{D}-\mathbf{A}) \mathbf{v}
$$

and further

$$
2 \mathbf{v}^{\top} \mathbf{L} \mathbf{v}.
$$

The factor of two arises from double counting each edge, which corresponds to a directed edge in each direction.
Because we tend to think of each undirected edge as only a single edge going both ways, folks tend to omit this factor, yielding the final expression

$$
\mathbf{v}^{\top} \mathbf{L} \mathbf{v}.
$$

{% enddetails %}

It turns out we can rearrange this expression into an eigenvalue problem:

$$
\lambda \mathbf{v} = \mathbf{L} \mathbf{v}.
$$

{% details How? %}

Starting with the quadratic form $\lambda = \mathbf{v}^{\top} \mathbf{L} \mathbf{v}$, we can normalize it to guarantee that the overall expression strength doesn't influence the frequency value.
After all, we really only care about the relative spatial distribution of the signal, not its overall magnitude.
We can thus normalize by dividing out the magnitude:

$$
\lambda = \frac{\mathbf{v}^{\top} \mathbf{L} \mathbf{v}}{\mathbf{v}^{\top} \mathbf{v}}.
$$

Then, with a bit of rearranging, we end up with an eigenvalue problem:

$$
\begin{align}
    & \lambda = {\frac{\mathbf{v}^{\top} \mathbf{L} \mathbf{v}}{\mathbf{v}^{\top} \mathbf{v}}} \nonumber \\
    &\rightarrow \lambda \mathbf{v}^{\top} \mathbf{v} = \mathbf{v}^{\top} \mathbf{L} \mathbf{v} \\
    &\rightarrow \lambda \mathbf{v} = \mathbf{L} \mathbf{v}. \nonumber
\end{align}
$$

{% enddetails %}


This is a critical insight because $\mathbf{L}$ is symmetric positive semidefinite and thus has an eigenbasis of eigenvectors with real, nonnegative eigenvalues.
This eigenbasis is given by the matrix

$$
\mathbf{V} = [\mathbf{v}_1 |...| \mathbf{v}_n] \in \mathbb{R}^{n \times n}.
$$

Let's plug all of this terminology into our context of interest by realizing two key points.
**First** of all, we found that there exists a finite set of ideal signals ${\mathbf{v}_1, ..., \mathbf{v}_n}$ that represent "all possible frequencies" -- and thus all possible length scales -- over our tissue domain.
This is great because it means we can rigorously talk about any length scale of interest.
Below are some examples plotted in the tissue to illustrate this intuition.
Note that no gene expression information went into calculating these frequencies; rather, they are an abstract representation of variation on different length scales.

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/frequencies.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 3:</strong> Examples of different frequencies over the tissue domain.  </figcaption>
</figure>

{% details Why do highs appear constrained to one part of the tissue? %}
When taking a look at $\mathbf{v}_{301}$, the fluctuations appear constrained to the lower left portion of the tissue.
Uncertainty principle
How I see it: consequence of "irregular" topology
If all cells had the same degree:
But because they don't, we have:
{% enddetails %}

The **second** key point is that this set of ideal frequencies forms a basis in the linear algebraic sense.
This means that when we do measure a gene expression signal, we can project it into this "frequency space" to quantify its prevalence on each length scale.

That's what we'll do next.


### Spectra

We can think of projecting a given gene expression signal $\mathbf{x}$ into frequency space as comparing it to each frequency $\mathbf{v}_i$.
For a single frequency, this is given by the inner product $\mathbf{v}_i^{\top} \mathbf{x} \in \mathbb{R}$.
For all frequencies, this is given by the matrix product $\mathbf{s} = \mathbf{V}^{\top} \mathbf{x} \in \mathbb{R}^n$, where the output is the similarity of $\mathbf{x}$ to each of $\mathbf{v}_1, ..., \mathbf{v}_n$.
The vector $\mathbf{s}$ is often referred to as the signal's "spectrum".
An analogy I tend to think of is that the original gene expression signal over the tissue is like a dish you might cook.
You could always think of that dish in terms of its corresponding *recipe*, i.e. the amounts of each ingredient necessary to construct it.
In this case, the recipe is the spectrum, and the ingredients are the frequencies.

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/spectra.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 3:</strong> An example gene expression signal in tissue space and in frequency space.  </figcaption>
</figure>

Note that spectra generally do contain negative values.
Our visualization above involved taking the absolute value of the spectrum to better convey the intuition of how prevalent a signal is over a given length scale.
Additionally, we chose not to visualize the first value $s_1$, as it just corresponds a scaling factor.

{% details How is $s_1$ is just a scaling factor? %}
Nice
{% enddetails %}

There are many interesting things we could do with spectra.
You might think of comparing gene expression signals based on their spectra, perhaps to find classes of spectral patterns (i.e. biologically-relevant length scales) using PCA or NMF.
However, there are also many interesting issues with these ideas.
For instance, a spectrum has the same dimension as its corresponding tissue.
This makes comparison across different tissues difficult because the dimensions of spectra likely differ.
We'll explore these pitfalls and opportunities in a future blog post.

For now, let's instead focus on modifying these spectra, i.e. performing filtering.


### Filtering

In the language of the above analogy, modifying spectra allows us to ask "what happens to the dish when I remove this ingredient?"

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

We can put all of these steps together to define a single filter function

$$
\begin{align}
  & \mathbf{V} e^{-\tau \mathbf{\Lambda}} \mathbf{V}^{\top} \\
  & = e^{-\tau \mathbf{L}} \\
  & = f(\mathbf{L}),
\end{align}
$$

where f(\lambda)

This gives a glimpse into an interesting fact: any (analytic) filter $h(\lambda)$ can be described as a function of the Laplacian matrix:

$$
h(\mathbf{L}) = \mathbf{V} h(\mathbf{\Lambda}) \mathbf{V}^{\top}
$$


Alternatively, we could emphasize the highs by applying, say, a square-root filter.
(This will be an important kernel for defining interactions in a later post.)

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