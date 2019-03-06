# Deseq2 Analysis Basics

We're going to use an R package called DESeq2.  It has extensive documention here:

[DESeq2 at Bioconductor](https://bioconductor.org/packages/release/bioc/html/DESeq2.html)

The *vignette* is particularly useful, and is frequently updated.  You'll be learning a small subset of it today, which should give you a foundation to explore the extent of DESeq2's functionality.


## The basic steps in abstract

Before we get into the actual code we'll be using, here's a quick outline of the analysis:

1. Load DESeq2 R library
2. Read in Count Data
3. Read in Experiment Design
4. Combine Data & Design into DDS object
5. Run DESeq
6. Data Interpretation - the hard part



The two main pieces of data we need for steps 2 and 3:
 * a table of the counts themselves
 * the experiment design - essentially all the samples and what conditions we assign them (at a minimum control vs treated)

The counts we should be familiar with now, however let's look at the experimental design file we'll be using.


A lot of the work above is in loading the data into R, and then making sure they are formatted correctly.  Once this is done, running DESeq is fairly straightforward.

## Running R on Longleaf

As with other programs, we need to load the R module:

~~~
$ module load r
$ module list
~~~

~~~
Currently Loaded Modules:
  1) r/3.3.1
~~~

The main thing to note here is the module is lowercase `r` while the command to invoke is uppercase `R`

We're going to run R interactively, so we're going to use a different slurm command - `srun`

~~~
$ srun -p interact --pty R
~~~

We're asking for the interactive queue with `-p interact` and `--pty` actually indicates to run in interactive mode.  In theory we could let this run on the `general` queue, but the nodes in `interact` are specifically reserved for interactive jobs.

One of the things you'll notice is our prompt is now `>` instead of `$` - this is just the prompt style of R.

**We're no longer working in the shell, so the commands you learned don't apply anymore.  Instead we're using R commands.**


### Step 1: load DESeq2 library

Similar to the modules system, R doesn't have all its packages loaded at once.  An important difference is that adding R packages is adding function code (and thus memory usage) to the R instance.  In addition, many packages might use the name for functions which do wildly different things.

~~~
> library("DESeq2")
~~~

This looks fairly simple, partly because ITS Research Computing has already added many packages.  If you install R on your own laptop, you'll have to install the packages

## Local installs

** Note, we're not going to do any of this section - this is FYI for running R on your own computer**

We won't go much into installing R and the very useful RStudio on a MacOS or Windows computer.



Once running R locally (again, RStudio highly recommended)

~~~
> install.packages("BiocManager")
~~~

This installs the `BiocManager` package which manages other packages.  A lot of biostats and bioinformatics packages make use of it to make installing them easier.  Once this finishes - you may need to agree to some new packages being installed, you can install `DESeq2`

~~~
> BiocManager::install("DESeq2", version = "3.8")
~~~

The above syntax tells `R` to use `BiocManager`'s version of installing, and it queries its own repository for DESeq2.

You may also want to install the useful `ggplot2` package, which provides a lot of useful graphing capabilities.

~~~
> install.packages("ggplot2")
~~~

## Formatting data

Deseq wants data in a particular format, there are several ways to get to this format depending on how you generated it in the first place.  The steps we will follow are for the table format we produced with `featureCounts`


### Step 2: read in count data
~~~
> Counts_bbmap <- read.table("Gm_counts_all.txt", header = TRUE, row.names=1)
~~~


## Step 2a: remove unneeded columns
~~~
> CountTable <- Counts_bbmap[,6:15]
~~~

## Step 2b: truncate count headers to match experiment design names
~~~
> colnames(CountTable) <- sapply(colnames(CountTable), substr, 0,7)
~~~


## Step 3: read in experiment design
~~~
> sampleInfo <- read.csv("Gm_sampleInfo.txt")
~~~

## Step 3a: format for DDS object
~~~
> sampleInfo <- DataFrame(sampleInfo)
~~~



## Step 4: Make DDS object
~~~
> ddsFullCountTable <- DESeqDataSetFromMatrix(countData = CountTable, colData = sampleInfo, design = ~pheno)
~~~


## Step 5: Run Deseq2 analysis
~~~
> dds <- DESeq(ddsFullCountTable)
~~~


## convert results into more useable format
~~~
> res <- results(dds)
~~~


## write results to a file
~~~
> write.table(res, "Gm_result_table.txt", col.names=NA, sep="\t")
~~~





## Now for some exploration of the data -------------------------

# some plotting packages
library("gplots")
library("ggplot2")

# Let's make some subsets of the data that might be interesting to look at

# cutoff by present
~~~
> resPresent <- res[which(res$baseMean > 1),]
~~~


# cutoff by FoldChange
~~~
> resFoldCutoff <-res[which(abs(resPresent$log2FoldChange)>1 & !is.na(resPresent$log2FoldChange)),]
~~~


# cutoff by multiple testing adjusted p-value
~~~
> resPvalCutoff <-resPresent[which(resPresent$padj<0.05),]
~~~


# order by padj
~~~
> resOrdered <- resPvalCutoff[order(resPvalCutoff$padj),]
~~~


# write out the ordered Pval Cutoff results
~~~
> write.table(resOrdered, "export_significant_result_table.txt", col.names=NA, sep="\t")
~~~




## some graphing -----------------------------------

# basic MA-plot

plotMA(res, alpha =0.05,  main="MA-plot\nfemale vs male", ylim=c(-10,10))


# volcano plot

plot(res$log2FoldChange, -log(res$padj, 10), xlim = c(-10,10), pch = 20, col = "#00000040", main = "Volcano Plot\nfemle vs male",xlab = "log2(fold change)", ylab = "-log10(adjusted p-value)")

# color in resOrdered subset

points(resOrdered$log2FoldChange, -log(resOrdered$padj, 10), pch = 20, col = "red")

# add some reference lines

abline(h=-log(0.05, 10), lty=2, col = "red")

abline(v=0, lty=2)

