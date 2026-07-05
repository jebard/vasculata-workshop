# 1. The Pipeline: Where Bioinformatics Starts

Before you can do anything interesting, meaning any clustering or spatial
statistics, your raw sequencing has to become a count matrix. That's just a table
of how many molecules of each gene showed up in each cell, or spot, or bin. It's
easy to treat this as plumbing, but understanding where that table comes from tells
you which problems are already baked in by the time you open your analysis
notebook.

## Primary versus secondary analysis

```{admonition} The one distinction to hold onto
:class: important
Primary analysis turns raw instrument signal into a count matrix. It's mostly
automated, and it's usually run by the core or the vendor pipeline. Secondary
analysis turns that matrix into biology. That's where your decisions live, meaning
filtering, annotation, and comparison, and it's where basically every error that
ends up in a figure actually gets made.
```

### Primary analysis (mostly not your job, but know what it did)

| Step | scRNA-seq | Spatial (Visium and Xenium) |
|---|---|---|
| Raw signal | BCL to FASTQ (BCL Convert) | Same for Visium; Xenium images processed on-instrument |
| Alignment and counting | Cell Ranger (`count`) | Space Ranger (Visium), Xenium Onboard Analysis |
| Cell calling | Barcode knee, EmptyDrops | Spots under tissue (Visium), segmentation (Xenium) |
| Output | Filtered feature-barcode matrix | Matrix plus spatial coordinates plus image |

It's worth pausing here. The vendor pipeline makes real decisions for you, and you
inherit every one of them. The two that matter most are which barcodes
count as cells (scRNA and Visium) and where the cells actually are (Xenium
segmentation). Chapter 6 is entirely about that last one, because in imaging-based
spatial it's the single biggest source of error you'll deal with.

### Secondary analysis (this is the workshop)

Now, the nice part. The single-cell backbone is remarkably consistent no matter
which tool you pick up. In order, it runs like this:

1. Count matrix
2. Quality control and filtering (Chapter 2)
3. Normalization
4. Feature selection (highly variable genes)
5. Dimensionality reduction (PCA, then UMAP)
6. Clustering (Leiden or Louvain)
7. Cell-type annotation (Chapter 3)
8. Differential expression and comparison across conditions (Chapter 4)

Spatial data runs that exact same backbone and then adds a spatial layer on top,
meaning neighborhood analysis, spatial domains, and cell-to-cell communication
(Chapters 5 and 6).

## Why this map matters for you

Two things fall out of this immediately, and both of them will save you.

First, garbage upstream is invisible downstream. A UMAP will always hand you pretty
clusters, even from an experiment that completely failed. I cannot stress this
enough. The QC step in Chapter 2 is your only real checkpoint, and it happens
before anything on your screen looks like biology.

Second, the count matrix is not raw truth. It's already the product of alignment
choices and cell-calling thresholds somebody else set. So when a result looks
weird, the answer is sometimes sitting upstream of where you're staring.

```{tip}
When a collaborator hands you "the data," I'd ask three questions before anything
else. What version of the vendor pipeline made this? What reference genome
or transcriptome did it use? And is this the filtered matrix or the raw one? The
answers genuinely change what you should do next.
```
