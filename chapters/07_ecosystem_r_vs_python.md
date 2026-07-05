# 7. Seurat versus scanpy and squidpy: An Honest Pro and Con

"Should I learn R or Python for this?" is far and away the question I get most, and
the honest answer is that it depends on what you're analyzing. So instead of
picking a side and defending it, let's look at where each ecosystem actually wins.

## The headline

```{admonition} The takeaway in one sentence
:class: important
For core single-cell work the two are basically a tie, so use whatever your lab and
your collaborators already use. The more spatial and image-based you go, the more
Python earns its keep, until you hit Xenium and it becomes the practical default.
```

## Decision by decision

```{list-table}
:header-rows: 1

* - Task
  - R (Seurat)
  - Python (scanpy and squidpy)
  - Edge
* - scRNA QC, clustering, UMAP
  - Seurat, mature with great docs
  - scanpy, fast and scales to millions
  - Tie, so lab preference wins
* - Annotation
  - Azimuth, SingleR
  - CellTypist, scANVI, foundation models
  - Python, for automated and newest methods
* - Visium
  - Load10X_Spatial, RCTD, SPOTlight
  - squidpy, cell2location, Tangram
  - Tie, with a slight Python edge on deconvolution
* - Xenium and imaging spatial
  - v5 reads and plots, but thinner stats
  - squidpy and SpatialData, native
  - Python, clearly
* - Very large datasets (over one million cells)
  - Memory-heavy, though BPCells helps
  - AnnData with backed mode or Dask scales well
  - Python
* - Publication-ready plots fast
  - The ggplot2 ecosystem is superb
  - matplotlib works, but it is more fiddly
  - R, for figure polish
```

## Why the divide exists

Two reasons, mostly. The first is data structures. Seurat is built around the
`Seurat` object, while Python is built around AnnData, meaning `.X`, `.obs`, `.var`,
and `.obsm`, and now SpatialData for the multi-modal spatial stuff. Most new
spatial-omics methods get written against AnnData first, so they tend to land in
Python first, and R plays catch-up. The second reason is just momentum. Foundation
models like scGPT and Geneformer, squidpy's spatial statistics, CellBender, they're
all Python-native. Seurat's strengths are its maturity, its documentation, and a
ggplot2 figure ecosystem that nothing in Python quite matches.

## Interoperability: you do not have to fully commit

This part is worth knowing. The two ecosystems talk to each other, so a mixed
workflow isn't a compromise, it's normal. You can convert between `.h5ad` and
`.rds` with `sceasy`, `zellkonverter`, or the `anndata` package in R. You can run
Python from inside R with `reticulate`, which means you can call scanpy or scrublet
without ever leaving your R session. The pattern I actually use all the time is to
do QC and clustering in whichever I like, then hop over to Python for the Xenium
spatial statistics.

## Practical recommendation for this audience

```{admonition} If you are starting fresh in a vascular lab
:class: tip
If your work is mostly scRNA-seq and Visium and your lab already lives in R, then
Seurat is a perfectly good home, and you only need to reach for Python when you want
a Python-only method. If you're planning Xenium or other imaging-based spatial work,
I'd invest in Python now, meaning scanpy plus squidpy plus SpatialData, because
that's plainly where the field and the tools are heading. And either way, learn just
enough of the `.h5ad` and AnnData format to move your data across the bridge. That
one skill future-proofs you against the whole debate.
```
