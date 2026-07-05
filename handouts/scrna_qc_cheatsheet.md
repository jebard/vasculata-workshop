# scRNA-seq QC and Annotation, One-Page Cheat Sheet

*Vasculata 2026. Jonathan E. Bard, PhD. Department of Biochemistry and Genomics
and Bioinformatics Core, University at Buffalo.*

## The 3 QC metrics (per cell)

| Metric | Seurat | scanpy | Too low | Too high |
|---|---|---|---|---|
| Total UMIs | `nCount_RNA` | `total_counts` | empty or dying droplet | doublet |
| Genes detected | `nFeature_RNA` | `n_genes_by_counts` | low-quality cell | doublet |
| Percent mitochondrial | `percent.mt` | `pct_counts_mt` | (not a concern) | stressed or dying cell |

**Starting points (PBMC defaults, so adjust for your tissue):** `nFeature > 200`,
and `percent.mt` under 5 to 20 percent, with the higher end for muscle, cardiac,
and metabolic tissue.

Thresholds are dataset-specific. Set them by looking at your own violin plots, and
never copy a tutorial number. A 5 percent mito cutoff can delete your most
metabolically active cells.

## The 2 silent killers

| Artifact | What it fakes | R tools | Python tools |
|---|---|---|---|
| Doublets | false transitional cell types | DoubletFinder, scDblFinder | Scrublet |
| Ambient RNA | false co-expression across types | SoupX, decontX | CellBender, decontX |

If two lineage markers, for example `PECAM1` and `ACTA2`, appear in one cell,
suspect a doublet or ambient RNA before you believe a novel state.

## Standard workflow

Filter on QC, then normalize, then select highly variable genes, then run PCA,
then build neighbors, then compute a UMAP, then cluster with Leiden or Louvain,
then annotate, then compare across conditions.

Remember that clustering never fails. It produces pretty clusters even from
garbage. QC is your only checkpoint, and it happens before anything looks like
biology.

## Annotation, three tiers

1. Manual markers. Cluster, then label by canonical genes. This is the gold
   standard.
2. Reference-based. Azimuth and SingleR in R, or CellTypist and scANVI in Python.
3. Foundation and LLM methods. GPTCelltype, scGPT, Geneformer. Treat these as
   hypotheses and verify them.

**Vascular markers.** Endothelial: `PECAM1`, `CDH5`, `VWF`. Arterial: `GJA5`,
`HEY1`. Venous: `NR2F2`, `ACKR1`. Lymphatic: `PROX1`, `LYVE1`. Smooth muscle:
`ACTA2`, `MYH11`, `TAGLN`. Pericyte: `PDGFRB`, `RGS5`, `NOTCH3`. Fibroblast:
`COL1A1`, `PDGFRA`, `DCN`. Immune: `PTPRC`.

Automation removes tedium, not responsibility. Always confirm automated calls
against a few canonical markers.
