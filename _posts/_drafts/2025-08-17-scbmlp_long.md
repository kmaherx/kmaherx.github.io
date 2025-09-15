---
layout: distill
title: "Bilinear MLPs for Single-Cell Analysis"
description: Weights-based interpretability applied to single-cell transcriptomics analysis
tags:
giscus_comments: false
date: 2025-08-17
featured: true
categories: mech-interp
thumbnail: assets/figures/scbmlp/thumbnail.png

authors:
  - name: Kamal Maher
    affiliations:
      name: Independent

toc:
  - name: Introduction
    subsections:
        - name: Mechanistic interpretability for scientific discovery
        - name: Bilinear MLPs
        - name: Applications to single-cell transcriptomics
        - name: Roadmap
        - name: Disclaimers
  - name: Cell type classification
    subsections:
        - name: Cell types in the developing pancreas
        - name: Cell type model training
        - name: Cell type model interpretation
        - name: Cell type summary
  - name: Frequency regression
    subsections:
        - name: Transcriptional frequencies in the developing pancreas
        - name: Frequency model training
        - name: Frequency model interpretation
        - name: Frequency summary
  - name: Perturbation regression
    subsections:
        - name: Perturbation dataset
        - name: Perturbation model training
        - name: Perturbation model interpretation
        - name: Perturbation summary
  - name: Conclusion
---

## Introduction

### Mechanistic interpretability for scientific discovery

Mechanistic interpretability treats trained networks as scientific objects.
It's an attempt to recover the internal basis on which a model makes decisions, not just verify that its outputs correlate with labels.
Recent work has shown this is possible even for high‑performing systems.
For example, [researchers extracted superhuman tactical chess concepts from AlphaZero](https://arxiv.org/abs/2310.16410) and taught them to chess grandmasters.
Similar approaches in biological foundation models such as [Evo2](https://www.biorxiv.org/content/10.1101/2025.02.18.638918v1) have revealed emergent sequence‑level structure (rRNA, alpha helices, [phylogenetic organization](https://x.com/GoodfireAI/status/1960749185940250748)) by analyzing internal representations.
These successes hint that carefully chosen inductive biases can make models simultaneously powerful and dissectable.

Here we explore a complementary, lighter‑weight strategy.
Instead of searching through high‑dimensional activation spaces post hoc, we directly read interpretable structure from the weights of a purposely constrained architecture trained on single‑cell transcriptomic tasks.

### Why bilinear MLPs?

Standard MLP layers mix inputs through two affine transforms and a nonlinearity.
Their expressivity comes at the cost of weights that are hard to map onto biologically meaningful modules.
A [bilinear MLP layer](https://arxiv.org/abs/2410.08417) replaces that opaque composition with element‑wise products of two linear projections.
Each output logit (or regression target) becomes a quadratic form in the input gene expression vector:

$$ \mathbf{y}_a = \mathbf{x}^{\top} \mathbf{Q}_a \mathbf{x}. $$

Training therefore yields, for every output dimension $a$, a gene-gene interaction matrix $\mathbf{Q}_a$.
Eigendecomposing the (symmetrized) $\mathbf{Q}_a$ plays the same role that PCA loadings or WGCNA modules do—except the resulting eigenvectors are immediately tied to predictive performance for that specific task output.
Instead of asking “which post hoc attribution heuristic lights up this gene set?”, we ask “what gene combination of gene modules enables the model to succeed?”
Practically, each task follows the same recipe: we train the model, pull out each output’s $\mathbf{Q}_a$, symmetrize it, run an eigendecomposition, and interpret the dominant eigenvectors as candidate causal gene modules.

### Why single‑cell data?

Single‑cell RNA‑seq analysis today stacks heuristic transformations (QC, normalization, selection of highly variable genes, PCA, neighborhood graph construction, embedding, clustering, marker calling).
Biological interpretation usually arrives only at the end, divorced from the geometry that drove separation.
More expressive models (deep MLPs, VAEs like scVI, diffusion models, emerging foundation models) capture richer structure but often require auxiliary attribution to become explainable.

We would prefer something interpretable by construction -- parameters that are simultaneously the mechanism of prediction and the object of biological explanation.
A single bilinear layer gives exactly that.
Each output produces a symmetric interaction matrix whose eigenvectors surface the gene modules the model actually leverages.
This trades some depth for immediate inspectability.

### Tasks and roadmap

We apply the same bilinear layer to three single-cell transcriptomics tasks:
1. **Cell type classification** (relies on noisy human annotations).
2. **Regression onto intrinsic “transcriptional frequencies”** that summarize large‑scale variation without manual labels.
3. **Perturbation identity/dose regression**, tying interaction spectra to externally applied interventions.

Across tasks the workflow is constant: train; extract $\mathbf{Q}_a$ per output; symmetrize; eigendecompose; interpret eigenmodes as candidate causal gene modules.

### Disclaimers

This is an exploratory proof‑of‑concept meant to probe whether bilinear layers can surface biologically sensible gene interaction modules across heterogeneous supervision regimes. Several caveats apply:

- **Not a benchmark**: We did not exhaustively tune architectures, regularization, or preprocessing; performance should be read as “good enough to interpret,” not state of the art.
- **Biological caution**: Module narratives (gene lists, GO enrichments, pathway labels) are hypotheses, not validated mechanisms. Some assignments inevitably reflect generic stress, cell cycle, or housekeeping signals.
- **AI assistance**: Large language models were used to help condense gene/function summaries and GO term groupings. Manual curation removed egregious hallucinations, but subtle over‑interpretations may remain.
- **Reproducibility**: All code (including scripts and notebooks) is available in the public repository: [github.com/kmaherx/ScBMLP](https://github.com/kmaherx/ScBMLP). Random seeds were fixed where feasible, but certain GPU/cuDNN nondeterminisms may perturb eigenvalue ordering.
- **Annotation noise**: Cell type labels inherit uncertainty from upstream manual or semi‑automated curation; mislabeling can shift which interaction spectra appear salient.
- **Single layer only**: We intentionally restrict ourselves to one bilinear layer because stacking would produce higher‑order tensors whose interpretability toolkit is immature.


---

## Cell type classification

### Cell types in the developing pancreas

We begin by training and interpreting a model on a simple task: cell type classification.
The [dataset](https://scvelo.readthedocs.io/en/stable/scvelo.datasets.pancreas.html#scvelo.datasets.pancreas) we will use consists of [developing endocrine cells from the mouse pancreas](https://journals.biologists.com/dev/article/146/12/dev173849/19483/Comprehensive-single-cell-mRNA-profiling-reveals-a).
It provides pre-existing, expert-labeled cell types as well as continuous developmental dynamics which will prove illustrative in the next section as well.

Let's first visualize the data by plotting cells in UMAP space colored by cell type.
Note that the UMAP coordinates used were precomputed when downloaded using the `scvelo` package.
The developmental trajectory is visible, forming a long axis from ductal cells on the left to alpha and beta cells on the right.
Finer structure can be seen in the separation between epsilon/alpha and beta trajectories, which appear to weave past one another.
We will quantify these decreasing scales of transcriptional variation in the next section, but for now we will focus on predicting cell types.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/cell_type/cell_type_umap_v2.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>


### Cell type model training

In order to get a $\mathbf{Q}_a$ worth interpreting, we must train our bilinear MLP model.
Here are the model and training details:
- **Data**: `scv.datasets.pancreas()`; remove genes starting with Rpl/Rps; normalize total + log1p (layer `spliced`).
- **Features**: top 10,000 HVGs.
- **Split**: 70% train / 30% val; no test set (exploratory).
- **Architecture**: bilinear layer; two linear projections $\mathbf{W},\mathbf{V}$ (10k→128) whose element‑wise product feeds a linear head $\mathbf{P}$ to $t$ cell type logits (additional mathematical details provided in the following section on interpretation).
- **Objective**: cross entropy.
- **Optimizer**: Adam, lr = 1e-4, weight decay = 0.35.
- **Schedule**: cosine annealing over 100 epochs.
- **Batch** size: 64.
- **Device**: CPU; seeds fixed (Python, NumPy, Torch) for reproducibility.
- **Biases**: much like the original paper, we observed similar performance with/out bias terms and thus omitted them to make downstream interpretation mathematically simpler.

Below are the loss and accuracy curves for train and val splits.
The gap between train and val indicated overfitting, but we chose not to pursue further optimization because we believed the 90% validation accuracy indicated the model had learned something worth interpreting.
Additionally, the cell type labels relied on human input and are thus susceptible to error themselves, perhaps setting a ceiling on performance.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/cell_type/cell_type_training.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

We next plotted some simple model performance metrics to better understand our model's performance.
By breaking down accuracy across cell types, we found that the model performed uniquely poorly on delta cells: ~40% accuracy compared to the ~90% accuracy for all other cell types.
As seen in the confusion matrix, they were often mislabeled as pre-endocrine cells.
Whether this is a shortcoming of the model is difficult to say due to the potential ground truth cell type labeling error.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/cell_type/cell_type_model_analysis.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Altogether, the model learned to classify cell types reasonably well.
Thus, we believed it may have learned an internal representation worth interpreting.

### Cell type model interpretation

#### Math

Now that we have an idea of what the data and model look like, we can introduce an intuitive derivation of the key mathematics behind our approach.
Note that the core mathematics provided here is just a recap of work from [the original bilinear MLP paper](https://arxiv.org/abs/2410.08417).

Let's start by defining some dimensions.
- $g$: number of genes
- $h$: hidden dimension
- $t$: number of cell types

Weights $\mathbf{W}, \mathbf{V} \in \mathbb{R}^{h \times g}$ map from transcriptional space to hidden space.
An additional linear transformation, $\mathbf{P} \in \mathbb{R}^{t \times h}$ then maps from hidden space to cell type space.
We will assume for simplicity that $\mathbf{P}$ is "folded into" the weights, allowing them to map directly from transcriptional space to cell type space.

\begin{equation}
  \mathbf{P} \mathbf{W} \rightarrow \mathbf{W}  \in \mathbb{R}^{t \times g} \nonumber
\end{equation}
\begin{equation}
  \mathbf{P} \mathbf{V} \rightarrow \mathbf{V} \in \mathbb{R}^{t \times g} \nonumber
\end{equation}

A bilinear layer is defined as

\begin{equation}
  g(\mathbf{x}) = \mathbf{W} \mathbf{x} \odot \mathbf{V} \mathbf{x}, \nonumber
\end{equation}

where input $\mathbf{x} \in \mathbb{R}^g$ represents a given cell's transcriptional profile, and the "o-dot" corresponds to element-wise multiplication.
We omit biases for conceptual clarity.
Consider just one of the output dimensions (i.e cell types), $a$.
We end up with the equation

\begin{equation}
  \mathbf{w}_a^{\top} \mathbf{x} \odot \mathbf{v}_a^{\top} \mathbf{x}. \nonumber
\end{equation}

Each side of the equation is now a scalar, so element-wise multiplication is just a simple multiplication between two numbers, and we can get rid of the o-dot.
We can also take the transpose of the left-hand side (i.e. swap the order of $\mathbf{x}$ and $\mathbf{w}_a$ and place the transpose on $\mathbf{x}$) since the output is just a scalar and would remain unchanged.
Applying both of these changes, we get

\begin{equation}
  (\mathbf{x}^{\top} \mathbf{w}_a) (\mathbf{v}_a^{\top} \mathbf{x}). \nonumber
\end{equation}

Finally, we can remove the parentheses and define the cell type-specific gene-gene interaction matrix $\mathbf{Q}_a = \mathbf{w}_a \mathbf{v}_a^{\top} \in \mathbb{R}^{g \times g}$, resulting in a bilinear (or, perhaps more accurately, quadratic) form

\begin{equation}
  \mathbf{x}^{\top} \mathbf{Q}_a \mathbf{x}. \nonumber
\end{equation}

Notice that only the symmetric part of $\mathbf{Q}_a$ is relevant to our analysis, as our inputs are always symmetric.
Thus, we can redefine our interaction matrix as only its symmetric part.
\begin{equation}
  \frac{1}{2} (\mathbf{Q}_a + \mathbf{Q}_a^{\top}) \rightarrow \mathbf{Q}_a \nonumber
\end{equation}

{% details Justification of symmetrization %}
Because $\mathbf{Q}_a$ is a square matrix, it can be decomposed into symmetric and anti-symmetric parts.
\begin{equation}
    \mathbf{Q}_a = \frac{1}{2} (\mathbf{Q}_a + \mathbf{Q}_a^{\top}) + \frac{1}{2} (\mathbf{Q}_a - \mathbf{Q}_a^{\top}) \nonumber
\end{equation}
It follows that the quadratic form $\mathbf{x}^{\top} \mathbf{Q}_a \mathbf{x}$ can also be decomposed this way.
The symmetric part is given by
\begin{equation}
  \frac{1}{2} \mathbf{x}^{\top} (\mathbf{Q}_a + \mathbf{Q}_a^{\top}) \mathbf{x} = \frac{1}{2} (\mathbf{x}^{\top} \mathbf{Q}_a \mathbf{x} + \mathbf{x}^{\top} \mathbf{Q}_a^{\top} \mathbf{x}). \nonumber
\end{equation}
and the antisymmetric part by
\begin{equation}
  \frac{1}{2} \mathbf{x}^{\top} (\mathbf{Q}_a - \mathbf{Q}_a^{\top}) \mathbf{x} = \frac{1}{2} (\mathbf{x}^{\top} \mathbf{Q}_a \mathbf{x} - \mathbf{x}^{\top} \mathbf{Q}_a^{\top} \mathbf{x}). \nonumber
\end{equation}
Because $\mathbf{x}^{\top} \mathbf{Q}_a \mathbf{x} \in \mathbb{R}$,
\begin{equation}
  \mathbf{x}^{\top} \mathbf{Q}_a \mathbf{x} = (\mathbf{x}^{\top} \mathbf{Q}_a \mathbf{x})^{\top} = \mathbf{x}^{\top} \mathbf{Q}_a^{\top} \mathbf{x}. \nonumber
\end{equation}
This means the antisymmetric part is zero.
\begin{equation}
  \frac{1}{2} (\mathbf{x}^{\top} \mathbf{Q}_a \mathbf{x} - \mathbf{x}^{\top} \mathbf{Q}_a^{\top} \mathbf{x}) = 0. \nonumber
\end{equation}
Thus, we take only the symmetric part.
{% enddetails %}

The symmetric gene-gene interaction matrix $\mathbf{Q}_a$ can be interpreted similarly to a gene-gene covariance matrix in conventional single-cell analysis.
Just as a gene-gene covariance matrix can be eigendecomposed to identify prominent gene modules within the data, so can $\mathbf{Q}_a$ be eigendecomposed to find prominent gene modules related to cell type $a$.
However, the modules calculated from $\mathbf{Q}_a$ have more of a causal interpretation in that they directly influence our model's ability to predict cell type $a$.
Symmetrization of $\mathbf{Q}_a$ makes this interpretation much simpler, as all eigenvalues become real by the spectral theorem.

<!-- 
{% details Justification of bias omission %}
A bilinear layer with bias terms is given by
\begin{equation}
  g(\mathbf{x}) = (\mathbf{W} \mathbf{x} + \mathbf{b}) \odot (\mathbf{V} \mathbf{x} + \mathbf{c}), \nonumber
\end{equation}
with biases $\mathbf{b}, \mathbf{c} \in \mathbb{R}^t$.
Considering output cell type $a$ and using simplification steps explained above, we have
\begin{equation}
  (\mathbf{w}_a^{\top} \mathbf{x} + b_a) (\mathbf{v}_a^{\top} \mathbf{x} + c_a). \nonumber
\end{equation}
Factoring leads to
\begin{equation}
  \mathbf{x}^{\top} \mathbf{w}_a \mathbf{v}_a^{\top} \mathbf{x} + c_a \mathbf{w}_a^{\top} \mathbf{x} + b_a \mathbf{v}_a^{\top} \mathbf{x} + b_a c_a.
\end{equation}
{% enddetails %} -->


<br>

#### Ductal

We'll look at cell type results in order of developmental trajectory, starting with ductal cells.
Ductal cells are the epithelial cells that line the pancreatic ductal tree.
During endocrinogenesis they form the tubular scaffold for secretion and act as a progenitor/support niche and signaling source for emerging endocrine cells.

We would primarily expect the model to have learned to separate ductal cells from all others, as that's the crux of the task.
We can visualize the model's ability to do this by projecting cells onto the resulting gene modules (i.e. eigenvectors of $\mathbf{Q}_{Ductal}$) and visualizing each cell type's distribution along that module space.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/cell_type/Ductal_modules.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

As expected, the first module appears to separate the ductal cell distribution from the rest (i.e. red on the right, others on the left).
Note that the direction of separation is arbitrary.
The model could very well have separated the ductal cells toward the left instead and pushed the rest to the right.
We'll see this with other cell types in a moment.
This is because the signs of each eigenvector are arbitrary; it's still an eigenvector even if it's multiplied by $-1$, so solvers essentially have to return them at random.
That means we should only read into the separation -- not that positivity/negativity carry any interpretable significance.

It seems that only the first module separates ductal cells from all others.
We should get a sense of that from the eigenvalues, which represent module strength.
This is analogous to PCA, in which eigenvalues describe the variance explained by each eigenvector, i.e. PC.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/cell_type/Ductal_eigvals.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Indeed, the first eigenvalue is far greater than the others.
This supports our observation that only the first module provides the desired separation between cell types.
Thus, for biological interpretations in this cell type, we will focus only on the first module.

Here are some brief details describing our biological interpretation workflow:
- **Gene ranking**: Calculate the top 1% of genes by loading per sign (top 1% = 100 genes per sign from the 10,000 HVGs). Signs are arbitrary; we treat the “positive” and “negative” sides strictly as opposing poles on a one‑dimensional axis, not inherently activating/repressing directions. We will refer to the top genes for a given side as a gene program, e.g. "positive program" or "negative program".
- **GO enrichment**: For each side we performed over‑representation analysis with Enrichr (gseapy) across the combined libraries: `GO_Biological_Process_2023`, `GO_Molecular_Function_2023`, `GO_Cellular_Component_2023`, `KEGG_2019_Mouse`, `Reactome_2022`, `MSigDB_Hallmark_2020`, `WikiPathways_2019_Mouse` (`organism="mouse"`). Benjamini–Hochberg adjusted p‑values supplied by Enrichr were used; terms with adjusted $p \leq 0.05$ were retained and the top 15 (by adjusted $p$) were displayed per side. No additional manual term size filtering or mitochondrial/ribosomal/generic “cellular process” pruning was applied beyond earlier preprocessing (ribosomal Rpl/Rps genes removed prior to HVG selection).
- **Interpretation limits**: Quadratic weight structure encodes statistical associations leveraged for prediction, not proven causal regulatory interactions. GO term enrichment compounds multiple hypothesis layers (gene selection + eigenvector choice); we therefore treat labels as hypotheses requiring experimental validation.

Applying this interpretation workflow to ductal cell module 0, we find that the positive program contained many **transcriptional regulators** (*Pdx1*, *Hhex*, *Sox4*) and **epithelial and surface markers** (*Cd24a*, *Prrg2*), both of which are indicative of ductal cells.
On the other hand, the negative program consisted of several mature endocrine cell markers (*Gcg*, *Pyy*, *Iapp*, *Gast*, *Cpe*, *Ttr*).
Together, this indicates that module 0 effectively separates out ductal cells on the basis of developmental stage, as one might intuitively do based on the UMAP visualized above.

In all following sections, per‑cell‑type “Module” expandable tabs follow this template (Overall interpretation → Top genes (per sign) → Key GO terms) produced by the above pipeline.
This should provide comprehensive detail to those who are interested.

{% details Module 0 %}
**Overall interpretation**
- Positive side: Developmental / progenitor and early endocrine specification program combining transcriptional regulators (*Pdx1*, *Hhex*, *Sox4*), morphogen/secreted factors (*Ghrl*, *Sst*, *Enho*), and epithelial / surface markers (*Cd24a*, *Prrg2*). Sparse classical secretory granule machinery suggests a less terminally differentiated duct-associated / progenitor‑leaning population retaining endocrine lineage priming.
- Negative side: Mature endocrine / secretory granule and peptide hormone module enriched for polyhormonal cargo (*Gcg*, *Pyy*, *Iapp*, *Gast*, *Cpe*, *Ttr*) plus metabolic / signaling scaffolds (*Gnas*, *Malat1*). GO terms highlight ER protein processing, vesicle / granule lumen, platelet (secretory) degranulation, and peptide hormone metabolism—typical of an actively secreting endocrine compartment.
- Comparison: Axis separates regulatory priming & developmental transcriptional control (positive) from established multi‑hormone secretory function (negative) – an early–to–mature endocrine differentiation contrast embedded within cells labeled Ductal.

**Top 10 genes**
- Positive: *Ghrl*, *Cd24a*, *Npepl1*, *Sst*, *Pdx1*, *Enho*, *Hhex*, *Prrg2*, *Sox4*, *Mdk*
- Negative: *Gcg*, *Pyy*, *Ttr*, *Tmem27*, *Gnas*, *Malat1*, *Arx*, *Cpe*, *Iapp*, *Gast*

**Key GO terms**
- Positive: Maturity Onset Diabetes of the Young (developmental TFs); Pancreas Beta Cells (early regulators); Cellular Response to Glucose; Insulin Secretion (early priming)
- Negative: Pancreas Beta Cells; Protein Processing in ER; Peptide Hormone Metabolism; Platelet/Secretory Degranulation; Gap Junction
{% enddetails %}

{% details Module 1 %}
**Overall interpretation**
- Positive side: Oxidative / metabolic stress adaptation with cytoskeletal & junctional remodeling (*Cd24a*, *Hhex*, *Ghrl*, *Jun*, *Arg1*, *Gp x3*, *Epcam*) plus calcium / contractility elements (smooth muscle contraction, tight junction). Secretory granule lumen enrichment indicates a transitional state ramping vesicle biogenesis under redox and hypoxic pressure.
- Negative side: Core endocrine differentiation & insulin secretory machinery (Pancreas β cell genes, *Mafb*, *Pdx1*, *Nkx6-1*, *Ins1/2*, *Nnat*, *Chgb*) with vesicle trafficking and tubulin folding pathways; lower stress/reactive signatures but high specialized processing (insulin secretion, KRAS signaling up – often linked to beta maturation signaling axes).
- Comparison: Stress‑remodeling + structural plasticity (positive) versus stabilized insulin / β‑identity program (negative); reflects a maturation / stabilization gradient after redox & junctional remodeling.

**Top 10 genes**
- Positive: *Sst*, *Cd24a*, *Hhex*, *Ghrl*, *Spp1*, *Jun*, *Arg1*, *Mdk*, *Gpx3*, *Epcam*
- Negative: *Mafb*, *Slc38a5*, *Pdx1*, *Chgb*, *Nnat*, *Nkx6-1*, *Ins1*, *Neurog3*, *Ins2*, *Ociad2*

**Key GO terms**
- Positive: Secretory Granule Lumen; Tight Junction; Hypoxia; Reactive Oxygen Species Pathway; Smooth Muscle Contraction
- Negative: Pancreas Beta Cells; Maturity Onset Diabetes of the Young; Insulin Secretion; Gap Junction; Peptide Hormone Metabolism
{% enddetails %}

{% details Module 2 %}
**Overall interpretation**
- Positive side: Proliferative / cell cycle & transcriptional regulation cluster (Mitotic spindle, G2/M checkpoint, TP53 regulation) with AP‑1 factors (*Jun/Jund*), chromatin & transcription co‑regulators, and mixed early endocrine TFs (*Nkx6-1*, *Neurog3*, *Foxa2*)—indicative of cycling endocrine progenitors engaging division and lineage specification simultaneously.
- Negative side: ER / secretory chaperone and insulin processing program (*Hspa5*, *Calr*, *Pdias*, *Iapp*, *Sst*) plus platelet / complement & peptide hormone metabolism – a differentiated hormone‑processing state with metabolic stress buffering (UPR, mTORC1 signaling, hypoxia).
- Comparison: Division/transcriptional remodeling (positive) opposed to high-load secretory ER maturation (negative); suggests a proliferative to secretory commitment transition.

**Top 10 genes**
- Positive: *Neurog3*, *Chgb*, *Krt7*, *Fev*, *Hmgn3*, *Cystm1*, *Tm4sf4*, *Cck*, *8430408G22Rik*, *Cdkn1a*
- Negative: *Ghrl*, *Pyy*, *Iapp*, *Rbp4*, *Pdia6*, *Dlk1*, *Calr*, *Nnat*, *Hspa5*, *Sdf2l1*

**Key GO terms**
- Positive: Mitotic Spindle; G2/M Checkpoint; Pancreas Beta Cells; TP53 Transcriptional Regulation; EGFR1 Signaling
- Negative: Protein Processing in ER; Pancreas Beta Cells; Unfolded Protein Response; Peptide Hormone Metabolism; Antigen Processing & Presentation
{% enddetails %}


<br>

#### Ngn3 high EP

Ngn3-high endocrine progenitors (Ngn3-high EP) are a transient, rare population in the developing pancreas characterized by high expression of the transcription factor *Neurog3* (which codes for the protein Ngn3).
These cells are committed endocrine precursors that are transcriptionally immature and plastic.
They are enriched for developmental programs (Notch/Delta and lineage TFs), often show proliferative signatures, and will differentiate into the various hormone‑producing endocrine cell types (α, β, δ, etc.) as development proceeds.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/cell_type/Ngn3_modules.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

At first glance, it appears the purpose of module 0 is instead to separate out alpha cells on the right.
However, Ngn3 high EP cells are also separated out toward the left.
They're just a bit mixed with pre-endocrine cells, which makes sense given that they represent similar developmental stages.
Interestingly, the top gene associated with the positive side was *Neurog3*, the gene coding for Ngn3, despite being the opposite side from Ngn3 high EP cells.
However, this may not be that surprising given that *Neurog3* is involved in alpha cell development.
Additional genes in the positive program included canonical alpha cell markers such as *Gcg* and *Arx*.
On the other hand, the negative program consisted of more general secretory stress (*Hspa5*, *Calr*, *Gnas*, *Aldoa*, *Gapdh*) and hormone activity (*Sst*, *Pyy*, *Ttr*, *Ghrl*).
Such stress adaptation could correspond to Ngn3 high EP and pre-endocrine cells adapting to secretory demands during development.

Here, we see that the second module actually does seem relevant, as it separates the Ngn3 high EP and pre-endocrine cells that were not separated by the first module.
This might allow the model to isolate Ngn3 high EP cells alone, improving their classification.
The corresponding eigenvalues support this hypothesis, as the relative contribution of the second is greater than that of any other cell type we investigated.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/cell_type/Ngn3_eigvals.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

{% details Module 0 %}
**Overall interpretation**
- Positive side: Notch / endocrine progenitor & cell cycle signaling (*Neurog3*, *Arx*, *Amotl2*, *Cited4*, *Rasd1*) with Rho / chromatid cohesion and Delta‑Notch pathway enrichment—captures actively specifying endocrine precursors before strong secretory specialization.
- Negative side: Broad ER / secretory stress and hormone activity (Protein processing ER, cAMP signaling, secretory/vesicle lumen, mTORC1, TNF‑α via NF‑κB) featuring mature peptide handling (*Sst*, *Pyy*, *Ttr*, *Ghrl*), chaperones (*Hspa5*, *Calr*), and metabolic scaffolds (*Gnas*, *Aldoa*, *Gapdh*).
- Comparison: Early endocrine lineage commitment & proliferative machinery (positive) contrasted with established secretory / ER expansion & stress adaptation (negative); a temporal maturation axis within the scarce Ngn3 high EP population.

**Top 10 genes**
- Positive: *Neurog3*, *8430408G22Rik*, *Amotl2*, *Gcg*, *Tmem171*, *Cited4*, *Arx*, *2010107G23Rik*, *Rasd1*, *Gast*
- Negative: *Sst*, *Rbp4*, *Pyy*, *Hhex*, *Dlk1*, *Cd24a*, *Malat1*, *Eef1a1*, *Isl1*, *Iapp*

**Key GO terms**
- Positive: Delta-Notch Signaling; Regulation of β-Cell Development; Mitotic Spindle / G2-M Checkpoint; Pancreas Beta Cells (early TFs)
- Negative: Protein Processing in ER; Thyroid Hormone Synthesis; cAMP Signaling; Hormone Activity; mTORC1 Signaling
{% enddetails %}

{% details Module 1 %}
**Overall interpretation**
- Positive side: Canonical endocrine hormone processing & secretion program (Pancreas β cell hallmark, MODY pathway, peptide hormone metabolism, ERAD / ER folding) combining insulin & polyhormonal genes (*Ins1/2*, *Pcsk1/2*, *Chga*, *Scgn*, *Iapp*, *Nnat*, *Isl1*, *Pdx1*) with vesicle trafficking (tubulin folding, *Rab27a*, *Exoc7*).
- Negative side: Stress / inflammatory remodeling & junctional / EMT features (TNF‑α, UV response, EMT, oxidative stress, tight junction) with cell cycle regulators (*Ccnd1/3*), redox enzymes (*Gpx3*, *Mgst1*), and structural plasticity (*Vim*, *Rhob*).
- Comparison: Specialized high-fidelity hormone secretory module (positive) versus inflammatory / stress-responsive epithelial remodeling (negative); delineates maturation under stress constraints.

**Top 10 genes**
- Positive: *Ghrl*, *Rbp4*, *Pdx1*, *Nnat*, *Mafb*, *Isl1*, *1700086L19Rik*, *Gng12*, *Ins1*, *Lrpprc*
- Negative: *Gcg*, *Spp1*, *Gpx3*, *Gast*, *Clu*, *Smarca1*, *Arx*, *Serpinh1*, *Ttr*, *Pgrmc1*

**Key GO terms**
- Positive: Pancreas Beta Cells; Maturity Onset Diabetes of the Young; Peptide Hormone Metabolism; Insulin Processing; Regulation of Insulin Secretion
- Negative: TNF‑α Signaling via NF‑κB; UV Response Up; EMT; Hypoxia; Oxidative Stress
{% enddetails %}

{% details Module 2 %}
**Overall interpretation**
- Positive side: Secretory / ER expansion with peptide hormone metabolism and chaperone activation (Protein processing ER, ATF6 chaperones, COPII vesicle, hormone activity) showing polyhormonal output (*Pyy*, *Sst*, *Gcg*, *Ghrl*, *Gast*, *Cck*), UPR chaperones (*Hspa5*, *Calr*, *Hsp90b1*) and vesicle trafficking components.
- Negative side: Junctional adhesion, focal adhesion, tight/actin cytoskeletal architecture and metabolic stress signaling (cell‑substrate junction, focal adhesion, Hypoxia, KRAS, Glycolysis) with adhesion / structural genes (*Actn1*, *Vim*, *Claudins*), and developmental regulators (*Pax4*, *Nkx6-1*, *Pdx1*).
- Comparison: Active hormone secretion + ER folding (positive) vs structural / metabolic remodeling and adhesion (negative); suggests a shift from secretory expansion to tissue integration and metabolic adaptation.

**Top 10 genes**
- Positive: *Ghrl*, *Pyy*, *Gcg*, *Maged2*, *Cdkn1a*, *Arg1*, *Rbp4*, *Sst*, *Cck*, *Gpx3*
- Negative: *Chgb*, *Pdx1*, *Nkx6-1*, *Mafb*, *Fev*, *Ube2e3*, *Cryba2*, *Chga*, *Npepl1*, *Nnat*

**Key GO terms**
- Positive: Protein Processing in ER; Hormone Activity; Peptide Hormone Metabolism; ATF6 Chaperone Activation; COPII Vesicle / ER-Golgi Transport
- Negative: Tight Junction; Focal Adhesion / Cell-Substrate Junction; G2-M Checkpoint; Hypoxia; KRAS Signaling Up
{% enddetails %}

<br>

#### Alpha

Alpha cells are a principal endocrine cell type of the pancreas, defined by high expression of glucagon (*Gcg*) and lineage determinants such as *Arx*.
They function to raise blood glucose via glucagon secretion.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/cell_type/Alpha_modules.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Module 0 separates alpha cells to the right with a positive program consisting of progenitor-leaning markers (*Neurog3*, *Pax4*, *Nkx6-1*, *Insm1*).
On the negative side, markers such as *Sst* and *Hhex* indicate other endocrine lineages.
This appears similar to the top module for the Ngn3 high EP class, suggesting that Ngn3 high EP and alpha cells are inherently quite different.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/cell_type/Alpha_eigvals.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Visualization of the eigenvalues indicated that module 0 contributes the vast majority toward classification.

{% details Module 0 %}
**Overall interpretation**
- Positive side: Early endocrine lineage specification & stress/platelet degranulation signaling interwoven with transcriptional control (*Neurog3*, *Pax4*, *Nkx6-1*, *Insm1*), calcium/vesicle release (*Cd63*, *Pfn1*, *Calm1*), and apoptosis/p53 response (*Cdkn1a*, *Btg2*). Represents a progenitor‑leaning α‑associated endocrine priming state.
- Negative side: Mixed metabolic stress, proliferation, and remodeling (Apoptosis, TNF‑α, AMPK, UV response, Myc targets) with cell cycle and stress genes (*Ccnd1*, *Atf3*, *Gadd45a*, *Mcl1*, *Hes1*) but reduced classic secretory peptides.
- Comparison: Primed endocrine progenitor / specification program (positive) versus adaptive stress / proliferative remodeling (negative) within α lineage context.

**Top 10 genes**
- Positive: *Neurog3*, *Eef1a1*, *Malat1*, *Cd63*, *Tmsb4x*, *Mdk*, *Cdkn1a*, *Btg2*, *Gnas*, *Cyb5r3*
- Negative: *Sst*, *Hhex*, *Ccnd1*, *Iapp*, *Deb1*, *Atp1b1*, *Meis2*, *Psat1*, *Hpgd*, *Cyr61*

**Key GO terms**
- Positive: Regulation of β-Cell Development; Platelet / Secretory Degranulation; Pancreas Beta Cells (specification factors); Apoptosis; Maturity Onset Diabetes of the Young
- Negative: Apoptosis; TNF‑α Signaling via NF‑κB; Unfolded Protein Response; AMPK Signaling; Maturity Onset Diabetes of the Young (late differentiation subset)
{% enddetails %}

{% details Module 1 %}
**Overall interpretation**
- Positive side: Polyhormonal / ghrelin–glucagon secretory module (*Ghrl*, *Gcg*, *Cck*, *Isl1*, *Fev*, *Cryba2*) with axonogenesis / neuronal morphogenesis and regulation of insulin secretion – consistent with neuroendocrine identity features of α cells including shared developmental neuronal programs.
- Negative side: ER/tubulin folding, chaperone & stress-responsive secretory maturation (MODY, UPR, gap junction, phagosome, tubulin folding) marking a structural / biosynthetic reorganization phase with increased protein folding load.
- Comparison: Diverse peptide hormone / neurodevelopmental signaling (positive) contrasted with ER expansion & cytoskeletal maturation (negative); indicates transition toward specialized secretory infrastructure.

**Top 10 genes**
- Positive: *Ghrl*, *Fev*, *Bex2*, *Prrg2*, *Isl1*, *Cck*, *Emb*, *Cryba2*, *Tm4sf4*, *Akr1c19*
- Negative: *Iapp*, *Nnat*, *Pyy*, *Hadh*, *C2cd4b*, *Dlk1*, *Ins2*, *Pcsk2*, *Spp1*, *Pdx1*

**Key GO terms**
- Positive: Pancreas Beta Cells (shared factors); Peptide Hormone Metabolism; Regulation / Positive Regulation of Insulin Secretion; Axonogenesis / Neuron Projection Morphogenesis; Ghrelin Synthesis & Secretion
- Negative: Maturity Onset Diabetes of the Young; Gap Junction; Phagosome; Tubulin Folding Pathway; Unfolded Protein Response
{% enddetails %}

{% details Module 2 %}
**Overall interpretation**
- Positive side: Mature endocrine hormone & vesicle cargo (*Pyy*, *Gcg*, *Iapp*, *Tmem27*, *Cpe*, *Rbp4*) with β/α shared developmental factors (*Neurog3*, *Hmgn3*) and cytoskeletal transport (*Tuba1b*); enriched for mitotic cell cycle / spindle programs indicating proliferative hormone‑expressing intermediates.
- Negative side: Transcriptional and signaling regulation (*Sox4*, *Pax4*, *Hes6*, *Camk2n1*, *Gadd45a*) plus actin remodeling (*Marcksl1*) with lower peptide hormone load—an earlier regulatory / plastic state.
- Comparison: Proliferative polyhormonal secretory expansion (positive) versus pre‑secretory regulatory remodeling (negative); shows coupling of proliferation with broad hormone expression before specialization.

**Top 10 genes**
- Positive: *Pyy*, *Gcg*, *Iapp*, *Tmem27*, *8430408G22Rik*, *Rbp4*, *Neurog3*, *Cpe*, *Hmgn3*, *Tuba1b*
- Negative: *Sox4*, *Mdk*, *Gadd45a*, *Npepl1*, *Pax4*, *Btbd17*, *Ppp1r14b*, *Marcksl1*, *Hes6*, *Camk2n1*

**Key GO terms**
- Positive: Cell Cycle / Mitotic Anaphase & G2/M Transition; Mitotic Spindle Assembly Checkpoint; Secretory / Platelet Degranulation; Pancreas Beta Cells (hormone genes); Maturity Onset Diabetes of the Young
- Negative: p53 Signaling; Glioma / Melanoma (cell cycle stress contexts); Chronic Myeloid Leukemia (proliferation); Cellular Senescence; Protein Folding / ER Lumen
{% enddetails %}

<br>

#### Beta

Finally, beta cells are the insulin‑secreting endocrine cells of the pancreas defined by high expression of insulin genes (*Ins1/2*) and key transcriptional regulators like *Pdx1* and *Nkx6-1*.
They maintain blood glucose homeostasis by secreting insulin in response to elevated glucose.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/cell_type/Beta_modules.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Module 0 appeared to very strongly separate these cells from the rest.
On the positive side, representing beta cells, top genes included canonical beta cell and insulin secretion markers (*Nnat*, *Pdx1*, *Ins1*, *Ins2*).
On the negative side were several general markers for other hormone secretory programs (*Ghrl*, *Neurog3*, *Gcg*, *Isl1*).
This separation between general and insulin-specific secretory programs seems to be the critical factor for classifying beta cells.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/cell_type/Beta_eigvals.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Like most other classes, it appears the first module is essentially the only factor playing a role in classification.

{% details Module 0 %}
**Overall interpretation**
- Positive side: Canonical insulin / β cell lineage program (Pancreas β Cells, MODY, FoxO signaling, insulin secretion, glucose homeostasis) with coordinated metabolic and cell cycle regulators (*Nnat*, *Pdx1*, *Ins1/2*, *Nkx6-1*, *Mafb*, *G6pc2*, *Slc30a8*). Moderate cell cycle & FoxO / AMPK signaling suggests metabolically active differentiating β cells balancing proliferation and functional acquisition.
- Negative side: Polyhormonal / secretory stress & ER remodeling (Secretory granule lumen, peptide hormone metabolism, ER processing, degranulation, chaperones, mTORC1) plus inflammatory / apoptosis signatures indicating a generalized secretory / stress‑buffering endocrine state (mixed lineage) with broad vesicle biogenesis.
- Comparison: Specialized insulin-centric metabolic maturation (positive) versus broad stress-adaptive secretory module (negative); depicts resolution from polyhormonal stress buffering to β functional consolidation.

**Top 10 genes**
- Positive: *Nnat*, *Pdx1*, *Ins2*, *Gng12*, *Mafb*, *Ins1*, *Dlk1*, *Nkx6-1*, *Ociad2*, *Hadh*
- Negative: *Ghrl*, *Cdkn1a*, *Cck*, *Malat1*, *Neurog3*, *Peg3*, *Maged2*, *Eef1a1*, *Gcg*, *Isl1*

**Key GO terms**
- Positive: Pancreas Beta Cells; Maturity Onset Diabetes of the Young; Regulation of Insulin Secretion; FoxO Signaling; Cellular Response to Glucose
- Negative: Secretory Granule Lumen; Peptide Hormone Metabolism; Protein Processing in ER; Platelet Degranulation; mTORC1 Signaling
{% enddetails %}

{% details Module 1 %}
**Overall interpretation**
- Positive side: Lineage transcriptional regulation and nuclear remodeling (PAX factors, FOXA2/A3, β‑cell development, TP53 transcription control) with mRNA turnover / deadenylation and early stress-responsive signals (*Btg2*, *Cnot6l/7*, *Plk2*, *Vim*) – a regulatory / chromatin & transcript processing module shaping β identity.
- Negative side: High-load ER folding / secretory expansion (Protein processing ER, peptide hormone metabolism, chaperonin-mediated folding, tubulin folding, mTORC1) plus vesicle biogenesis & degradation (phagosome, lysosome, neutrophil degranulation) indicating intense biosynthetic throughput for hormone packaging.
- Comparison: Regulatory transcription & post-transcriptional fine-tuning (positive) opposed to bulk secretory factory operation (negative); delineates control vs execution phases of β functional maturation.

**Top 10 genes**
- Positive: *Fev*, *Slc25a5*, *Prrg2*, *Krt8*, *Cryba2*, *Tm4sf4*, *Krt7*, *Chgb*, *Btg2*, *Pax4*
- Negative: *Iapp*, *Pyy*, *Ghrl*, *Rbp4*, *Sst*, *Nnat*, *Hspa5*, *Scgn*, *Arg1*, *Pcsk2*

**Key GO terms**
- Positive: Regulation of β-Cell Development; Maturity Onset Diabetes of the Young; TP53 / Cell Cycle Gene Regulation; Deadenylation of mRNA; EGFR1 Signaling / Intermediate Filament Organization
- Negative: Protein Processing in ER; Peptide Hormone Metabolism; Chaperonin-Mediated Protein Folding; Post-Chaperonin Tubulin Folding; mTORC1 Signaling
{% enddetails %}

{% details Module 2 %}
**Overall interpretation**
- Positive side: Proliferative, hypoxia / EMT and cytoskeletal remodeling program (Hypoxia, Mitotic spindle, E2F targets, EMT, Gap junction) including checkpoint and spindle assembly pathways—marks cycling / stress-responsive β lineage precursors or regenerative states with structural reorganization (GJA1, VIM, SPARC).
- Negative side: Polyhormonal secretory ER & UPR-enriched module (ER processing, peptide hormone metabolism, chaperones, mTORC1, lysosome, antigen presentation) featuring *Sst*, *Chga*, *Pax4*, *Isl1*, *Iapp*—a differentiated secretory state with refined ER folding and vesicle maturation.
- Comparison: Active cell cycle & structural transition (positive) versus mature, high-fidelity secretory endocrine processing (negative); outlines a proliferation to secretory maturation trajectory.

**Top 10 genes**
- Positive: *Neurog3*, *8430408G22Rik*, *Spp1*, *Mt1*, *Mt2*, *Gsta3*, *Bicc1*, *Cited4*, *Vim*, *Ppp1r1b*
- Negative: *Ghrl*, *Sst*, *Cck*, *Rbp4*, *Mdk*, *Gpx3*, *Arg1*, *Pyy*, *Camk2n1*, *Dlk1*

**Key GO terms**
- Positive: Hypoxia; Mitotic Spindle / G2-M Checkpoint; E2F Targets; EMT; Gap Junction
- Negative: Protein Processing in ER; Peptide Hormone Metabolism; mTORC1 Signaling; Lysosome / Phagosome; Chaperonin-Mediated Folding
{% enddetails %}


### Cell type summary

- We trained a decently accurate cell type classifier (~90% validation accuracy; notable overfitting).
- We eigendecomposed $\mathbf{Q}_a$ to identify the gene modules responsible for recognizing cell type $a$.
- Using GO terms analysis, we found that modules separate cell types in a one-versus-all fashion using gene programs that align with conventional biological knowledge.

But cell typing requires cell type labels to train on, which may be biased or inaccurate due to human labeling.
Instead, **it would be nice to learn similar properties that are instead inherent to the data itself** -- such as transcriptional frequencies.


---

## Frequency regression

We could see from the UMAP above that the transcriptional landscape largely consists of a single trajectory, from naive ductal to mature alpha/beta/epsilon/delta cells.
But there are also smaller patterns, such as the difference between these mature cells.
This notion of pattern size could be seen as a sort of "transcriptional scale".
It can be made rigorous as transcriptional "frequencies".
Just as frequencies in music (i.e. time) describe variation on different *time*scales, so do frequencies over our gene expression space describe cellular variation on different *transcriptional* scales.
Crucially, **frequencies can capture the same patterns of transcriptional variation offered by cell types, except they rely purely on the data rather than on human labeling**.

Mathematically, we can define these frequencies using tools from graph signal processing.
More generally, however, this just boils down to eigendecomposition of a transcriptional similarity graph, and it is thus very similar to the intuition shown above.
Briefly, we calculate a $k$ nearest neighbors graph between cells in transcriptional space, resulting in the adjacency matrix

$$ \mathbf{A} \in \mathbb{R}^{n \times n}, $$

where $n$ is the number of cells in the dataset.
This represents our "transcriptional domain" where distance or "scale" is defined by how many hops along edges it'd take to get from one cell to another.
Frequencies are then given by the eigenvectors of the Laplacian

$$ \mathbf{L} = \mathbf{D} - \mathbf{A}, $$

where $\mathbf{D} \in \mathbb{R}^{n \times n}$ is the diagonal degree matrix.
One way to justify relying on $\mathbf{L}$ rather than just $\mathbf{A}$ is that the former is normalized such that the eigenvalues are non-negative.
This is important for intuition, since these eigenvalues represent frequency values associated with each eigenvector/frequency, and negative frequencies wouldn't match our conventional intuitions.
For additional details and examples, see [this post about spatial frequencies over tissues](/blog/2025/graph-fourier).

Note that these frequencies are very similar to diffusion components.
In fact, they are the equivalent up to a couple choices of matrix normalization.
We choose the frequency lens because it feels more fundamental, as it's related to fundamental concepts in signal processing and perhaps more intuitive due to the close relationship to physics.


### Transcriptional frequencies in the developing pancreas

Let's visualize some of the resulting frequencies to see if they capture the desired patterns.
The lowest frequency (which has frequency/eigen value $\lambda=0.272$) stretches across the long developmental axis from naive to mature cells, which is indeed the largest and most visible variation.
On this scale, naive cells have negative values, mature cells have positive values, and developing cells are somewhere in between.
Another way to put it is that this frequency captures the largest transcriptomic differences between cells that tend to be observed often.
It makes sense this large and reliable difference would be the most prominent pattern.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/frequency/frequency_0_umap.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Now let's inspect the second frequency.
Before interpreting biologically, notice that this pattern is visually higher frequency in that the first frequency went from low to high across this same long axis while the second frequency goes from low to high and back to low.
This is similar to comparing a sine wave with one period (high peak to low peak) to a sine wave with 1.5 periods (high peak to low peak to high peak), each shoved into the same interval.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/frequency/frequency_1_umap.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

While this might make sense in terms of general pattern recognition, there is likely a true gene expression signal to back it up since it appears as such a prominent frequency.
One possibility is that it represents a proliferative state unique to actively dividing cells as opposed to more stable ductal or mature cells.

The third frequency represents a smaller fluctuation still, restricted to the upper righthand branches of the transcriptional space.
Comparing to the cell type labels plotted above, this pattern appears to represent the difference between mature β and α/δ/ε cells, which is indeed a subtler pattern than those captured by the first two frequencies.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/frequency/frequency_2_umap.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Finally, we can visualize the fourth frequency.
On first inspection, it looks a bit noisy, with a large but faint fluctuation among naive cells on the left and a strong but small negative valued group branching off of the center.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/frequency/frequency_3_umap.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

So, while there are technically as many frequencies as there are cells (because the Fourier transform is essentially a rotation, in this case from cellular space into a frequency space of equal dimension), **we will only demonstrate results from the first three frequencies**.
While the small negative group in particular may be a real signal, it was not consistent with expert cell typing, and we chose not to demonstrate results from this or any later frequencies.

Altogether, while cell typing attempts to separate cells into *disjoint* groups, frequencies seek to identify increasingly subtle *relationships* between cells.
Thus, our model's performance on a given output dimension will indicate not how well it can isolate one group of cells but instead **how well it can capture a cell's position along a given transcriptional scale** (e.g. naive vs developed or beta vs alpha).


### Frequency model training

Model and training details were the same as for cell type classification, apart from the objective and minor changes to the architecture and optimizer:
- **Data**: `scv.datasets.pancreas()`; remove genes starting with Rpl/Rps; normalize total + log1p (layer `spliced`).
- **Features**: top 10,000 HVGs.
- **Split**: 70% train / 30% val; no test set (exploratory).
- **Architecture**: bilinear layer; two linear projections $\mathbf{W},\mathbf{V}$ (10k→128) whose element‑wise product feeds a linear head $\mathbf{P}$ to $f$ frequency values.
- **Objective**: Huber loss; far better performance than MSE or L1, perhaps because of robustness to outliers often present in biological data.
- **Optimizer**: Adam, lr = 1e-4, weight decay = 1e-4.
- **Schedule**: cosine annealing over 100 epochs.
- **Batch** size: 64.
- **Device**: CPU; seeds fixed (Python, NumPy, Torch) for reproducibility.
- **Biases**: much like the original paper, we observed similar performance with/out bias terms and thus omitted them to make downstream interpretation mathematically simpler.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/frequency/frequency_training.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Overfitting appeared negligible for the exploratory purpose of this work.
Validation MAE appeared quite low but should be compared to the distribution of frequency values trained on.

Here are some random plots related to model performance.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/frequency/frequency_model_analysis.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

To evaluate the performance on each output, we can first break down validation MAEs per frequency.
Normally, we could compare MAEs to standard deviations to get a neat summary.
However, we can plot frequency value histograms and see they are not normally distributed.
But we can still just plot +/- 1 MAE as vertical bars to get a visual sense of model performance.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/frequency/frequency_mae_histograms.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Frequency 0 appears trimodal, with positive, negative, and zero populations, and the MAE seems small enough to avoid misplacing cells between the three modes.
Frequency 1 appears bimodal with a similar level of accuracy.
Finally, frequency 2 is unimodal, and the MAE bounds appear well suited to identify cells toward the positive and negative extremes.


### Frequency model interpretation

Same math as above.

#### Frequency 0

Instead of plotting histograms of module values per category, we can plot module values against frequency values.
After all, this is a regression, and module values are supposed to capture frequency values.
We'll also color points by cell type as an extra biological sanity check.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/frequency/frequency_0_modules.png"
       alt=""
       style="width:120%; display: block; margin: 0 auto;">
</figure>

Module 0 appears to capture this frequency 0 well.
The correlation appears tight, and the progression of cell types matches the developmental trajectory captured by frequency 0, from ductal to specialized endocrine cells.
Later modules loosely capture this progression, but not as well as the first module.
Like in the previous section, we can plot module eigenvalues to evaluate their relative importance, and the first module is indeed the most important.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/frequency/frequency_0_eigvals.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Now we can see which genes define module 0.
On the negative side, genes such as *Sox9* describe ductal cells while others are indicative of progenitors (*Gas6*, *Cd24a*, *Csrp1*, *Col9a3*, *Bicc1*, *Lamb1*).
On the positive side, peptide hormone and secreetory programs describe mature endocrine cell types.
Thus, this module appears to capture the expected scale of biological variation.

Like in the previous section, dropdown tabs containing all module details are included below.

{% details Module 0 %}
**Overall interpretation**
- Positive side: Enriched for classic endocrine secretory and peptide hormone programs (Pancreas β cell set; hormone activity; regulation of insulin secretion; peptide hormone metabolism). Top genes (*Malat1*, *Gnas*, *Pcsk1n*, *Cpe*, *Pfn1*, *Rbp4*, *Iapp*, *Pyy*, *Dynll1*, *Aplp1*) include insulin granule processing (*Cpe*, *Pcsk1n*, *Iapp*), endocrine peptides (*Pyy*), signaling scaffolds (*Gnas*) and cytoskeleton/transport (*Dynll1*, *Pfn1*) consistent with a differentiated hormone‑secreting phenotype. Literature consistently links CPE, PCSK family inhibitors, IAPP, and PYY with mature islet endocrine function and secretory granule biogenesis; the presence of GNAS aligns with cAMP‑regulated insulin secretion. This matches conventional β/enteroendocrine biology.
- Negative side: Enrichment for extracellular matrix / structural and developmental terms (collagen-containing ECM, ER lumen) with genes (*Gas6*, *Cd24a*, *Sox9*, *Csrp1*, *Col9a3*, *Bicc1*, *Lamb1*) indicative of progenitor / ductal / stromal or less differentiated states. *Sox9* and ECM/collagen genes are well known markers of pancreatic ductal / progenitor compartments, contrasting the secretory endocrine program. This inverse relationship (hormone secretion vs progenitor/ECM) is biologically expected during endocrine maturation.
- Comparison: Captures a differentiation axis contrasting mature secretory endocrine identity against progenitor/ECM remodeling; positive and negative sides appear as endpoint states along endocrine maturation.

**Top 10 genes**
- Positive: *Malat1*, *Gnas*, *Pcsk1n*, *Cpe*, *Pfn1*, *Rbp4*, *Iapp*, *Pyy*, *Dynll1*, *Aplp1*
- Negative: *Gas6*, *Cd24a*, *Sox9*, *Csrp1*, *Col9a3*, *Bicc1*, *Rbp2*, *Lamb1*, *Maob*, *Pi4k2b*

**Key GO terms**
- Positive: Pancreas β Cells; Hormone Activity; Regulation of Insulin Secretion; Peptide Hormone Metabolism; Secretory Granule Degranulation; Neuropeptide Hormone Activity
- Negative: Collagen‑Containing Extracellular Matrix; Urogenital System Development
{% enddetails %}


{% details Module 1 %}
**Overall interpretation**
- Positive side: Dominated by proliferative / cell cycle and mitotic control signals (strong enrichment for E2F Targets, G2/M Checkpoint, multiple mitotic Reactome pathways) with genes such as *Birc5*, *Dynll1*, *Jun*, *Myl6*, *Marcksl1*, *Tmsb10* indicating cytoskeletal remodeling coupled to division. *Birc5* (Survivin), E2F programs, and G2/M checkpoint signatures are canonical markers of proliferating endocrine progenitors or replicating β cells described in islet regeneration and development literature.
- Negative side: Enriched for mature β cell secretory and ER protein processing pathways (Pancreas β Cells, Protein Processing in ER, Insulin Secretion, Unfolded Protein Response) driven by *Ins1*, *Ins2*, *Iapp*, *Chga*, *Rbp4*, *Sec61b*. This reflects the expected trade‑off where high proliferation correlates with reduced specialized secretory gene expression, consistent with documented inverse coupling of β cell differentiation and cell cycle entry.
- Comparison: Defines a proliferation vs functional specialization trade‑off; the module partitions dividing progenitor-like cells from quiescent, secretion-optimized beta cells.

**Top 10 genes**
- Positive: *Tmsb10*, *Marcksl1*, *Dynll1*, *2810417H13Rik*, *Myl6*, *Jun*, *Malat1*, *Mpzl1*, *Birc5*, *Pnliprp1*
- Negative: *Eef1a1*, *Ins2*, *Ins1*, *Iapp*, *Rbp4*, *Dlk1*, *Sec61b*, *Chga*, *Pfn1*, *Hmgn3*

**Key GO terms**
- Positive: E2F Targets; G2-M Checkpoint; Cell Cycle, Mitotic; Mitotic Anaphase/Metaphase; Resolution of Sister Chromatid Cohesion
- Negative: Pancreas β Cells; Protein Processing in Endoplasmic Reticulum; Unfolded Protein Response; Insulin Secretion; Maturity Onset Diabetes of the Young; mTORC1 Signaling
{% enddetails %}


{% details Module 2 %}
**Overall interpretation**
- Positive side: Mixture of multi‑lineage endocrine hormone genes (*Ghrl*, *Gcg*, *Cck*, *Ttr*, *Isl1*) and translation/processing chaperones (*Hsp90aa1*, *Eef1a1*, *Ssr2*) plus cytoskeletal / microtubule organization (*Tmsb4x*) and peptide hormone metabolism / secretory granule lumen GO terms. This suggests a module capturing generalized secretory vesicle biogenesis and hormone diversity (α/ε/PP/β lineage signatures) rather than a single lineage, aligning with known co‑expression of ghrelin/glucagon/CCK in immature or transitional pancreatic endocrine states.
- Negative side: Enrichment for core insulin secretory and metabolic stress pathways (Pancreas β Cells, Regulation of Insulin Secretion, Maturity Onset Diabetes of the Young, mTORC1 Signaling, Hypoxia, Glycolysis) with classic insulin granule genes (*Ins1*, *Ins2*, *Nnat*) and metabolic regulators (*Calm1*, *Pdia6*, *Tsc22d1*). This inverse pattern (broad hormone / cytoskeletal remodeling vs specialized insulin metabolic program) is consistent with literature describing diversification and metabolic reprogramming during β cell functional maturation and stress adaptation.
- Comparison: Represents a continuum from broad polyhormonal/structural remodeling to insulin-focused metabolic specialization—an expected maturation narrowing of lineage output.

**Top 10 genes**
- Positive: *Ghrl*, *Tmsb4x*, *Gcg*, *Eef1a1*, *Rbp4*, *Hsp90aa1*, *Isl1*, *Ssr2*, *Ttr*, *Cck*
- Negative: *Ins2*, *Ins1*, *Malat1*, *Spp1*, *Tmsb10*, *Nnat*, *Calm1*, *Pdia6*, *Tsc22d1*, *Sh3bgrl3*

**Key GO terms**
- Positive: Peptide Hormone Metabolism; Secretory Granule Lumen; Hormone Activity; Tubulin Folding Pathway
- Negative: Pancreas β Cells; Regulation of Insulin Secretion; mTORC1 Signaling; Glycolysis
{% enddetails %}


<br>

#### Frequency 1

Looks great, nice job.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/frequency/frequency_1_modules.png"
       alt=""
       style="width:120%; display: block; margin: 0 auto;">
</figure>

Module 1 looks great.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/frequency/frequency_1_eigvals.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Also looks like it's the most important one.

Here are the key markers and GO terms.

Here are all markers and GO terms for all modules in case you're interested.

{% details Module 0 %}
**Overall interpretation**
- Positive side: Strong ER / protein folding and adaptive stress signature (Protein Processing in ER, Unfolded Protein Response, ATF6 chaperone program) combined with cytoskeletal / junctional remodeling (Tight Junction, RHO GTPase–ROCK signaling) and early endocrine specification (Pancreas β Cells set including *Neurog3*, *Pax4*, *Mafb*). Genes (*Hspa5*, *Pdia6*, *Ssr2*, *Tuba1a*, *Cdk4*, *Jun*) reflect a transitional, stress‑acclimating endocrine population ramping biosynthetic load—consistent with literature linking UPR and cytoskeletal remodeling to insulin granule biogenesis and early β lineage commitment.
- Negative side: Enrichment for protease / endopeptidase and ubiquitin ligase binding plus hormone activity (Gcg, Npy, Ghrl/Chgb represented) suggests a comparatively secretion‑biased, proteostasis / degradative vesicle state with reduced acute UPR activation. This inverse UPR vs proteolytic/hormone emphasis aligns with staged ER expansion: cells entering high secretory capacity initially activate chaperones, while more homeostatic secretory cells show balanced proteostasis.
- Comparison: Contrasts early ER stress adaptation / biosynthetic ramp-up with a more stabilized secretory proteostasis state, marking temporal phases of secretory capacity establishment.

**Top 10 genes**
- Positive: Hspa5, Tuba1a, Epcam, Jun, Ssr2, Cd81, Tubb5, Cdk4, Ier2, Pdia6
- Negative: Eef1a1, Arf5, Hsp90aa1, Ngfrap1, Gabarapl2, Eif4a2, Clps, Slc25a5, Gcg, Krt18

**Key GO terms**
- Positive: Protein Processing in Endoplasmic Reticulum; Unfolded Protein Response; Pancreas β Cells; RHO GTPases Activate ROCKs; Tight Junction
- Negative: Endopeptidase Activity; Ubiquitin Protein Ligase Binding; Hormone Activity
{% enddetails %}


{% details Module 1 %}
**Overall interpretation**
- Positive side: Barrier / junctional architecture and immune / stress adaptation (Tight Junction, Leukocyte Transendothelial Migration, Innate Immune System, Neutrophil Degranulation), coupled with metabolic / growth signaling (mTORC1, Myc Targets) and modest endocrine identity (Pancreas β Cells). Suggests a remodeling epithelial endocrine subset balancing secretory stress (UPR) with cytoskeletal tension (ROCK/actomyosin) and immune‑associated chaperone activity; parallels reports of β cell states under inflammatory or metabolic pressure showing junctional and Myc/mTOR adjustments.
- Negative side: Secretory granule lumen, lysosomal / ER lumen and antigen presentation enrichment indicate a state specialized for vesicle maturation, proteolysis and antigen processing (consistent with professional secretory stress buffering). Reduced junction/immune signaling vs enhanced vesicle turnover reflects a functional dichotomy between structural remodeling and dedicated secretory organelle refinement.
- Comparison: Highlights a remodeling/inflammatory junctional program opposing a refined vesicle maturation program; suggests branching into structural adaptation vs secretion-focused specialization.

**Top 10 genes**
- Positive: Cdk4, Ier2, Tuba1b, Tuba1a, Bex2, Mt1, Trappc2l, Hadh, Hspa5, Ambp
- Negative: Arf5, Gapdh, Hmgn3, Ctsb, Krt18, Hmgb1, Npc2, Eef1a1, Nt5dc2, App

**Key GO terms**
- Positive: Tight Junction; Neutrophil Degranulation; Leukocyte Transendothelial Migration; Innate Immune System; mTORC1 Signaling
- Negative: Secretory Granule Lumen; Lysosomal Lumen; Protein Processing in ER; Antigen Processing and Presentation
{% enddetails %}


{% details Module 2 %}
**Overall interpretation**
- Positive side: ER / vesicle biogenesis plus inflammatory and metabolic stress (Protein Processing in ER; Phagosome; mTORC1 Signaling; Complement; TNF‑α via NF‑κB; Hypoxia; Protein Secretion) with chaperones (Hspa5), metabolic enzymes (PHGDH, ALDOA, GAPDH) and secretory regulators (PCSK1N, CPE). Mirrors literature describing an adaptive secretory stress / inflammatory beta cell or mixed endocrine stress state preceding proliferative exit.
- Negative side: Classical proliferative cell cycle program (E2F Targets, G2-M Checkpoint, DNA topological change) indicating cells in division with relative down‑weighting of stress/secretory remodeling. This reciprocal relationship between cell cycle activation and secretory stress modules is a known axis in endocrine maturation where proliferation declines as specialized secretory function intensifies.
- Comparison: Encodes a stress-adaptive secretory vs proliferative cycling antagonism; cells trade off between coping with secretory/inflammatory load and cell division.

**Top 10 genes**
- Positive: Calm1, Cpe, Pcsk1n, Jun, Hmgn3, Pfn1, Gars, Uqcc2, Gch1, Chgb
- Negative: Tmsb4x, Clps, Spint2, Tmsb10, H2afv, Cdk4, Aplp1, Hsp90aa1, Jund, Peg3

**Key GO terms**
- Positive: Protein Processing in Endoplasmic Reticulum; Phagosome; mTORC1 Signaling; TNF‑alpha Signaling via NF‑κB; Hypoxia
- Negative: E2F Targets; G2-M Checkpoint; Cell Cycle, Mitotic; DNA Topological Change
{% enddetails %}


<br>

#### Frequency 2

Looks great, nice job.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/frequency/frequency_2_modules.png"
       alt=""
       style="width:120%; display: block; margin: 0 auto;">
</figure>

Module 2 looks great.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/frequency/frequency_2_eigvals.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Also looks like it's the most important one.

Here are the key markers and GO terms.

Here are all markers and GO terms for all modules in case you're interested.

{% details Module 0 %}
**Overall interpretation**
- Positive side: Broad multi‑hormone endocrine secretory program (*Ghrl*, *Gcg*, *Iapp*, *Ttr*, *Isl1*) with robust ER folding / antigen presentation (Protein Processing in ER, Peptide Hormone Metabolism, Secretory Granule Lumen, Antigen Processing) and cytoskeletal/tubulin maturation. Reflects a polyhormonal / transitional endocrine state (reported during developmental and regenerative phases) engaging chaperone and metabolic pathways (mTORC1) to expand peptide output.
- Negative side: Enrichment for insulin lineage maturation and glucose responsiveness (MODY pathway, Regulation of Insulin Secretion, Type B pancreatic cell differentiation, Insulin Secretion, *G6pc2*, *Pdx1*, *Adra2a*, *Gip*) marking a more specialized β program suppressing broader polyhormonal identity—consistent with literature where polyhormonal signatures resolve into insulin‑focused expression as cells mature.
- Comparison: Delineates polyhormonal transitional cells vs insulin-specialized mature beta cells; suggests progressive lineage resolution.

**Top 10 genes**
- Positive: *Ghrl*, *Gcg*, *Rbp4*, *Isl1*, *Pfn1*, *Ssr2*, *Malat1*, *Iapp*, *Ngfrap1*, *Ttr*
- Negative: *Ins2*, *Ins1*, *Npy*, *Adra2a*, *Nnat*, *Gip*, *Sytl4*, *Spock2*, *Scaper*, *Mapt*

**Key GO terms**
- Positive: Protein Processing in Endoplasmic Reticulum; Peptide Hormone Metabolism; Secretory Granule Lumen; mTORC1 Signaling; Antigen Processing and Presentation
- Negative: Maturity Onset Diabetes of the Young; Regulation of Insulin Secretion; Insulin Secretion; Type B Pancreatic Cell Differentiation
{% enddetails %}


{% details Module 1 %}
**Overall interpretation**
- Positive side: Hormone activity and polyhormonal secretory signaling (PYY, GAST, CCK, GCG, GHRL) with vesicle trafficking (CD63, TMED10) and generic secretory machinery, indicating an endocrine secretagogue module emphasizing diverse peptide release and exocytosis.
- Negative side: Metabolic stress and beta maturation (mTORC1 Signaling, MODY, oxidative phosphorylation, glycolysis) plus developmental ossification / SOX9 associated pathways and cell cycle checkpoint—suggesting cells shifting metabolic reprogramming and biosynthetic investment toward refined insulin secretory competence while dialing down broad hormone cocktail output.
- Comparison: Opposes broad peptide secretion diversity with metabolically intensive beta maturation, indicating a functional convergence toward insulin-centric efficiency.

**Top 10 genes**
- Positive: *Tmsb4x*, *Clps*, *Rbp4*, *Ghrl*, *Aplp1*, *Ttr*, *Gcg*, *Hsp90aa1*, *Eef1a1*, *Cck*
- Negative: *Calm1*, *Spp1*, *Nnat*, *Malat1*, *Jun*, *Tsc22d1*, *Gars*, *Pdia6*, *Pfn1*, *Gapdh*

**Key GO terms**
- Positive: Hormone Activity; Protein Secretion; Secretory Granule Lumen
- Negative: mTORC1 Signaling; Maturity Onset Diabetes of the Young; Oxidative Phosphorylation; Glycolysis
{% enddetails %}


{% details Module 2 %}
**Overall interpretation**
- Positive side: High proliferation / cell cycle and cytoskeletal remodeling (E2F Targets, G2-M Checkpoint, multiple mitotic Reactome pathways, Microtubule Cytoskeleton) with ubiquitin ligase binding—representing a dividing endocrine progenitor / expansion state where secretory specialization is secondary.
- Negative side: Secretory / developmental endocrine profile (Pancreas β Cells, Phagosome / vesicle pathways, Gap Junction, Platelet/Secretory Degranulation, Innate Immune System) including differentiation regulators (*Neurog3*, *Pyy*, *Cpe*) consistent with exit from cell cycle toward functional endocrine maturation. Literature supports inverse coupling of *Neurog3*‑driven differentiation and proliferative cycling.
- Comparison: Captures proliferative progenitor expansion versus differentiation toward secretory endocrine identity; reflects maturation exit from the cell cycle.

**Top 10 genes**
- Positive: *Spp1*, *Myl12a*, *Rbp4*, *Dynll1*, *Cpa1*, *Hsp90aa1*, *Hmgb2*, *Tmsb10*, *Clu*, *Mif*
- Negative: *Pyy*, *Cpe*, *Ypel3*, *Neurog3*, *Ssr2*, *Hmgn3*, *Tuba1a*, *Psmc2*, *Jun*, *Xist*

**Key GO terms**
- Positive: E2F Targets; G2-M Checkpoint; Cell Cycle, Mitotic; Microtubule Cytoskeleton; Mitotic Spindle Assembly Checkpoint
- Negative: Pancreas β Cells; Phagosome; Gap Junction; Platelet/Secretory Degranulation; Antigen Processing & Presentation
{% enddetails %}

<br>

### Frequency summary

- Frequencies represent transcriptional scales, analogous to diffusion components.
- Bilinear MLPs can learn to map from a given cell's gene expression profile to its coordinate in this frequency space.
- Provides explcit markers for each frequency, which are otherwise typically calculated via correlations of single genes with each frequency.
- Efficient label transfer to other datasets, which is otherwise prohibitive and inductive due to eigendecomposition (classic GNN -> MLP logic).


---

## Perturbation regression

Could also do this with perturbations to identify corresponding gene modules.
This will also take advantage of the ability to choose combinations of output directions.
In principle, we could use this approach to predict the effect of combined perturbations.

### Perturbation dataset

Low-counts cells formed a mix perturbation clusters, so we filtered using a lower bound threshold of 5k counts per cell.
This greatly reduced the number of cells in the dataset from ~25k to ~10k, but it yielded a much cleaner dataset, a far more accurate model, and more reliable interpretations.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/perturbation_umap.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Perturbations are separable considering high doses.
All but BMS have large separable, high-dose subpopulations, so we expect BMS to be harder to fit.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/dose_umap.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Also notice that only low-dose BMS perturbations appear separable.
We'll see that the model's performance matches our ability to visually separate perturbation classes.
In particular, the patterns of separation based on dose will be reflected in the modules we identify.


### Perturbation model training

Used regression to account for varying perturbation strengths.
Dose-scaled one-hot encodings of perturbation type.
Log transformed dose values to stabilize variance.

Model and training details were mostly the same as for frequency regression:
- **Data**: `scv.datasets.pancreas()`; remove genes starting with Rpl/Rps; normalize total + log1p (layer `spliced`).
- **Features**: top 5,000 HVGs; very strong overfitting using 10,000 HVGs.
- **Split**: 70% train / 30% val; no test set (exploratory).
- **Architecture**: bilinear layer; two linear projections $\mathbf{W},\mathbf{V}$ (10k→128) whose element‑wise product feeds a linear head $\mathbf{P}$ to $p$ perturbation values. Dropout rate = 0.4.
- **Objective**: Huber loss; far better performance than MSE or L1, perhaps because of robustness to outliers often present in biological data.
- **Optimizer**: Adam, lr = 1e-4, weight decay = 1e-3.
- **Schedule**: cosine annealing over 100 epochs.
- **Batch** size: 128.
- **Device**: CPU; seeds fixed (Python, NumPy, Torch) for reproducibility.
- **Biases**: much like the original paper, we observed similar performance with/out bias terms and thus omitted them to make downstream interpretation mathematically simpler.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/perturbation_training.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/perturbation_model_analysis.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Are these errors related to dose?


### Perturbation model interpretation

<br>

#### BMS

Looks great, nice job.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/BMS_modules_by_class.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/BMS_modules_by_strength.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

We show all other perturbation types as a baseline.
They all have zero BMS strength, so they show up in these plots at the zero strength position.
While they tend to bunch up, they provide a useful visual baseline.

Looks like only the low-dose cells are separable.
Is that a biological phenomenon?

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/BMS_eigvals.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>


Also looks like it's the most important one.

Here are the key markers and GO terms.

Here are all markers and GO terms for all modules in case you're interested.

{% details Module 0 %}
**Overall interpretation**
 - Positive side: Coagulation / acute inflammatory and nuclear receptor response (fibrin clot factors *FGA*, *FGB*, *F5*; Nuclear Receptors Meta-Pathway; VEGFA–VEGFR2; TNF-alpha / EMT) indicating a pro-thrombotic, stress / remodeling program.
 - Negative side: No coherent opposing enrichment (no significant GO terms), suggesting the weight mass is concentrated on the pro-coagulation/inflammatory direction.
 - Comparison: Module captures a unipolar acute phase / coagulation plus inflammatory NF-κB & EMT axis rather than a balanced contrast.

**Top 10 genes**
- Positive: *DKK1*, *TP53I3*, *FAM129A*, *SREBF1*, *SEMA3A*, *ASTN2*, *Z83843.1*, *HGD*, *FKBP5*, *FGA*
- Negative: *AC027288.3*, *PHLDA2*, *OPCML*, *ARHGAP15*, *SERTAD2*, *AC011287.1*, *FRMD4A*, *CATSPERB*, *AC005614.2*, *SCUBE3*

**Key GO terms**
- Positive: Blood Clotting Cascade; Common / Formation of Fibrin Clot; Nuclear Receptors Meta-Pathway; VEGFA–VEGFR2 Signaling; TNF-alpha Signaling via NF-kB
- Negative: None significant (p<0.05)
{% enddetails %}


{% details Module 1 %}
**Overall interpretation**
 - Positive side: Strong cell cycle / mitotic spindle and kinetochore assembly program (G2-M Checkpoint, E2F Targets, multiple spindle Reactome terms) indicating proliferating cells.
 - Negative side: p53 / apoptosis / DNA damage and androgen receptor signaling enrichments suggest stress / damage response and survival signaling opposite proliferation.
 - Comparison: Axis contrasts active proliferation versus DNA damage–associated apoptotic and receptor-mediated stress signaling.

**Top 10 genes**
- Positive: *SCUBE3*, *SH3BP2*, *AC011287.1*, *HSPA1A*, *TUBA1B*, *MKI67*, *AC016205.1*, *PDE4B*, *MACROD2*, *PDE7B*
- Negative: *CDKN1A*, *CLU*, *HIF1A-AS2*, *DKK1*, *MDM2*, *TRAM1*, *MT-TL2*, *SULF2*, *IGFL2-AS1*, *SYBU*

**Key GO terms**
- Positive: G2-M Checkpoint; Kinetochore Assembly; Sister Chromatid Segregation; E2F Targets; Mitotic Spindle Checkpoint
- Negative: Apoptosis; DNA Damage Response; Androgen Receptor Signaling; TP53 Network; miRNA Regulation of DNA Damage Response
{% enddetails %}


{% details Module 2 %}
**Overall interpretation**
 - Positive side: p53 / DNA damage, androgen receptor and hypoxia signaling (DNA Damage Response, TP53 Network, Androgen receptor, Hypoxia) indicating stress-triggered checkpoint activation.
 - Negative side: Secretory / platelet granule and estrogen response terms (Platelet Alpha Granule Lumen, Estrogen Response Early, Cholesterol Homeostasis) suggest a differentiated secretory / metabolic state.
 - Comparison: Opposes stress-induced genome surveillance and androgen/hypoxia responses against a platelet/secretory metabolic program.

**Top 10 genes**
- Positive: *MDM2*, *SCUBE3*, *SSH2*, *FRMD5*, *CDKN1A*, *SH3BP2*, *TUBA1B*, *CERS6*, *HIF1A-AS2*, *PTPRT*
- Negative: *FGG*, *CLU*, *LINGO2*, *DKK1*, *KAZN*, *IGSF11*, *AC011287.1*, *FHIT*, *ERRFI1*, *AC025627.1*

**Key GO terms**
- Positive: DNA Damage Response; miRNA Regulation of DNA Damage Response; TP53 Network; Androgen Receptor Signaling; Hypoxia
- Negative: Adrenergic Receptor Binding; Platelet Alpha Granule Lumen; Estrogen Response Early; Cholesterol Homeostasis; Plasminogen Activation
{% enddetails %}

<br>

#### Dex

Looks great, nice job.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/Dex_modules_by_class.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

Looks like zero-strength module values are higher than baseline in Dex.
This is weird but may be due to...

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/Dex_modules_by_strength.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/Dex_eigvals.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>


Also looks like it's the most important one.

Here are the key markers and GO terms.

Here are all markers and GO terms for all modules in case you're interested.

{% details Module 0 %}
**Overall interpretation**
 - Positive side: Hypoxia / nuclear receptor metabolic adaptation plus keratin & intermediate filament cytoskeletal remodeling and angiogenic/vasculature regulation, indicating stress-driven epithelial reprogramming.
 - Negative side: No significant opposing enrichment, implying a focused directional module capturing Dex-induced hypoxia–glucocorticoid response.
 - Comparison: Unipolar activation of glucocorticoid-responsive metabolic and structural stress pathways.

**Top 10 genes**
- Positive: *COBLL1*, *ITGB4*, *MT2A*, *CIDEC*, *SDK2*, *FGD4*, *FKBP5*, *TFCP2L1*, *ANGPTL4*, *CHST9*
- Negative: *C5orf66*, *LINC00824*, *PHLDA2*, *PAPLN*, *CCDC175*, *PTPRG-AS1*, *MT-ATP6*, *FAM196A*, *KCNT2*, *TCF4*

**Key GO terms**
- Positive: Hypoxia; Nuclear Receptors Meta-Pathway; Estrogen Response Early; Keratin Filament; Apoptosis
- Negative: None significant (p<0.05)
{% enddetails %}


{% details Module 1 %}
**Overall interpretation**
 - Positive side: Secretory / acute phase and coagulation-associated program (Secretory Granule Lumen, Platelet/Plasminogen pathways, Estrogen Response) indicating enhanced secretory and inflammatory coagulation milieu.
 - Negative side: Hypoxia, cell cycle checkpoint (G2-M), and mTORC1 signaling reflect metabolic and proliferative stress processes suppressed on positive side.
 - Comparison: Axis contrasts secretory / coagulation enrichment with hypoxia–proliferative stress and growth signaling.

**Top 10 genes**
- Positive: *C5orf66*, *SLLI*, *SERPINA3*, *MT-ATP6*, *GPX2*, *HSD11B2*, *FGG*, *S100P*, *MT-ND3*, *C3*
- Negative: *COBLL1*, *TP53I3*, *IGFL2-AS1*, *PLD5*, *AKAP12*, *EPN2*, *NEU1*, *ASTN2*, *LYST*, *TNFSF15*

**Key GO terms**
- Positive: Secretory Granule Lumen; Estrogen Response Early; Estrogen Response Late; Positive Regulation of Cell-Substrate Adhesion; Platelet Alpha Granule Lumen
- Negative: Estrogen Response Late; Hypoxia; G2-M Checkpoint; Estrogen Response Early; mTORC1 Signaling
{% enddetails %}


{% details Module 2 %}
**Overall interpretation**
 - Positive side: Apoptosis and mTORC1 signaling with DNA damage / spindle checkpoint terms indicating stress-induced proliferative control and metabolic regulation.
 - Negative side: Estrogen Receptor Pathway suggests suppressed hormone receptor signaling opposite apoptotic/mTOR axis.
 - Comparison: Opposes stress-apoptotic checkpoint activation versus estrogen receptor–mediated signaling.

**Top 10 genes**
- Positive: *FDXR*, *MDM2*, *CDKN1A*, *MORC3*, *FSTL4*, *DIAPH3*, *GALNT18*, *KRT18*, *DHRS2*, *LINC01021*
- Negative: *LUCAT1*, *PLCXD3*, *NCKAP5*, *TP63*, *ACLY*, *DPYD*, *PDK4*, *LMCD1-AS1*, *SSH2*, *ITFG1*

**Key GO terms**
- Positive: Apoptosis; mTORC1 Signaling; Gastric Cancer Network 1; Spindle Assembly Checkpoint; Estrogen Response Early
- Negative: Estrogen Receptor Pathway
{% enddetails %}

<br>

#### Nutlin

Looks great, nice job.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/Nutlin_modules_by_class.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/Nutlin_modules_by_strength.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/Nutlin_eigvals.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>


Also looks like it's the most important one.

Here are the key markers and GO terms.

Here are all markers and GO terms for all modules in case you're interested.

{% details Module 0 %}
**Overall interpretation**
 - Positive side: Canonical p53 activation (p53 signaling, DNA damage, TP53 network, mTORC1) reflecting Nutlin-induced stabilization of p53 and downstream stress responses.
 - Negative side: Ion homeostasis and calcium handling / cardiac conduction processes indicating reduced excitable / ion transport state opposite p53 stress program.
 - Comparison: Contrasts p53-driven apoptotic and metabolic checkpoint activation with diminished calcium/ion signaling pathways.

**Top 10 genes**
- Positive: *MDM2*, *CDKN1A*, *FDXR*, *TP53I3*, *FAM129A*, *ALDH3A1*, *PLD5*, *LINC01021*, *TRANK1*, *AC025627.1*
- Negative: *USP37*, *ENOX1*, *NR0B1*, *SCUBE3*, *DPYD*, *AC016205.1*, *BX470209.1*, *PLCXD3*, *CKB*, *ROBO2*

**Key GO terms**
- Positive: p53 Signaling Pathway; DNA Damage Response; Genotoxicity Pathway; TP53 Network; mTORC1 Signaling
- Negative: Cell Communication by Electrical Coupling; Calcium Ion Transport Into Cytosol; Ion Homeostasis; Cellular Response to Caffeine; Cardiac Muscle Contraction
{% enddetails %}


{% details Module 1 %}
**Overall interpretation**
 - Positive side: cAMP / phosphodiesterase signaling plus nuclear receptor / detox (AHR, NRF2) indicating transcriptional rewiring of signaling and redox homeostasis under p53 perturbation.
 - Negative side: Angiogenesis, keratin/cytoskeletal remodeling, hypoxia and apoptosis signatures suppressed relative to signaling / metabolic reprogramming.
 - Comparison: Opposes PDE/nuclear receptor mediated signaling and xenobiotic / oxidative stress management against structural stress / angiogenic remodeling.

**Top 10 genes**
- Positive: *PDE4B*, *LINC00511*, *ALDH3A1*, *ST8SIA4*, *IGSF1*, *CSGALNACT1*, *FHIT*, *CRY1*, *MT-RNR2*, *FTL*
- Negative: *FKBP5*, *KRT18*, *DOCK4*, *C3*, *HGD*, *CIDEC*, *FGD4*, *BIRC3*, *MDM2*, *ERRFI1*

**Key GO terms**
- Positive: 3',5'-cAMP Phosphodiesterase Activity; Nuclear Receptors Meta-Pathway; Phosphodiesterases in Neuronal Function; Aryl Hydrocarbon Receptor Pathway; NRF2 Pathway
- Negative: Positive Regulation of Vasculature Development; Positive Regulation of Angiogenesis; Keratin Filament; Estrogen Response Early; Hypoxia
{% enddetails %}


{% details Module 2 %}
**Overall interpretation**
 - Positive side: p53 pathway with binding/interaction terms (Adrenergic Receptor Binding, PDZ Domain Binding, p53 Pathway) suggesting scaffold-mediated p53 effector integration.
 - Negative side: Amino acid (leucine / branched-chain) transport and negative regulation of motility—nutrient sensing and migratory restraint opposite p53 signaling emphasis.
 - Comparison: Encodes p53-centric signaling vs nutrient transport / motility suppression.

**Top 10 genes**
- Positive: *MDM2*, *TP53I3*, *DLG2*, *KRT81*, *ALDH3A1*, *HSPA1A*, *SLC22A4*, *FBN2*, *P3H2*, *IGFBP7*
- Negative: *CA12*, *MCTP1*, *AC124319.2*, *P4HA3*, *AC010197.1*, *RHOB*, *SVEP1*, *DHRS2*, *FGD4*, *AC138819.1*

**Key GO terms**
- Positive: Adrenergic Receptor Binding; PDZ Domain Binding; p53 Pathway; Disordered Domain Specific Binding
- Negative: Leucine Transport; Branched-Chain Amino Acid Transport; Negative Regulation of Cell Motility
{% enddetails %}

<br>

#### SAHA

Looks great, nice job.

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/SAHA_modules_by_class.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/SAHA_modules_by_strength.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>

<figure style="text-align: center;">
  <img src="/assets/figures/scbmlp/perturbation/SAHA_eigvals.png"
       alt=""
       style="width:100%; display: block; margin: 0 auto;">
</figure>


Also looks like it's the most important one.

Here are the key markers and GO terms.

Here are all markers and GO terms for all modules in case you're interested.

{% details Module 0 %}
**Overall interpretation**
 - Positive side: Wnt antagonist / differentiation modulation (DKK1) with selective cardiac differentiation regulation terms, indicating targeted developmental signaling modulation under HDAC inhibition.
 - Negative side: Hypoxia / Myogenesis / Estrogen Early enrichment on the opposite weights reflects broad stress, metabolic and differentiation programs suppressed relative to DKK1/lineage-focused signaling.
 - Comparison: Axis contrasts focused Wnt/cardiac differentiation regulation against generalized hypoxic / myogenic / estrogenic stress programs.

**Top 10 genes**
- Positive: *FTL*, *AC025627.1*, *DKK1*, *AC078923.1*, *LINC02045*, *CRYBG2*, *RNF103-CHMP3*, *DNAH12*, *AC139493.2*, *LINC01885*
- Negative: *ALDOC*, *STC1*, *SMPD3*, *AC008050.1*, *RHOB*, *CKB*, *NR4A1*, *ATP1A3*, *PLA2G4C*, *ZNF815P*

**Key GO terms**
- Positive: Negative Regulation of Cardiac Muscle Cell Differentiation; Negative Regulation of Cardiocyte Differentiation; Regulation of Cardiac Muscle Cell Differentiation
- Negative: Hypoxia; Estrogen Response Early; Myogenesis; TNF-alpha Signaling via NF-kB; Estrogen Response Late
{% enddetails %}


{% details Module 1 %}
**Overall interpretation**
 - Positive side: EMT and cytoskeletal remodeling (EMT, Estrogen Response Early) reflecting HDAC inhibition–driven plasticity and epithelial transition.
 - Negative side: Hypoxia, metabolic (mTORC1), KRAS signaling and amino acid transport denote suppressed stress/metabolic adaptation on the opposite side.
 - Comparison: Balances chromatin-relaxed epithelial plasticity vs hypoxia/mTOR metabolic stress programs.

**Top 10 genes**
- Positive: *TFPI2*, *DKK1*, *PEG10*, *KRT19*, *GNGT1*, *SLC8A1*, *DNAH12*, *PDE5A*, *FAM155A*, *MIR325HG*
- Negative: *ALDH3A1*, *KRT7*, *TFPI*, *SERPINE1*, *MDM2*, *TP53I3*, *FKBP5*, *PCSK6*, *HGD*, *NXF1*

**Key GO terms**
- Positive: Epithelial Mesenchymal Transition; Estrogen Response Early; Regulation of Tau-Protein Kinase Activity; Cellular Response to Purine-Containing Compound
- Negative: Hypoxia; Nuclear Receptors Meta-Pathway; mTORC1 Signaling; KRAS Signaling Up; Estrogen Response Late
{% enddetails %}


{% details Module 2 %}
**Overall interpretation**
 - Positive side: Cholesterol / sterol biosynthesis (multi-pathway SREBP & HMGCR axis) and androgen response indicating metabolic reprogramming under HDAC inhibition.
 - Negative side: Coagulation / platelet granule and inflammatory TNF-alpha / signal transduction pathways reflect suppressed hemostatic-inflammatory signaling relative to lipid anabolism.
 - Comparison: Contrasts lipid / sterol anabolic remodeling with inflammatory coagulation and signal transduction networks.

**Top 10 genes**
- Positive: *COL26A1*, *INSIG1*, *FDFT1*, *CEACAM20*, *IDI1*, *TNFRSF10B*, *AC093001.1*, *HIST1H2BC*, *HMGCR*, *SQSTM1*
- Negative: *PTGS2*, *DKK1*, *ALDOC*, *MDM2*, *FGA*, *TOP2A*, *NR4A1*, *TRPM3*, *SEMA3A*, *PDK4*

**Key GO terms**
- Positive: Cholesterol Biosynthesis (Reactome); SREBP Gene Expression; Cholesterol Biosynthesis Pathway; Metabolism of Steroids; Cholesterol Homeostasis
- Negative: Platelet Alpha Granule; Blood Clotting Cascade; Signal Transduction; Common Pathway of Fibrin Clot Formation; TNF-alpha Signaling via NF-kB
{% enddetails %}


### Perturbation summary

- Interesting that some top module emphasize moderate perturbation doses. Is this biologically meaningful?
- In the future, we should evaluate the ability to predict responses to combinations of perturbations by choosing linear combinations of output directions. Would've done it here, but lacked validation data.


---

## Conclusion

- Nice job
- Looks great
- Great work everyone


---