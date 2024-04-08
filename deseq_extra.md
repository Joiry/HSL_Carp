# Deseq2 Extras

* Restarting R
* Using R Studio
* OnDemand with Longleaf
* PCA plots
* Multi-factor design
* Heat Maps
* Functions

As a reminder, DESeq2 has extensive documention here:

[DESeq2 vignette](https://bioconductor.org/packages/release/bioc/vignettes/DESeq2/inst/doc/DESeq2.html)

We'll dive into a few more areas covered in it, but no where near everything.

***  

## Restarting R

So, in the last lesson, we saved our session.  Let's look at how that works when you want to restart R at the top of your home directory:

~~~
$ module load r
$ srun -p interact --pty R
~~~

If we look around, we can't find any of the variables or data or we were using last lesson that got saved.  There is a load function, but the easier method is to restart R in the same directory you ran before, because R will automatically look for the two save files.

~~~
$ cd /work/users/t/r/tristand/project_dm/deseq
$ ls -la
~~~

We'll see amongst the files:

~~~
-rw-r--r-- 1 tristand its_employee_psx 12843105 Mar  7 13:18 .RData
-rw------- 1 tristand its_employee_psx      736 Mar  7 13:18 .Rhistory
~~~

If we now rerun R in the current directory, we'll have all the previous data and history loaded:

~~~
$ srun -p interact --pty R
~~~

We can find all those old data structures, but something is missing...

~~~
> summary(res)
> mcols(res)$description
~~~

Listing the currently loaded packages:

~~~
> (.packages())
~~~

The data was loaded from the saved session, but which packages were loaded are not saved.  So we have to reload DESeq2 if we want to continue with the analysis.

~~~
> library("DESeq2")
~~~

However, we're going to quit out of the terminal only access to R, and use RStudio.  If you run RStudio on your own computer, after doing the main analysis say on the cluster, we will need to grab those data files.

First let's see where we are:

~~~
$ pwd
~~~

You can either use a file transfer program like filezilla, or if a local terminal like OSX is available, with this construction:

~~~
[local] $ scp tristand@longleaf.unc.edu:/work/users/t/r/tristand/project_dm/deseq/.R* .
~~~

***  

## Using R Studio

I won't be going over the installation instructions at the end of the last episode.  Depending on which OS and which version, different R and RStudio downloads can have some compatibility issues that can take a while to iron out.  However, if you were not able to get all the bioconductor packages you need installed, don't worry, the next section will let you follow along.

***  

## OnDemand with Longleaf

OnDemand is a service that ITS Research Computing runs to allow access to the cluster in a more graphical means.  However, since its essentially sending a constant stream of your virtual desktop on the cluster, it can at times be laggy and lower resolution than if you were running locally on your own computer.

[OnDemand at Research Computing](https://ondemand.rc.unc.edu/pun/sys/dashboard)

***  

## PCA plots
Last lesson, we briefly looked at PCA - [Principle Component Analysis](https://en.wikipedia.org/wiki/Principal_component_analysis) - this is a method of reducing high dimensionality data by finding a rotation, or set of axes, that captures the greatest set of differences within the data.

When running Rstudio, we don't need to bracket our plot functions with any functions to write the image to a file:

~~~
> plotPCA(rld, intgroup=c("condition"))
> plotPCA(vsd, intgroup=c("condition"))
~~~

`rlog` - 'Regularized Log' transforms the data to Log2 scale
`plotPCA` - does the PCA plotting

***  

## Multi-factor design

If using RStudio locally, download this alternate experiment design file:

~~~
[local] $scp tristand@longleaf.unc.edu:/proj/seq/data/carpentry/deseq_dm/dm_samples_2.txt .
~~~

or copy it to the working directory for OnDemand

~~~
$ cp /proj/seq/data/carpentry/deseq_dm/dm_samples_2.txt .
~~~

Now, load in the new experiment design, take a look at it to confirm, and transform it into a data frame type object.

~~~
> si_2 <- read.csv("dm_samples_2.txt")
> si_2
> si_2 <- DataFrame(si_2)
~~~

Our count data is already correctly formated from the earlier analysis, so we can just plug that in with our new sample info, with one change:

~~~
> dds_2f <- DESeqDataSetFromMatrix(countData = CountTable, colData = si_2, design = ~ harvest + condition)
> dds_2f$condition <- relevel(dds_2f$condition, ref = "3LW")
~~~

Additional factors in your experiment design are simply listed, separated by a plus symbol.

**Import** it is the last factor on which the DE analysis is performed.

~~~
> dds_2f <- DESeq(dds_2f)
> res_2f <- results(dds_2f)
> summary(res_2f)
~~~

and we can compare to our original results:

~~~
> summary(res)
~~~

The basic PCA plot will not change with this extra factor, its actual a parallel look at the data than the DESeq algorithm, but it utilizes the dispersion calculations.

~~~
> rld_2f <- rlog(dds_2f, blind=FALSE)
> vsd_2f <- vst(dds_2f, blind=FALSE)
> plotPCA(rld_2f, intgroup=c("condition"))
> plotPCA(vsd_2f, intgroup=c("condition"))
~~~

`rlog` and `vst` are two methods of transforming the data to make it suitable for visualization.  `rlog` will take longer to run with large numbers of samples.  For more, we can look at the DESeq section describing these functions:

[Data Visualization](https://bioconductor.org/packages/release/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#data-transformations-and-visualization)

***  

## Heat Maps

We're going to need a few more packages for more sophisticated graphing than base R provides:

~~~
> library("ggplot2")
> library("RColorBrewer")
> library("pheatmap")
~~~

Now we can do some graphing.  Most of below is prepping the data into the right format.

~~~
> sampleDists <- dist(t(assay(vsd)))
> sampleDistMatrix <- as.matrix(sampleDists)
> rownames(sampleDistMatrix) <- paste(vsd$condition)
> colnames(sampleDistMatrix) <- NULL
> colors <- colorRampPalette( rev(brewer.pal(9, "Blues")) )(255)
> pheatmap(sampleDistMatrix,
         clustering_distance_rows=sampleDists,
         clustering_distance_cols=sampleDists,
         col=colors)
~~~

***  

## Functions

If you end up doing many analyses, either refinements of one data set or work on several projects over time, you may want to start thinking about making more modular code.  This is similar to some of the scripting I showed in previous lessons - allowing you to pass in a few variables to change which data the code is working on.  

We can take the heat map commands from above, and put them into a function:  

~~~
HeatMapSamples <- function(data,col1)
{
sampleDists <- dist(t(assay(data)))
sampleDistMatrix <- as.matrix(sampleDists)
rownames(sampleDistMatrix) <- paste(data[[col1]])
colnames(sampleDistMatrix) <- NULL
colors <- colorRampPalette( rev(brewer.pal(9, "Blues")) )(255)
pheatmap(sampleDistMatrix,
         clustering_distance_rows=sampleDists,
         clustering_distance_cols=sampleDists,
         col=colors)
}
~~~

I have omitted the usual R prompts, `>`, that are used to show each command being entered separately.  This is just for teaching purposes, and if you have code in text files, you'll often just paste it in blocks, and you don't want all those prompts in there.

The one major change we make is adding in generic variable names, which we'll pass into the function.  Note when accessing the columns in the data set, we're no longer using the `$` shortcut, but rather have to use the full canonical way to specify columns with double angle brakets `[[...]]`.  This is because R can't tell, when using `$` formulation that its looking for the name of the variable, or the value of the variable.  This may seem odd, given in Unix `$` specifies something as a variable.  But it just shows these are two different languages.

Now we can invoke the function much like the built in functions in R and the DESeq2 package:

~~~
> HeatMapSamples(vsd,"condition")
~~~

***
