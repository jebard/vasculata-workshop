# 2. Quality Control: Where Analysis May End
Clustering never fails. Hand any matrix to tSNE or UMAP and you will likely get clean-looking clusters and a
UMAP you could put on a poster, even if the experiment was a technical disaster.
QC is the one place where the data can tell you it is bad, and it does so before
anything looks like biology.

## The core metrics

Three numbers per cell carry most of the signal. Seurat and scanpy call them
slightly different things.

| What it measures | Seurat | scanpy | What a bad value means |
|---|---|---|---|
| Total molecules (UMIs) in the cell | `nCount_RNA` | `total_counts` | Very low means empty droplet or dying cell. Very high means a likely doublet |
| Distinct genes detected | `nFeature_RNA` | `n_genes_by_counts` | Very low means a low-quality cell. Very high means a likely doublet |
| Percent of reads from mitochondrial genes | `percent.mt` | `pct_counts_mt` | High means a stressed or dying cell leaking cytoplasm |

```{admonition} Thresholds are not universal, so resist copy-paste
:class: warning
The tutorial defaults you see everywhere, meaning `nFeature > 200` and
`percent.mt < 5`, come from PBMCs. Tissue matters enormously. Cardiomyocytes,
skeletal muscle, and metabolically active vascular tissue legitimately carry far
more mitochondrial content, so a 5 percent cutoff can delete your most interesting
cells. Set thresholds by looking at the distribution for your own tissue, not by
inheriting a number.
```

```{admonition} The mitochondrial cutoff is the one I see people get wrong most
:class: danger
I want to be blunt about this one, because it's the QC mistake I see most. The
`percent.mt < 5` cutoff from the standard Seurat vignette is extremely stringent,
and it gets copied into pipeline after pipeline without a second thought. For PBMCs
it's fine. For real tissue it's often a disaster.

The problem is that a high mitochondrial percentage does not only mean a dying
cell. It also legitimately marks metabolically demanding cell types. Cardiomyocytes,
skeletal muscle, hepatocytes, kidney proximal tubule, brown fat, and plenty of
activated vascular and immune cells can sit at 20, 30, even 40 percent
mitochondrial reads while being perfectly healthy. A flat 5 percent filter does not
politely trim a few bad cells. It reaches in and deletes entire populations, and
because it removes them silently, you never see the cells you lost. Your UMAP still
looks great. It is just missing the biology.

So do not hard-code 5 percent. Two better habits:

- **Look first.** Plot the `percent.mt` distribution for each sample. If it is
  bimodal, the gate is obvious. If your tissue rides high across the board, that is
  a signal about the tissue, not a reason to throw it away.
- **Let the data set the gate.** Adaptive, distribution-based cutoffs (for example
  a median-absolute-deviation rule in scater, typically around 3 MADs) travel
  across tissues far better than a fixed number. Better still, **miQC** models
  mitochondrial percentage jointly with library complexity so it can tell a
  genuinely dying cell apart from a healthy cell that just runs hot.

One more note. In single-nucleus RNA-seq the expectation flips: you want very
little mitochondrial signal, so a high value there really is a red flag.
```

## Reading the plots, not the numbers

The QC violin and scatter plots are the deliverable, not a single threshold. What
I am looking for is this. A bimodal distribution in `nCount` or `nFeature` points
to a population of empty or dying droplets you can gate out. A tail of high-count,
high-gene cells usually means doublets. And a cluster that is only high in
`percent.mt` usually means dying cells masquerading as a cell type.

::::{tab-set}
:::{tab-item} Python (scanpy)
```python
import scanpy as sc

adata = sc.read_10x_h5("filtered_feature_bc_matrix.h5")
adata.var_names_make_unique()

# tag mitochondrial genes (human: MT-, mouse: mt-)
adata.var["mt"] = adata.var_names.str.startswith("MT-")
sc.pp.calculate_qc_metrics(adata, qc_vars=["mt"], inplace=True)

sc.pl.violin(adata, ["n_genes_by_counts", "total_counts", "pct_counts_mt"],
             jitter=0.4, multi_panel=True)

# filter, with thresholds set by looking at YOUR distributions
adata = adata[adata.obs.n_genes_by_counts > 200]
adata = adata[adata.obs.pct_counts_mt < 15]
```
:::
:::{tab-item} R (Seurat)
```r
library(Seurat)

counts <- Read10X_h5("filtered_feature_bc_matrix.h5")
obj <- CreateSeuratObject(counts, min.cells = 3, min.features = 200)

# percent mitochondrial (human: ^MT-, mouse: ^mt-)
obj[["percent.mt"]] <- PercentageFeatureSet(obj, pattern = "^MT-")

VlnPlot(obj, features = c("nFeature_RNA", "nCount_RNA", "percent.mt"), ncol = 3)

# filter, with thresholds set by looking at YOUR distributions
obj <- subset(obj, subset = nFeature_RNA > 200 & percent.mt < 15)
```
:::
::::

## The two silent killers: doublets and ambient RNA

Basic thresholds miss two artifacts that quietly distort every downstream result.

The first is doublets, meaning two cells captured in one droplet and read as a
single cell with a blended profile. They masquerade as rare intermediate or
transitional cell types, which is exactly the kind of finding people get excited
about. In Python you can detect and remove them with Scrublet, which ships with
scanpy, or scDblFinder by way of R. In R the common choices are DoubletFinder and
scDblFinder.

The second is ambient RNA, meaning free-floating transcripts from lysed cells that
get counted in every droplet. This is a serious contaminant in tissue with fragile
cells, and much vascular and immune tissue qualifies. It makes cell types look
more alike than they are. In Python I reach for CellBender, a deep-learning method
you run before or around Cell Ranger, or decontX. In R the common choices are
SoupX and decontX.

```{admonition} Why a vascular audience should care
:class: note
Ambient RNA and doublets both manufacture co-expression. If the endothelial marker
`PECAM1` and the smooth-muscle marker `ACTA2` show up in the same cell, the
exciting interpretation is a novel transitional state, and the boring
interpretation, which is usually the correct one, is a doublet or ambient
contamination. Rule out the boring one first.
```

## The QC mindset

QC is not a checkbox with fixed numbers. It is an iterative conversation with the
distributions. You plot them, set a provisional gate, cluster, look for a cluster
that is only a QC artifact, tighten the gate, and repeat. Do all of this before
you let yourself believe any biology, because once the UMAP appears, everyone,
including you, starts believing the clusters are real.
