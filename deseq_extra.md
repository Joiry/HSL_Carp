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

## Using R Studio

download history and demostrate

## OnDemand with Longleaf

[OnDemand at Research Computing](https://its.unc.edu/research-computing/ondemand/)

## PCA plots

~~~
> rld <- rlog(dds, blind=FALSE)
> png('Gm_pca.png')
> plotPCA(rld, intgroup=c("pheno", "run"))
> dev.off()
~~~

`rlog` - 'Regularized Log' transforms the data to Log2 scale
`plotPCA` - does the PCA plotting

## confounding/co-factor variables

new sample info file

## Heat Maps

code from workbook

## Functions

functionalizing things

~~~
>
~~~
