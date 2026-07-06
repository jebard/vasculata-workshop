# 3. Cell Annotation: From Markers to Foundation Models

Clustering gives you groups. Annotation gives them names. This is where your domain
knowledge comes back in, and it's also the step that has changed the most in the
last few years. I think of it in three tiers, with more automation at each step.
The important caveat is that no amount of automation lets you skip the biologist
looking at the result, and that biologist is you.

## Tier 1: manual, marker-based

You cluster first, then you label the clusters by which canonical markers they
express. It's old-fashioned, and it's still the gold standard for a well-studied
tissue. It's also the thing every automated method is quietly checked against, so
it's worth knowing well.

```{admonition} A starter vascular marker set
:class: note
| Cell type | Canonical markers |
|---|---|
| Endothelial | `PECAM1` (CD31), `CDH5` (VE-cadherin), `VWF`, `CLDN5` |
| Arterial EC | `GJA5`, `SEMA3G`, `HEY1` |
| Venous EC | `NR2F2`, `ACKR1` |
| Lymphatic EC | `PROX1`, `LYVE1`, `PDPN` |
| Smooth muscle | `ACTA2`, `MYH11`, `TAGLN`, `CNN1` |
| Pericyte | `PDGFRB`, `RGS5`, `NOTCH3`, `KCNJ8` |
| Fibroblast | `COL1A1`, `PDGFRA`, `DCN`, `LUM` |
| Immune (pan) | `PTPRC` (CD45) |
```

::::{tab-set}
:::{tab-item} Python (scanpy)
```python
sc.tl.rank_genes_groups(adata, "leiden", method="wilcoxon")
sc.pl.rank_genes_groups(adata, n_genes=10)

sc.pl.dotplot(adata, ["PECAM1","CDH5","ACTA2","MYH11","PDGFRB","COL1A1"],
              groupby="leiden")

adata.rename_categories("leiden",
    ["Endothelial","Smooth muscle","Pericyte","Fibroblast", ...])
```
:::
:::{tab-item} R (Seurat)
```r
markers <- FindAllMarkers(obj, only.pos = TRUE,
                          min.pct = 0.25, logfc.threshold = 0.25)

DotPlot(obj, features = c("PECAM1","CDH5","ACTA2","MYH11","PDGFRB","COL1A1"))

obj <- RenameIdents(obj,
  `0` = "Endothelial", `1` = "Smooth muscle", `2` = "Pericyte", ...)
```
:::
::::

```{figure} ../figures/annotation_umap_public.png
:alt: UMAP of public PBMC data with clusters labeled by cell type
:width: 90%
:align: center

What you're working toward: a UMAP where every cluster has a name. This is the
classic public PBMC dataset, and each of these labels came from reading canonical
markers off the clusters. It's the same manual logic you'd apply to a vascular
dataset, just done here on cells everyone already knows. *Public 3k PBMC dataset,
10x Genomics, annotated following the standard scanpy tutorial.*
```

## Tier 2: reference-based, automated

Now you let a computer do the first pass. You transfer labels from a curated atlas
onto your cells, which is fast, reproducible, and consistent from dataset to
dataset. The obvious limit is that it's only as good as the reference, and the
reference may know nothing about your rare or perturbed states.

| Tool | Ecosystem | Notes |
|---|---|---|
| Azimuth | R and Seurat | Web app plus package, with strong curated human references |
| SingleR | R | Correlation to reference bulk or single-cell profiles, no training needed |
| CellTypist | Python | Logistic-regression models, and the immune atlas is excellent |
| scANVI, scArches | Python | Deep models that map onto a reference and integrate at the same time |

## Tier 3: foundation models and LLMs (the recent advance)

This is the newest tier, and it comes in two flavors worth knowing.

The first is LLM-assisted labeling. You hand a cluster's top marker genes to a
large language model and let it propose a cell type, for example with GPTCelltype.
It's genuinely fast for a first pass. Just remember its answers are hypotheses, not
conclusions.

The second is single-cell foundation models, meaning models pre-trained on tens of
millions of cells, like scGPT, Geneformer, and scFoundation. They produce
embeddings and annotations and can be fine-tuned to your data. They're powerful and
getting better fast, but they're heavier to run and still being validated against
plain manual labels.

The third is 10x Genomics' own automated cell type annotation, now built directly into 
the Cell Ranger family of pipelines, available in Cell Ranger, Cell Ranger ARC, and 
Space Ranger. You can choose a cloud based model co-developed with the Broad Institute, 
or the Satija lab's Pan-Human Azimuth model, which runs locally, avoiding the cloud route 
where your data would leave your institution under 10x's terms of use. The convenience is 
real: labels appear automatically as part of routine processing, no extra 
code required. That's also the risk. A label bundled with your standard output feels more 
authoritative than it is. The cloud model is still in beta, and like any reference based 
method, it will confidently label cells that don't really match anything it's seen. 
Treat it as a starting point, not a verdict.

```{admonition} The honest take
:class: warning
Automation removes tedium, not responsibility. Every one of these methods,
reference-based or foundation model, will confidently label cells it has never seen
anything like, and those are usually your perturbed, diseased, or rare vascular
states, meaning the exact cells you care about most. So always sanity-check the
automated calls against a handful of canonical markers before you trust them.
```

## A practical workflow

I use all three tiers together, not one in isolation. Run a reference method, or an
LLM, to get a fast first pass. Confirm and refine it with canonical markers. Then
manually resolve the clusters that actually matter to your biology. Automation gets
you most of the way there in minutes, and the last stretch, which is almost always
the interesting part, is still a human reading a dot plot.
