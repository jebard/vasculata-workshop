# 6. Xenium: Single Molecules, and Why Segmentation Is Everything

Xenium flips the Visium trade completely on its head. Instead of the whole
transcriptome at multi-cell resolution, you get a targeted gene panel, now anywhere
from a few hundred genes up to around 5,000, imaged at subcellular resolution. You literally see individual transcript molecules sitting at
their own x and y coordinates. Here's the catch. Those molecules show up as a point
cloud, and something has to decide which cell each one belongs to. That decision is
called segmentation, and it is the whole ballgame.

```{figure} ../figures/xenium_multiplex_tongue.png
:alt: Multiplexed Xenium fluorescence image of mouse tongue tissue
:width: 480px
:align: center

A raw Xenium image, before any of the analysis in this chapter. The colors are
multiplexed stains (DAPI, ATP1A1/CD45/E-Cadherin, 18S, alphaSMA/Vimentin), and the
epithelial architecture is beautifully preserved. Every downstream number depends
on how well we can turn this picture into individual cells. *Real Xenium data from
Dr. Keith Kirkwood's lab, University at Buffalo.*
```

## How Xenium differs from Visium

| | Visium | Xenium |
|---|---|---|
| Readout | Sequencing | Imaging (fluorescence) |
| Genes | Whole transcriptome | Targeted panel: a few hundred on the standard assays, up to about 5,000 with Xenium Prime 5K, plus custom add-ons |
| Resolution | Spot or bin | Individual transcripts, subcellular |
| Primary unit | Spot, which is a mixture | Segmented cell |
| Discovery | Unbiased | Limited to the panel (but the panel can now be large) |

```{figure} ../figures/xenium_process_panel.png
:alt: Three-panel Xenium process: morphology, cell segmentation, and marker gene analysis
:width: 100%
:align: center

The whole Xenium process on a single field of view, here a Low-Fat mouse-tongue
section. On the left is the high-definition morphology image (multiplexed stains for
DAPI, ATP1A1/CD45/E-Cadherin, 18S, and alphaSMA/Vimentin). In the middle, that same
region after cell segmentation, with each cell drawn as a polygon and colored by
cluster. On the right, marker-gene analysis over those same cells, with transcripts
colored by gene (Klf4 in magenta, Krt78 in orange, Trp63 in blue). The key thing to
notice is that everything on the right depends entirely on getting the middle panel
right. *Real Xenium data from Dr. Keith Kirkwood's lab, University at Buffalo.*
```

```{admonition} The panel is a real constraint
:class: warning
Xenium only measures the genes you chose ahead of time. Those panels have grown a
lot, from a few hundred genes on the standard assays to around 5,000 with the
Xenium Prime 5K, and you can add custom genes on top. That is a huge menu, but it is
still a menu. If your key marker, or your exciting surprise result, involves a gene
that isn't on the panel, it simply does not exist in your data. Even at 5,000 genes
you are choosing a targeted list, not capturing the whole transcriptome, so design
the panel like you mean it.
```

## The segmentation problem

To build a cell-by-gene matrix, the pipeline has to draw cell boundaries and assign
every transcript to a cell, and every method for doing this is imperfect.
Nucleus-plus-expansion segments the nuclei from DAPI and then just expands outward a
fixed distance. It's simple, but that expansion is basically a guess, and it ignores
cell shape entirely. Boundary-stain, or multimodal, segmentation uses 10x's
cell-segmentation kit to add membrane and interior stains, so the boundaries are
actually measured instead of assumed. It's much better, if you ran it.

```{figure} ../figures/xenium_transcripts.png
:alt: Individual Xenium transcript molecules plotted over tissue
:width: 90%
:align: center

This is what the pipeline actually starts from: individual transcript molecules,
each a dot at its own coordinate. There are no cells here yet. Segmentation is the
step that decides which dots belong together, and you can already see how a molecule
sitting between two cells could go either way. *Real Xenium data from Dr. Keith
Kirkwood's lab, University at Buffalo.*
```

Two failure modes are worth naming out loud, because you will meet both.

```{admonition} Under-segmentation versus over-segmentation
:class: important
Under-segmentation merges neighboring cells into one, and now you've got a blended,
doublet-like profile, say `PECAM1` and `ACTA2` in the same "cell."
Over-segmentation splits one cell into several, and now you've got fragmented,
low-count "cells" that look like a whole new population but aren't. And transcript
spillover is the third problem, where a molecule from cell A gets assigned to
neighbor B, smearing your cell-type boundaries and manufacturing co-expression that
was never real.
```

This is where people get misled. Because segmentation errors create co-expression,
they masquerade as exactly the biology everyone gets excited about,
meaning transitional states, novel double-positive populations, cell-to-cell
communication. And in a densely packed vessel wall, where endothelium, smooth
muscle, and pericytes are all sitting within microns of each other, spillover isn't
some rare edge case. It's the baseline you have to actively rule out.

## Re-segmentation: do not accept the default blindly

Good news: you are not stuck with whatever the onboard segmentation handed you.
Several tools will re-assign transcripts straight from the raw molecule coordinates,
and they often do a better job.

| Tool | Idea |
|---|---|
| Baysor | Probabilistic segmentation from transcript density and morphology |
| Cellpose | Deep-learning nuclear and cell masks from the images |
| ProSeg | Probabilistic cell assignment, built to scale |
| Segger | Graph-based transcript-to-cell assignment |

```{tip}
Any time a Xenium result hinges on cells co-expressing markers of two different
lineages, my first guess is a segmentation artifact, not biology. So go check
whether those "cells" happen to sit right on the boundary between two pure
populations, and try a re-segmentation before you let yourself believe it.
```

## Why Python and squidpy lead here

Xenium is where the ecosystem choice finally stops being a wash. The image-based,
spatial-statistics workflows, meaning neighborhood enrichment, co-occurrence,
spatial domains, and ligand-receptor analysis over real coordinates, are all
Python-first through squidpy and the SpatialData framework. Seurat v5 can read and
plot Xenium, but the spatial-analysis toolkit is noticeably thinner. Chapter 7 has
the full comparison.

## Typical Xenium workflow

In order, a Xenium analysis runs like this:

1. Xenium Onboard output, meaning the cells, the transcripts, and the images
2. Optional re-segmentation from transcript coordinates
3. QC on transcripts per cell, control-probe rate, and segmentation sanity
4. Normalization, clustering, and annotation (Chapter 3)
5. Spatial statistics, meaning neighborhoods, domains, and cell-to-cell
   communication
