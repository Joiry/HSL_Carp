# Deseq2 Extras

* Using R Studio
* OnDemand with Longleaf
* PCA plots
* confounding/co-factor variables
* Heat Maps
* Functions

As a reminder, DESeq2 has extensive documention here:

[DESeq2 at Bioconductor](https://bioconductor.org/packages/release/bioc/html/DESeq2.html)

We'll dive into a few more areas covered in it, but no where near everything.



So, in the last lesson, we saved our session.  Let's look at how that works when you want to restart R at the top of your home directory:

~~~
$ module load r
$ srun -p interact --pty R
~~~

If we look around, we can't find any of the variables or data or we were using last lesson that got saved.  There is a load function, but the easier method is to restart R in the same directory you ran before, because R will automatically look for the two save files.

~~~
$ cd project_Gm/deseq/
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
[local] $ scp tristand@longleaf.unc.edu:/nas/longleaf/home/tristand/project_Gm/deseq/.R* .
~~~

## Using R Studio

I won't be going over the installation instructions at the end of the last episode.  Depending on which OS and which version, different R and RStudio downloads can have some compatibility issues that can take a while to iron out.  However, if you were not able to get all the bioconductor packages you need installed, don't worry, the next section will let you follow along.

## OnDemand with Longleaf

OnDemand is a service that ITS Research Computing runs to allow access to the cluster in a more graphical means.  However, since its essentially sending a constant stream of your virtual desktop on the cluster, it can at times be laggy and lower resolution than if you were running locally on your own computer.

[OnDemand at Research Computing](https://its.unc.edu/research-computing/ondemand/)



## PCA plots
PCA - [Principle Component Analysis](https://en.wikipedia.org/wiki/Principal_component_analysis) - this is a method of reducing high dimensionality data by finding a rotation, or set of axes, that captures the greatest set of differences within the data.


~~~
> rld <- rlog(dds, blind=FALSE)
> plotPCA(rld, intgroup=c("pheno", "run"))
~~~

`rlog` - 'Regularized Log' transforms the data to Log2 scale
`plotPCA` - does the PCA plotting

## confounding/co-factor variables

If using RStudio locally, download this alternate experiment design file:

~~~
[local] $scp tristand@longleaf.unc.edu:/proj/seq/data/carpentry/project_Gm/Gm_samples_2.txt .
~~~

or copy it to the working directory for OnDemand

~~~
$ cp /proj/seq/data/carpentry/project_Gm/Gm_samples_2.txt .
~~~

~~~
> si_2 <- read.csv("Gm_samples_2.txt")
> si_2
> si_2 <- DataFrame(si_2)
~~~

Our count data is already correctly formated from the earlier analysis, so we can just plug that in with our new sample info, with one change:

~~~
> ddsCT_2 <- DESeqDataSetFromMatrix(countData = CountTable, colData = si_2, design = ~ group + pheno)
~~~

Additional factors in your experiment design are simply listed, separated by a plus symbol.

**Import** it is the last factor on which the DE analysis is performed.

~~~
> dds2 <- DESeq(ddsCT_2)
> res2 <- results(dds2)
> summary(res2)
~~~

and we can compare to our original results:

~~~
> summary(res)
~~~

## Heat Maps

code from workbook

## Functions

functionalizing things


