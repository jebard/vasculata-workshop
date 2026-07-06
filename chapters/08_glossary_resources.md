# 8. Glossary and Resources

Every field hides behind its jargon, and this one is worse than most. So here's a
plain-language cheat sheet for the terms that fly around in single-cell and spatial
work, followed by the links I'd actually send you.

## Glossary

**Ambient RNA.** Free-floating transcripts from lysed cells counted in every
droplet. It is a contaminant that makes cell types look more alike. Tools include
SoupX, CellBender, and decontX.

**AnnData (`.h5ad`).** Python's standard single-cell data container. It is the
lingua franca that most new methods target first.

**Barcode (cell).** The DNA tag identifying which droplet or cell a molecule came
from.

**Bin (Visium HD).** An aggregated square of the fine capture grid, at 2, 8, or 16
µm. Larger bins mean more counts but more cell mixing.

**Cell Ranger and Space Ranger.** 10x's primary-analysis pipelines, for scRNA and
Visium respectively, that turn reads into a count matrix.

**Count matrix.** The genes by cells (or spots or bins) table of molecule counts.
It is the input to secondary analysis.

**Deconvolution.** Estimating the cell-type proportions inside a multi-cell Visium
spot. Tools include cell2location, RCTD, SPOTlight, and Tangram.

**Doublet.** Two cells captured as one, which reads as a false intermediate cell.
Tools include Scrublet, DoubletFinder, and scDblFinder.

**Foundation model (single-cell).** A model pre-trained on millions of cells, such
as scGPT or Geneformer, that produces embeddings and annotations.

**Leiden and Louvain.** Graph-based clustering algorithms used to group cells.

**percent.mt and pct_counts_mt.** The fraction of a cell's reads that come from
mitochondrial genes. A high value can mean a stressed or dying cell, and the
threshold is tissue-dependent (see Chapter 2, because people get this one wrong a
lot).

**Pseudobulk.** Summing counts across all cells of one type within a sample to make
a sample-level profile, so you can test differential expression on your real
biological replicates. See Chapter 4.

**Pseudoreplication.** Treating cells from the same sample as independent
replicates. It wildly inflates false positives in differential expression. See
Chapter 4.

**Segmentation.** Assigning transcripts to cells in imaging spatial, meaning
Xenium. It is the dominant error source. Tools include Baysor, Cellpose, ProSeg,
and Segger.

**Spot (Visium).** A 55 µm barcoded capture area containing several cells.

**UMI.** Unique Molecular Identifier. It tags an original molecule so that PCR
duplicates are not double-counted. The `nCount_RNA` and `total_counts` metrics
count UMIs.

**UMAP.** A two-dimensional visualization of high-dimensional cell states.
Distances between far-apart clusters are not meaningful, so read local structure
rather than global geometry.

## Resources

### Documentation and tutorials

- Seurat: https://satijalab.org/seurat/
- scanpy: https://scanpy.readthedocs.io
- squidpy: https://squidpy.readthedocs.io
- SpatialData: https://spatialdata.scverse.org
- Single-cell best practices (the scverse book): https://www.sc-best-practices.org

### Annotation

- Azimuth: https://azimuth.hubmapconsortium.org
- CellTypist: https://www.celltypist.org
- 10X Annotations: https://www.10xgenomics.com/support/software/cell-ranger/latest/algorithms-overview/cr-cell-annotation-algorithm

### Differential expression (pseudobulk)

- muscat: https://bioconductor.org/packages/muscat
- PyDESeq2: https://pydeseq2.readthedocs.io
- decoupleR: https://decoupler-py.readthedocs.io

### Spatial deconvolution and segmentation

- cell2location: https://cell2location.readthedocs.io
- Baysor: https://github.com/kharchenkolab/Baysor
- Cellpose: https://www.cellpose.org

### Vascular references

- Tabula Sapiens, a multi-organ human atlas: https://tabula-sapiens.sf.czbiohub.org
- Human Lung Cell Atlas, with rich vascular endothelium: https://www.lungmap.net

## Get help at UB

I run analysis and consultation through the Genomics and Bioinformatics Core at the
Jacobs School of Medicine and Biomedical Sciences, and my honest advice is to come
find us before you sequence, not after. The cheapest way to fix a dataset is to
design it right in the first place, and by the time the data is bad, your options
are mostly damage control. So reach out through the Core, and we can talk through
experimental design, pipeline processing, and whatever downstream analysis you're
staring down.

This workshop was prepared with the support of the Department of Biochemistry and
Business and Entrepreneurial Partnerships at the University at Buffalo.

This is a companion guide to Vasculata 2026, Workshop Part 2, by Jonathan E. Bard,
PhD.
