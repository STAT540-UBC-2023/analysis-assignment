‘Downstream’ Analysis: from counts to Differential Expression
================
Your name here

**Reminder:** When answering the questions, it is not sufficient to
provide only code. Add explanatory text to interpret your results
throughout.

### Background

The dataset used for this assignment has been published by [Li et al. in
2016](https://journals.plos.org/plospathogens/article?id=10.1371/journal.ppat.1005511),
as well as included in a follow-up study by [Houston-Ludlam et al. also
in
2016](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0159197).
You should recognize the follow-up study, “Comparative Transcriptome
Profiling of Human Foreskin Fibroblasts Infected with the Sylvio and Y
Strains of Trypanosoma cruzi” as this is the paper you were asked to
review for the Paper Critique. The data used in this follow-up study was
a combination of newly generated data (for the Sylvio strain and one
replicate of Y strain), as well as the data generated in the original Li
et al. article (all for the Y strain). In this assignment, you’ll be
working with the Y strain data generated in the original article.

Although the processed data for this study are not available in GEO, the
*raw* RNA-Seq reads have been submitted to NCBI SRA under the accession
[SRP043008](https://trace.ncbi.nlm.nih.gov/Traces/sra/?study=SRP043008).
Fortunately, they have also been preprocessed using a standardized
pipeline from raw reads to mapped gene and transcript level counts, and
formatted into `RangedSummarizedExperiment` objects (ready for analysis
in R!) as part of the [Recount2
project](https://jhubiostatistics.shinyapps.io/recount/). The
`RangedSummarizedExperiment` experiment class functions much the same as
the `SummarizedExperiment` class we learned about in lecture. We can
read the data into R in this format using the [recount
package](https://bioconductor.org/packages/release/bioc/html/recount.html).

We can also get metadata (mapping of sample IDs to time points,
infection status, batch, etc) from [Table
S1](https://journals.plos.org/plospathogens/article/file?type=supplementary&id=info:doi/10.1371/journal.ppat.1005511.s011)
in the original study.

Some helper code to read the data in is provided.

### Load libraries

Note: only a few libraries are provided for the initial steps; add
additional libraries as needed for the rest of the analyses.

``` r
library(recount)
library(openxlsx)
library(tidyverse)
library(edgeR)
```

### Question 1: Importing the data and getting familiar with it (3 POINTS)

First, we’ll download and read the gene-level counts
`RangedSummarizedExperiment` object for project id “SRP043008” into R
using the `recount` package. We’ll add a column called `sample` to the
`colData` slot that contains the `run` variable (since this will serve
as our sample ID and is easier). And we’ll also add the sample IDs to
the colnames of the object. Note that there’s nothing you need to add
here.

``` r
download_study(project = "SRP043008")
```

    ## 2023-02-12 14:03:15 downloading file rse_gene.Rdata to SRP043008

``` r
load(file.path("SRP043008", "rse_gene.Rdata"))
colData(rse_gene)$sample <- colData(rse_gene)$run
colnames(rse_gene) <- colData(rse_gene)$sample
```

Now we’ll add sample IDs to the `colnames` of the object to the SRR
sample IDs: Use the IDs in the `run` variable of the `colData` slot.
Note that there’s nothing you need to add here.

``` r
colnames(rse_gene) <- colData(rse_gene)$sample
```

A. Investigate the object just loaded. How many genes are there? (0.5
pt)

``` r
# your code here
```

B. How many samples are there? (0.5 pt)

``` r
# your code here
```

Here, we convert the `RangedSummarizedExperiment` object into a
[`DGEList`](https://rdrr.io/bioc/edgeR/man/DGEList.html) object (for use
with `limma` and/or `edgeR`). As we discussed in lecture, this format is
similar to `SummarizedExperiment`, but has some special slots, including
one for normalization factors, that are important for methods like
`limma` and `edgeR`. We’ll include the `colData` of the
`RangedSummarizedExperiment` object as the `samples` data.frame in the
`DGEList`. Note there’s nothing you need to add here.

``` r
dge <- DGEList(counts = assays(rse_gene)$counts,
               samples = colData(rse_gene))
```

Next, we’ll read in the metadata from [Table
S1](https://journals.plos.org/plospathogens/article/file?type=supplementary&id=info:doi/10.1371/journal.ppat.1005511.s011)
in the original study. Note that this is an Excel file with some
nonstandard formatting, so this step needs to be done with care - the
`read.xlsx` package is suggested since this can skip some rows, and fill
in merged cells. Note there’s nothing you need to add here.

``` r
mdat <- read.xlsx("https://journals.plos.org/plospathogens/article/file?type=supplementary&id=info:doi/10.1371/journal.ppat.1005511.s011",
                  startRow = 3, fillMergedCells = TRUE) %>%
  mutate(sample=Accession.Number)
```

C. Subset the metadata to include only those samples that we have gene
counts for (Table S1 also includes the non-host (non-human) samples),
and add this information to the `samples` slot of the `DGEList` object
(the `samples` slot of the `DGEList` is where the sample metadata is
stored - analogous to `colData()` for a `SummarizedExperiment`). HINT:
perform a join operation by sample id (beginning with “SRR”). (1 pt)

``` r
# your code here
```

D. How many variables of interest are in our experimental design? Are
these factors (categorical) or continuous? If factors, how many levels
are there? List out the levels for each factor. (1 pt)

``` r
# your code here
```

### Question 2: Remove lowly expressed genes (2 POINTS)

A. Remove lowly expressed genes by retaining genes that have CPM $>$ 1
in at least 25% of samples. (1 pt)

``` r
# your code here
```

B. How many genes are there after filtering? (1 pt)

``` r
# your code here
```

### Question 3: Data wrangling (2 POINTS)

The different character values of `Developmental.stage` refer to time
points - these can be thought of as categorical or on a continous axis.
In order to make graphing easier, it will be helpful to convert this
variable to a numeric representation.

Create a new column in the samples metadata tibble. Call it “hpi” (which
stands for hours post infection) and populate it with the appropriate
numeric values.

``` r
# your code here
```

Remove the `eval=FALSE` tag in the next chunk so that you can check that
your added variable looks as expected.

``` r
# check the result
table(dge$samples$hpi, dge$samples$Developmental.stage)
class(dge$samples$hpi)
```

### Question 4: Assessing overall distributions (4 POINTS)

A. The expression values are raw counts. Calculate TMM normalization
factors (and add them to your `DGEList` object). (1 pt)

``` r
# your code here
```

B. Examine the distribution of gene expression on the scale of
$\sf{log_{2}}$ CPM across all samples using box plots (with samples on
the x-axis and expression on the y-axis). (1 pt)

- Hint 1: Add a small pseudo count of 1 before taking the log2
  transformation to avoid taking log of zero - you can do this by
  setting `log = TRUE` and `prior.count = 1` in the `cpm` function.
- Hint 2: To get the data in a format amenable to plotting with ggplot,
  some data manipulation steps are required. Take a look at the
  `pivot_longer` function in the tidyr package to first get the data in
  tidy format (with one row per gene and sample combination).

``` r
# your code here
```

C. Examine the distribution of gene expression in units of
$\sf{log_{2}}$ CPM across all samples using overlapping density plots
(with expression on the x-axis and density on the y-axis; with one line
per sample and lines coloured by sample). (1 pt)

``` r
# your code here
```

D. Which sample stands out as different, in terms of the distribution of
expression values, compared to the rest? (1 pt)

### Question 5: Single gene graphing (3 POINTS)

A. Find the expression profile for the gene *OAS* (Ensembl ID
ENSG00000089127). Make a scatterplot with (numeric) hours post-infection
(`hpi`) on the x-axis and expression value in log2(CPM + 1) on the
y-axis. Color the data points by infection status, and add in a
regression line for each one. (2 pt)

``` r
# your code here
```

B. Is there sign of interaction between infection status and hours post
infection for **OAS**? Explain using what you observed in your graph
from the previous question. (1 pt)

### Question 6: How do the samples correlate with one another? (4 POINTS)

A. Examine the correlation **between samples** using a heatmap with
samples on the x axis and the y axis, and colour indicating correlation
between each pair of samples. Again, use the log2 transformed CPM
values. Display batch (`Batch`), hours post infection (`hpi`), and
infection status (`Infected`) for each sample in the heatmap. (2 pt)

- Hint: Consider using `pheatmap()` with annotations and `cor` to
  correlate gene expression between each pair of samples.

``` r
# your code here
```

B. Among the variables `Batch`, `hpi`, and `Infected`, which one seems
to be most strongly correlated with clusters in gene expression data? (1
pt)

- Hint: Consider using ‘cluster_rows=TRUE’ in `pheatmap()`.

C. There is a sample whose expression values do not correlate as highly
with other samples of the same `hpi`, and in general. Identify this
sample by its ID. (1 pt)

### Question 7: Construct linear model for Differential expression analysis (4 POINTS)

A. First set up a model matrix with hours post infection (`hpi`),
infection status (`Infection`), and the interaction between them as
covariates. Then calculate variance weights with voom, and generate the
mean-variance trend plot. (2 pt)

- Hint: use the `DGEList` object as input to voom, since you want to
  input raw counts here (voom will automatically transform to log2 CPM
  internally).

``` r
# your code here
```

B. Use limma (`lmFit` and `eBayes`) to fit the linear model with the
model matrix you just created. (1 pt)

``` r
# your code here
```

C. Print the 10 top-ranked genes by adjusted p-value for the
`hpi:InfectedY` coefficient using `topTable` (1 pt)

``` r
# your code here
```

### Question 8: Interpret model (2 POINTS)

For the gene CDC20 (Ensembl id ENSG00000117399), what is the numeric
value of the coeffcient of the `hpi` term? What is the interpretation of
this value in terms of the effect of time on expression? Be sure to
include units and indicate which samples this applies to.

``` r
# your code here
```

### Question 9: Quantify the number of genes differentially expressed (3 POINTS)

Using the linear model defined above, determine the number of genes
differentially expressed by infection status *at any time point* at an
FDR (use adjust.method = “fdr” in `topTable`) less than 0.05.

- Hint: in other words, test the null hypothesis that all coefficients
  involving infection status are equal to zero.

``` r
# your code here
```

### Question 10: Interpret the interaction term (3 POINTS)

Explain what you are modeling with the interaction term. For a
particular gene, what does a signifcant interaction term mean?

### **Bonus Question** (2 POINTS - extra credit)

Compare your DE results to those obtained by [Li et
al. (2016)](https://journals.plos.org/plospathogens/article?id=10.1371/journal.ppat.1005511).
Discuss any discrepancies. List at least three explanations for these
discrepancies.
