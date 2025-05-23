---
layout: distill
title: Transcriptional signals over tissue domains
description: An introduction to graph signal processing in biological tissues
tags:
giscus_comments: false
date: 2025-05-14
featured: true
tags: spatial-omics

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
  - name: The Fourier transform
  - name: The tissue domain
  - name: Transcriptional signals
  - name: Frequencies
  - name: Spectra
  - name: Filtering
    # if a section has subsections, you can add them as follows:
    # subsections:
    #   - name: Example Child Subsection 1
    #   - name: Example Child Subsection 2

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }
---


Here, we will build an intuition for how to describe frequencies over graphs in terms of gene expression over biological tissues.
We will use simulations to


## The Fourier transform

When we think of music, 

This kind of thinking also applies to space rather than time.
For instance, it could be applied to images, which are not one dimensional and continuous, but rather two dimensional and discrete.
But the notion of frequency remains;

But what if some of these connections between pixels were removed?
What if some were added?
In other words, we can ask more generally: what if our domain was a graph?

---


## The tissue domain

There are many different examples of graph domains, including social networks.
The domain that we will focus on here is that of biological tissues, in which one might imagine hopping from cell to cell 

We can think of a "tissue domain" as an undirected graph over $n$ nodes, each of which represents a cell.
We could construct this graph in many ways, including connecting each cell to its k nearest physical neighbors.
Personally, I prefer using a Delaunay triangulation, as it creates a mesh that's embeddable in 2D, which respects my own visual intuition.
If you're optimistic, you might also believe that it [captures the mechanical forces present in biological tissues](https://pubmed.ncbi.nlm.nih.gov/20082148/).

To demonstrate this process, let's create a simulated tissue domain.
Our domain will simply consist of a bunch of cells scattered uniformly within a unit circle.
After performing a Delaunay triangulation, we can zoom in to see that the cells are indeed connected to their spatial neighbors to form a 2D mesh.

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/tissue_domain.png"
       alt=""
       style="width:80%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 1:</strong> Cells arranged in a spatial graph to form a tissue domain. </figcaption>
</figure>

Now let's define the key mathematical objects associated with this tissue domain.
Our graph can be represented by the symmetric adjacency matrix
$$
\mathbf{A} \in \{0,1\}^{n \times n}.
$$
Each entry of $\mathbf{A}$ is either $1$, which represents two cells that are spatially adjacent, or $0$, which represents two cells that are not adjacent.
While we could weight these edges based on physical distances between cells, we will instead stick to simple binary edges for simplicity.
We also will not consider self-loops, i.e. we have 
$
\mathbf{A}_{ii} = 0.
$
Finally, the number of neighbors, or degree, of each cell $i$ is given by the diagonal degree matrix
<!-- $
\mathbf{D} \in \mathbb{R}^{n \times n}
$
 with entries
$
\mathbf{D}_{ii} = \sum_j \mathbf{A}_{ij}
$
. -->

---


## Transcriptional signals

A transcriptional signal is

We can add some to our simulation.
In real tissues, they might vary in their spatial scale, some looking like a region and some looking noisy.
Allen atlas


---

## Frequencies

<!-- {% details Why not the the first frequency? %}
The first of the frequency bases represents a constant signal over the graph, i.e. the smoothest signal possible.
We could also see this by taking a signal we know is constant -- say the ones vector $$ -- and plugging it into the eigenvalue equation
$$
\mathbf{L} \mathbf{1} = \lambda \mathbf{1}
$$.

{% enddetails %} -->

<figure style="text-align: center;">
  <img src="/assets/figures/fourier/frequency.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
  <figcaption><strong>Figure 2:</strong> Cells arranged in a spatial graph form a tissue domain. </figcaption>
</figure>

---


## Spectra


---

## Filtering