# Differential Expression Analysis





## Motivation

The basic idea behind differential expression analysis is that sequencing of RNA fragments serves as a rough proxy for how much those genes are expressed.

Alignment can be to whole reference sequence or just transcriptome
Transcriptome alignment
Faster – smaller target reference
Implicit filter of non-transcribed reads, if you trust your reference transcriptome
Whole genome alignment
Discover new transcripts not in transcriptome annotation



## How many Biological replicates

[Schurch et al 2016](https://rnajournal.cshlp.org/content/22/6/839.short)


## Which analysis software should you use

![Kvam Fig5A](/images/Kvam_2012_fig5A.png)

## Getting counts

~~~
$ cp -r /proj/seq/data/carpentry/project_Gm/ .
~~~

### GFF files

review data location

/proj/seq/data/HG19_UCSC/Annotation/Genes/genes.gtf

### featureCounts

There are a number of counting programs.  We're going to use `featureCounts` which is in the `subread` package of tools.

`featureCounts` will make counts from multiple alignment files and put them in a table format which is close to the format that the DESeq uses.  Other counting programs may do file by file, and thus you'd need to load and combine each of these in R.  There are advantages to doing it that way if you plan to perform the analysis several times, comparing subsets of the data.

However, to keep things simple, we'll use the format `featureCounts` uses.

~~~
$ module load subread
$ module list
~~~

~~~
Currently Loaded Modules:
  1) subread/1.6.3
~~~

`subread` is an aligner, but the authors also made `featureCounts` as part of the same package.  So this is an example of a module that has a suite of programs within it.

However, `featureCounts` prints out its help information to standard error, which means its not trivial to pipe its usage info into pipe.

~~~
Usage: featureCounts [options] -a <annotation_file> -o <output_file> input_file1
 [input_file2] ...
~~~

At a minimum, these options are used:

`-a` this is the GFF or GTF file to tell `featureCounts` where the genes or other features are located within a genome
`-o` should be familiar, the name of file to write the counts.  This is just a text file.
`input_file1 [input_file2] ...` this is just a list of SAM or BAM files, eg `data1.bam data2.bam data3.bam`

However, in practice you're going to be using a few more options.  Here we have a slurm script for using tr

Let's take a look at this slurm script for running `featureCounts`

~~~
$ less Gm_fcounts.slurm.sh
~~~

~~~
featureCounts -a $gtf -o Gm_counts_all.txt -T 8 -g gene_id -p -s 2 $bam1 $bam2 $bam3 $bam4 $bam5 $bam6 $bam7 $bam8 $bam9 $bam10
~~~

`-p` tells `featureCounts` this is paired end data, if you don't use this option with paired data, the counts will be weird
`-s 2` indicates the sequencing libraries were reverse stranded.  This is standard from UNC HTSF, you may have to inquire from a sequencing facility which library kits they used.  `-s 0` indicates unstranded, and `-s 1` regular stranded.

If your read counts are low, try with `0`, as it will count in both directions, and then you can figure out if the library was actually in the other direction.

`-T 8` specifies to use 8 threads, which should have the same value as the `-n` for the slurm script
