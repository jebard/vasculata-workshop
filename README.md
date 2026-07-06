# Bioinformatics for Next-Gen Sequencing: From Single Cell to Spatial Architecture

**Vasculata 2026, Workshop Part 2**

Jonathan E. Bard, PhD ;
Donald A. Yergeau, PhD ;
Tyler M. Chinsky, MS 

Genomics and Bioinformatics Core. 
Center of Excellence in Bioinformatics and Life Sciences
Business and Entrepreneurial Partnerships
State University of New York at Buffalo.

This is a read-along guide to the secondary analysis of 10x Genomics single-cell
and spatial data. I built it as the companion to the live workshop, and I meant
it to be read and re-read at your own pace rather than run live in the room.

## What this covers

1. The pipeline, meaning where bioinformatics actually starts and what I mean when
   I say primary versus secondary analysis.
2. Single-cell RNA-seq quality control, and the metrics that make or break a
   dataset.
3. Cell annotation, from manual markers to reference-based and foundation-model
   methods.
4. Differential expression, and the pseudoreplication trap that quietly wrecks
   condition comparisons if you let cells stand in for biological replicates.
5. Visium, including spot and bin capture, the binning tradeoff, and
   deconvolution.
6. Xenium and segmentation, meaning imaging-based spatial and why cell boundaries
   are the whole ballgame.
7. Seurat versus scanpy and squidpy, with an honest pro and con at each decision
   point.
8. A glossary and resource list, covering the jargon, the tools, and where to go
   next.

## How to use it

If you are watching the workshop, everything I demo live is written up here so you
can revisit it afterward. If you are analyzing your own data later, each chapter
shows the R way (Seurat) and the Python way (scanpy and squidpy) of doing the same
thing, so you can follow along in whichever ecosystem your lab already uses. The
code is there for reference. The figures come from public 10x datasets so that
everything is reproducible, while the live demo uses real vascular tissue.

## Building the book locally (optional)

```bash
pip install -r requirements.txt
jupyter-book build .
open _build/html/index.html
```

## Handouts

Two one-page references live in [`handouts/`](handouts/). The first,
`scrna_qc_cheatsheet.md`, covers QC metrics, starting thresholds, and the
annotation tiers. The second, `spatial_visium_vs_xenium.md`, is a side-by-side
decision guide.

## Acknowledgments

Prepared for Vasculata 2026, hosted by the North American Vascular Biology
Organization. With thanks to the Department of Biochemistry and the Genomics and Bioinformatics Core Faciltiy at the Center of Excellence in Bioinformatics and Life Sicences,  Business and
Entrepreneurial Partnerships at the University at Buffalo. Claude AI model Opus 4.8 was used to scaffold, the code, initial writing, and github deployment. All sections reviewed by Bard and Chinsky for accuracy.
