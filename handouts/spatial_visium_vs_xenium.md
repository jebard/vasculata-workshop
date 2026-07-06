# Spatial Decision Guide: Visium versus Xenium, One Page

*Vasculata 2026. Jonathan E. Bard, PhD. Department of Biochemistry and Genomics
and Bioinformatics Core, University at Buffalo.*

## Side by side

| | Visium | Xenium |
|---|---|---|
| Readout | Sequencing | Imaging (fluorescence) |
| Genes measured | Whole transcriptome | Targeted panel: few hundred up to ~5,000 (Xenium Prime 5K), plus add-ons |
| Resolution | 55 µm spot (v1 and v2), or 2 µm bins (HD) | Individual transcripts, subcellular |
| Primary unit | Spot or bin, which is a mixture of cells | Segmented single cell |
| Discovery | Unbiased, any gene | Limited to panel genes (even at ~5,000, still a fixed list) |
| Main analytical headache | Deconvolution | Segmentation |

Rule of thumb. Use Visium when you do not yet know which genes matter, meaning
discovery across the whole transcriptome. Use Xenium when you know your genes and
need single-cell spatial resolution.

## Visium: the binning and mixture problem

A spot contains several cells, so its profile is a blend. Clustering spots gives
you regions, not cell types. Differential expression is confounded by cell
composition, so more of cell type X masquerades as gene Y changing. This matters a
great deal for vascular remodeling, where proportions shift. The fix is
deconvolution with a matched scRNA reference, using cell2location, RCTD, SPOTlight,
or Tangram. For Visium HD bin size, small bins give resolution but are sparse and
noisy, while large bins are smooth but mix cells, and there is no universal right
answer.

## Xenium: the segmentation problem

Building a cell by gene matrix requires assigning each transcript to a cell, and
errors here manufacture false biology. Under-segmentation merges cells and gives
you doublet-like blended profiles. Over-segmentation splits cells and gives you
fake low-count populations. Spillover assigns a transcript to the wrong neighbor
and produces false co-expression.

In a vessel wall, where endothelium, smooth muscle, and pericytes sit within
microns of each other, spillover is the baseline, not an edge case. If a result
depends on cells co-expressing two lineage markers, suspect segmentation first. For
better boundaries, run 10x's multimodal cell-segmentation kit with a membrane
stain rather than nucleus expansion. To re-segment from raw transcripts, use
Baysor, Cellpose, ProSeg, or Segger.

## Ecosystem note

For Visium, R (Seurat) and Python (squidpy) are both fine. For Xenium, Python with
squidpy and SpatialData clearly leads for spatial statistics, meaning
neighborhoods, domains, and cell-to-cell communication.

## Downstream ripple, the one thing to remember

Your spatial unit is not a cell until you make it one. Visium needs deconvolution,
and Xenium needs good segmentation. Skip either step and every downstream
comparison inherits the error, invisibly.
