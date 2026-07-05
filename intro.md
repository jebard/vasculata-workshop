# From Single Cell to Spatial Architecture

**Bioinformatics for Next-Generation Sequencing Datasets**

*Vasculata 2026, Workshop Part 2*

Jonathan E. Bard, PhD. Research Assistant Professor, Department of Biochemistry,
Jacobs School of Medicine and Biomedical Sciences, University at Buffalo.
Associate Director, Genomics and Bioinformatics Core. Affiliated with Business and
Entrepreneurial Partnerships at the University at Buffalo.

Single-cell and spatial technologies have reshaped how we study tissue, and
nowhere is that clearer than in vascular biology, where the interesting story is
almost always about which cell sits next to which. This guide is the companion to
the live workshop. It walks through the secondary analysis of 10x Genomics
single-cell and spatial data, meaning the steps that happen after the sequencer
and the vendor pipeline hand you a count matrix.

## What you will leave with

I want you to walk away with a mental map of the pipeline and a clear sense of
where your judgment as a scientist actually enters. You should understand the
handful of quality-control metrics that decide whether a dataset is trustworthy,
and how to read them. You should know the current menu of cell-annotation
strategies, from canonical markers to foundation models. You should have a working
understanding of the Visium versus Xenium tradeoff, meaning spot binning versus
cell segmentation, and how that choice ripples all the way downstream. And you
should have an honest, decision-by-decision comparison of the Seurat (R) and
scanpy and squidpy (Python) ecosystems.

## How to read this

```{tip}
You are not expected to run anything during the workshop. This is a read-along
guide, so the code is shown mainly so you can reproduce the ideas later in your
own lab, in whichever language you already use. Where a step exists in both
ecosystems, you will see R and Python side by side in tabbed code blocks.
```

The figures here come from public 10x datasets so that every result is
reproducible. The live demo uses real vascular tissue, because I want you to see
what these plots look like on data that has not been sanded down for a tutorial.

## A note on scope

Thirty minutes is not enough time to make anyone a bioinformatician, and I am not
going to pretend otherwise. My goal is narrower and, I think, more useful. I want
to make you a better-informed collaborator and data consumer, someone who knows
which questions to ask, which plots to demand, and where the analysis can quietly
go wrong before anyone notices.
