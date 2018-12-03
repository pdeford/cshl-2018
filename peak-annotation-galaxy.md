# Annotating ChIP peaks in Galaxy

After proceeding through a ChIP-seq pipeline, one of your products will be a file with the peaks you identified from the data. Unless you are only interested in identifying a _de novo_ motif, you will probably want to annotate this peaks, or to combine these peaks with other features you may have. We will go through an example of generating some of the annotations.

## Get your data

We are going to load a set of peak files from a Data Library. In the top menu bar, go to **Shared Data** `>` **Data Libraries** `>` **ChIP-seq Annotation Tutorial** `>` **To History** `>` **as Datasets**.

This data was originally produced by the Encode project. It is derived from K562 cells, a myelogenous leukemia cell line. We have ChIP-seq peaks for three factors:
- CTCF (ENCSR000BPJ)
- FoxA1 (ENCSR819LHG)
- GATA2 (ENCSR000DKA)

As well as for three histone marks:
- H3K4me3 (ENCSR000DWD)
- H3K27me3 (ENCSR000EWB)
- H3K9ac (ENCSR000EVZ)

In the data library, there is also an annotation file containing the NIH RefSeq genes from the UCSC table browser. All of the data is mapped to `hg38`. After importing the data, I would recommend tagging each file with the factor targeted. Something like the following:

![](https://raw.githubusercontent.com/pdeford/cshl-2018/master/img/1-tagged-input.png)

## Annotate peaks with _closest_ gene

One task you may be interested in is discovering which genes are the most proximal to each of your ChIP peaks. For some peaks, this may be a gene that overlaps the ChIP peak. Others will be non-overlapping, but nearby. To do this, we will use some of `bedtools` suite of utilities.

`bedtools` provides a host of functionality for operating on files containing genomic intervals, such as `bed` formatted files. Here we are going to be using the **sort** and **closest** functions.

### Prepare your data

The basic operations we will be performing will involve a number of pairwise comparisons between files. These operations can be made much more efficient if the input files are sorted. `bedtools` provides an easy way to sort bed files first by chromosome, then by start and end positions.

1. Navigate to the **BED tools** `>` **SortBED** tool.
2. Use the Batch Mode/Multiple datasets selector to select all of our datasets.
3. `Execute`
4. Rename the files to show that they are sorted. E.g. `Sorted GATA2 Peaks` or `Sorted RefSeq Genes`

![](https://raw.githubusercontent.com/pdeford/cshl-2018/master/img/2-sortbed.png)

### Annotate the peaks

To identify which gene is the closest to each ChIP peak, we will use the **ClosestBED** tool. One important functionality of this tool is that it can also report the distance between each peak and the nearest gene. If the peak and gene overlap, the reported distance will be 0, otherwise the value is the absolute distance. If you knew the orientation of the binding sites, you could even determine if the gene was upstream or downstream of the peak/binding site. This value can be useful for downstream analysis so we will choose to include it.

1. **BED tools** `>` **ClosestBED**
2. **bed,bedgraph,gff,vcf file**: `Sorted GATA2 Peaks`
3. **Overlap with:**: `Sorted RefSeq Genes`
4. **In addition to the closest feature in B, report its distance to A as an extra column**: `Yes`
5. `Execute`
6. Rename the output `Closest Genes to GATA2 Peaks`

![](https://raw.githubusercontent.com/pdeford/cshl-2018/master/img/3-closestbed.png)

Repeat for **FOXA1** and **CTCF**.

### Visualization and Gene sets

At this stage we have identified the genes that are the most proximal to each of the peaks from our ChIP-seq experiments. One question we may be interested in is where these peaks occur relative the these genes. We can visualize this by plotting the distribution of distances.

1. Use the **Text Manipulation** `>` **Cut columns from a table (cut)** tool to keep column 13 from `Closest Genes to GATA2 Peaks`. Rename the output `GATA2 distance to nearest gene`
2. **Filter and Sort** `>` **Filter data on any column using simple expressions** using the expression `c1!=0` on `GATA2 distance to nearest gene`. Rename `Filtered GATA2 distance to nearest gene`
3. Create the histogram with **Graph/Display Data** `>` **Histogram w ggplot2**. Label the x-axis: _"Distance to nearest gene"_, and the y-axis: _"Log10 Frequency"_. Set the **Bin width for plotting** to 1000. Expand the **Advanced Options**:
  - **Plot counts or density**: `Plot normalized frequency on the y-axis`
  - **Data transformation**: `Log10(value+1) transform my data`
4. Repeat for **FOXA1** and **CTCF**.

![](https://raw.githubusercontent.com/pdeford/cshl-2018/master/img/6-histogram.png)
![](https://raw.githubusercontent.com/pdeford/cshl-2018/master/img/6b-histogram.png)
![](https://raw.githubusercontent.com/pdeford/cshl-2018/master/img/7-example plot.png)

Alternately, you might be interested in those genes that are nearby these ChIP peaks for other analyses. A list of gene names can be extracted, again using the **Cut columns from a table (cut)** tool, this time on the name column from `Closest Genes to GATA2 Peaks`.

## Annotate promoters with corresponding marks

At this stage we have annotated the ChIP peaks with their closest genes. Instead, you might be interested in looking at which ChIP signals correspond to specific regions of the genome. Here we will consider which marks specifically overlap gene promoters.

### Identifying promoters

There are many definitions for promoters. Some definitions rely on functional characterization of the genomic elements adjacent to the transcription start site of a gene, others take into account histone marks and their boundaries, as well as a variety of other definitions. Here we are going to use a naive approach that defines a promoter based on distance from the transcription start site. Specifically we will use 1 kb upstream to 500 bp downstream of the TSS.

1. **Operate on Genomic Intervals** `>` **Get flanks**
2. **Select data:** `Sorted RefSeq Genes`
3. **Offset:** `500`
4. **Length of the flanking region(s)**: `1500`
5. `Execute` and rename output `Gene Promoters`

![](https://raw.githubusercontent.com/pdeford/cshl-2018/master/img/8-getflanks.png)

### Intersect promoters with ChIP peaks

Here again we will use the **bedtools** utilities to identify how many ChIP peaks overlap each promoter. We will specify that we want to know how many peaks overlap each promoter, even if that value is 0.

1. **BED tools** `>` **Intersect intervals**
2. **File A to intersect with B**: `Gene Promoters`
3. **File(s) B to intersect with A**: Use the batch mode to select all of the sorted ChIP peaks.
4. **For each entry in A, report the number of overlaps with B.**: `Yes`.
5. `Execute` and rename the output files in the style `GATA2 counts in promoters`

![](https://raw.githubusercontent.com/pdeford/cshl-2018/master/img/9-intersect intervals.png)
![](https://raw.githubusercontent.com/pdeford/cshl-2018/master/img/9b-intersect intervals.png)


### Visualize

For each promoter, we have counted the number of overlaps with each ChIP dataset we are interested in. If we extract these values into a matrix, we can generate a heatmap, clustering the gene promoters by those with similar marks. In addition we could look at the patterns in these clusters to try to determine their epigenetic state, for example.

#### Generate a count matrix

1. **Text Manipulation** `>` **Multi-Join**
2. **File to join**: `GATA2 counts in promoters`
3. **add additional file**: the other `counts in promoters` files.
4. **Common key column**: `4`
5. **Column with values to preserve**: `7`
6. **Add header line to the output file**: `Yes`
7. **Ignore duplicated keys**: `Yes`
8. **Value to put in unpaired (empty) fields**: `-1`
9. `Execute` and rename `Promoter-peak count matrix`

![](https://raw.githubusercontent.com/pdeford/cshl-2018/master/img/10-multi-join.png)
![](https://raw.githubusercontent.com/pdeford/cshl-2018/master/img/10b-multi-join.png)

#### Create the heatmap
1. **Graph/Display Data** `>` **heatmap2**
2. **Coloring groups**: `White to blue`

![](https://raw.githubusercontent.com/pdeford/cshl-2018/master/img/11-heatmap2.png)
