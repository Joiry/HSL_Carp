# Peak Calling and Multiomic Comparisons

* Peak calling
* How to call peaks with MACS
* Comparing interval data Bedtools
* Visulization

* 
* 
* 

***

## Peak Calling

Peak calling is a technique used for various types of DNA sequencing where specific areas of the genome have been isolated.

Some examples:

* [ChIP-Seq](https://en.wikipedia.org/wiki/ChIP_sequencing) - regions isolated by use of antibodies targeting proteins in DNA binding complexes

* [FAIRE-Seq](https://en.wikipedia.org/wiki/FAIRE-Seq) - use of formaldehyde to cross-link open areas of chromatin

* [ATAC-Seq](https://en.wikipedia.org/wiki/ATAC-seq) - uses a transposase to insert sequencing adapters in open chromatin


As we can see in the [Workflow Overview](https://joiry.github.io/HSL_Carp/workflow), the preceding steps for these types of NNN-Seq data also follow the common early steps of QC and trimming, and then alignment to the reference genome.  Note, for these sorts of targeted DNA sequencing, we must always align to the full genome and not a subset like only the transcriptome.

I've already aligned a set of FAIRE-Seq data for us to use.  These are big, but not too large, so we'd best work in our /pine/scr/ space for these



(note, I've added in some annotation on what each of these are):

~~~
SRR5457501_1.bam      # 3LW
SRR5457502_1.bam      # 3LW
SRR5457503_1.bam      # 3LW
SRR5457504_1.bam      # 24hAPF
SRR5457505_1.bam      # 24hAPF
SRR5457506_1.bam      # 24hAPF
SRR5457507_1.bam      # 44hAPF
SRR5457508_1.bam      # 44hAPF
SRR5457509_1.bam      # 44hAPF
~~~


[MACS project page](https://github.com/macs3-project/MACS)



***  

## Using MACS2



~~~
$ module load macs
$ 
~~~


There are four scripts we're going to use.  The first is a general purpose script that can call peaks on each one of our samples.

~~~
$ 
$ 
~~~

The next three combine the three replicates of each condition to allow for more data to call the peaks.  This can be useful at times, but isn't always necessary.  This is another decision that basically has to be made depending on the system and how the data shakes out.

~~~
$ 
$ 
~~~


***

## Bedtools

[Bedtools](https://bedtools.readthedocs.io/en/latest/) is a powerful suite of programs that can perform various comparisons with interval data, typicall bed format files, but also can work with BAM files and other genomic formats.

## Visualizing data with IGV


[Integrative Genomics Viewer IGV](https://software.broadinstitute.org/software/igv/)
