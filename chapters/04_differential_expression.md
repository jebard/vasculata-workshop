# 4. Differential Expression: The Pseudoreplication Trap

So you have made it through QC, clustering, and annotation, and now you want the
payoff. You want to know which genes change between your conditions, say diseased
versus healthy, or treated versus control. This is where the single most common
and most expensive mistake in the whole field is waiting for you, and almost
nobody warns you about it. It is called pseudoreplication.

## What goes wrong

Here is the trap. You have three diseased animals and three healthy ones. Each
animal gives you a few thousand cells. You pick your endothelial cluster, run a
differential-expression test across condition, and the tool happily reports
hundreds of genes with p-values so small they look like typos. You feel great. You
should not.

The problem is that you did not have thousands of independent measurements. You had
six animals. Every cell from a given animal shares that animal's genetics, its
batch, its capture, its whole biological context, so those cells are not
independent of each other. When you treat each cell as its own replicate, you are
telling the statistics you have thousands of independent samples when you really
have six. That is pseudoreplication, and it inflates your false-positive rate
enormously.

```{figure} ../figures/pseudoreplication_schematic.png
:alt: Schematic contrasting per-cell testing with pseudobulk aggregation
:width: 100%
:align: center

The same experiment, counted two ways. On the left every cell is treated as a
replicate, so the test thinks it has thousands of independent samples and reports a
tiny p-value it never earned. On the right, each animal's cells are summed into one
pseudobulk profile, which gives you the honest sample size: six animals, three per
group. *Schematic with illustrative synthetic data.*
```

```{admonition} The analogy I use
:class: note
Measuring one person's height ten thousand times does not tell you the average
height of a population. You will get an extremely precise number, with tiny error
bars, that describes exactly one person. Running a per-cell test on three-versus-
three is the same move. Your confidence looks spectacular, but it is confidence
about the individuals you happened to sample, not about the conditions you care
about. Worse, a single unusual animal can drive the entire result, and the per-cell
test will never tell you.
```

This is not a hypothetical concern. When people have gone back and checked, the
naive per-cell methods produce long lists of "significant" genes that do not
reproduce, and the apparent signal often traces back to one or two samples.

## The fix: pseudobulk

The clean and, frankly, boring solution is to stop pretending cells are replicates
and go back to your real experimental unit, which is the biological sample. You do
this with pseudobulk. For each cell type, you sum the raw counts across all cells
within each sample. That gives you one expression profile per sample per cell type,
which is exactly a small bulk RNA-seq experiment. Then you hand those sample-level
profiles to the tools the bulk world spent twenty years validating, meaning
DESeq2, edgeR, or limma-voom, with your real number of replicates.

::::{tab-set}
:::{tab-item} Python (scanpy + decoupleR + PyDESeq2)
```python
import decoupler as dc

# sum raw counts to one profile per sample, per cell type
pdata = dc.get_pseudobulk(
    adata, sample_col="sample_id", groups_col="cell_type",
    layer="counts", mode="sum", min_cells=10, min_counts=1000)

# then run PyDESeq2 on the endothelial pseudobulk, condition as the design
endo = pdata[pdata.obs.cell_type == "Endothelial"]
# DeseqDataSet(counts=endo.X, metadata=endo.obs, design_factors="condition") ...
```
:::
:::{tab-item} R (Seurat + DESeq2)
```r
# sum raw counts to one profile per sample, per cell type
pb <- AggregateExpression(obj, assays = "RNA",
        group.by = c("cell_type", "sample_id"), return.seurat = FALSE)

# build the endothelial count matrix (genes x samples), then:
library(DESeq2)
dds <- DESeqDataSetFromMatrix(countData = endo_counts,
        colData = sample_meta, design = ~ condition)
dds <- DESeq(dds)
res <- results(dds)
```
:::
::::

If you would rather not wire this up by hand, several packages do the aggregation
and testing for you, including **muscat**, **dreamlet**, and **Libra** in R. If you
truly need a per-cell model, a mixed-effects model with a random effect for sample,
such as **NEBULA**, respects the replicate structure too, but honestly pseudobulk
is simpler, faster, and performs as well or better in most comparisons.

## The part no software can fix

```{admonition} You cannot buy power with more cells
:class: warning
This is the uncomfortable part. Pseudobulk protects you from false positives, but
it cannot manufacture statistical power you never had. If you profiled one diseased
mouse and one healthy mouse, you have an n of one per group, and no amount of
sequencing depth or clever modeling changes that. Sequencing more cells per animal
makes each animal's profile more precise. It does not give you more animals.

So the real fix happens before you ever touch a keyboard. Design the experiment
with genuine biological replicates, meaning independent animals or donors, and aim
for at least three per condition, more if you can afford it. More cells per sample
is nice. More samples is what lets you make a claim about the condition.
```

## This applies to spatial too

Do not think this is only a single-cell problem. Two Xenium sections cut from the
same block are not independent replicates of the animal, any more than two cells
are. The biological replicate is still the animal or the donor. Whenever you
compare conditions in Visium or Xenium, aggregate to the sample level and count
your replicates honestly, exactly as you would here.
