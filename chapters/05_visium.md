# 5. Visium: Spots, Bins, and the Deconvolution Problem

Visium makes a trade. It keeps the whole transcriptome and adds location, so you
get to measure nearly every gene, but you measure it at a spatial unit that is not
a single cell. Hold onto that one fact, because it drives every analysis decision
that follows.

## How capture works

Your tissue goes down on a slide printed with spatially barcoded capture spots. The
polyadenylated mRNA diffuses down, gets captured, and the barcode records where it
came from. That's the whole trick.

| Platform | Capture unit | Cells per unit | Transcriptome |
|---|---|---|---|
| Visium (v1 and v2) | 55 µm spot | About 1 to 10, depending on tissue | Whole |
| Visium HD | 2 µm bins, binnable to 8 and 16 µm | Sub-cellular up to a few | Whole |

```{figure} ../figures/visium_key_metrics.png
:alt: Visium HD run summary metrics
:width: 480px
:align: center

The kind of top-line numbers you get from a Visium HD run. These are your first QC
gut-check, meaning how many bins fell under tissue, how deeply each one was
sequenced, and how many genes you actually saw. *Representative Visium HD run,
courtesy of Dr. Remi Adelaiye-Ogala, University at Buffalo.*
```

## The binning tradeoff (Visium HD)

Visium HD captures on a fine 2 µm grid, but you don't actually analyze it there.
You aggregate grid squares into bins first, and picking that bin size is a real
decision with real consequences.

```{admonition} Smaller bins versus larger bins
:class: important
Small bins, in the 2 to 8 µm range, give you near-single-cell resolution, but each
bin is sparse and noisy because it holds so few molecules, and the bins don't care
where your cell boundaries are. Large bins, around 16 µm, are smoother and pack in
more counts, which makes clustering easier, but now each bin blends several cells
together, and that smears out exactly the cell-to-cell distinctions you bought
spatial data to see in the first place.
```

```{figure} ../figures/visium_bin_metrics.png
:alt: Visium HD bin metrics comparing 8 micron and 16 micron bins
:width: 650px
:align: center

The tradeoff in plain numbers, from one real dataset. Going from 8 µm to 16 µm bins
drops you from 552,642 bins to 139,446, but the mean UMIs per bin jumps from about
67 to 268. That's the whole story: coarser bins are far less sparse, but there are
far fewer of them and each one blends more cells together. *Representative Visium
HD run, courtesy of Dr. Remi Adelaiye-Ogala, University at Buffalo.*
```

So there's no universally correct bin size. I go fine when I need to resolve
something tightly packed, like a vessel wall, and coarser when I want robust
region-level signal. A lot of workflows now fit cell boundaries onto the HD grid,
using 10x's own cell-segmentation output or a tool like Bin2cell, to recover
something close to single cells. And the moment you do that, you've inherited the
Xenium segmentation problem from Chapter 6.

## The consequence: a spot is a mixture

This is the subtle part. Because a classic Visium spot holds several cells, its
expression profile is a blend of all of them. So if you cluster spots naively, you
get regions, not cell types, and any differential-expression result you run is
confounded by cell composition instead of real per-cell-type change. The fix is
deconvolution, meaning you estimate the cell-type proportions inside each spot,
usually with a single-cell reference from the same tissue.

| Tool | Ecosystem | Approach |
|---|---|---|
| cell2location | Python | Bayesian, and excellent with a good scRNA reference |
| RCTD (spacexr) | R | Robust and widely used |
| SPOTlight | R | NMF-based |
| Stereoscope, Tangram | Python | Probabilistic and deep mapping |

```{admonition} Why this matters downstream
:class: warning
Say you compare diseased versus healthy Visium spots without accounting for
composition. That apparent gene-expression change you found is very often just more
of cell type X in one condition, not any gene actually changing inside a cell type.
And for vascular remodeling, where cell proportions swing dramatically, this is an
easy way to mislead yourself. So deconvolve, or interpret with real caution.
```

## Typical Visium workflow

In order, a Visium analysis runs like this:

1. Space Ranger output, meaning the matrix, the image, and the coordinates
2. QC on spots under tissue, counts and genes per spot, and percent mitochondrial
3. Normalization
4. Clustering of spots into spatial domains
5. Deconvolution with a matched single-cell RNA-seq reference
6. Composition-aware comparison across conditions

Both ecosystems handle Visium just fine. In R you've got Seurat with
`Load10X_Spatial`, and in Python you've got scanpy plus squidpy with
`sc.read_visium`. Honestly the tool choice is close to a wash here. Where it stops
being a wash is Xenium, which is next.
