# Differential Expression Analysis





## Motivation

The basic idea behind differential expression analysis is that sequencing of RNA fragments serves as a rough proxy for how much those genes are expressed.


## Aligned data

Alignment for DE can be to a whole reference sequence or just transcriptome  
* Transcriptome alignment
  * Faster – smaller target reference
  * Implicit filter of non-transcribed reads, if you trust your reference transcriptome
  * may not be as good with isoforms
* Whole genome alignment
  * Discover new transcripts not in transcriptome annotation
  * Time to align less of a factor nowadays



## How many Biological replicates

The more biological replicates you can manage, the more robust your analysis results will be.

The MODEncode project, which wrapped up data submissions ~2011-12 required 2 biological replicates.  Soon after 3 biological replicates became the recommendation, then 4.

[Schurch et al 2016](https://rnajournal.cshlp.org/content/22/6/839.short)

In this paper, 48 biologial replicates were sequenced both from WT yeast and a known mutant strain.  42 of each where used to perform a variety of analyses.  Unsurprisingly, the more replicates used, the more of the "full set" of changes were captured (here defined by using all the replicates).

The authors' take away was that at least 6 biological replicates be used, preferably 12.


## Which analysis software should you use

Like other areas of bioinfomatics, there is a large number of tools that have be written.

There are many papers out there comparing many of the tools (Schurch above also includes an more recent comparison of major tools).  Take a look at this Venn diagram of a comparison of three tools:

![Kvam Fig5A](/images/Kvam_2012_fig5A.png)

[Kvam 2012](https://bsapubs.onlinelibrary.wiley.com/doi/full/10.3732/ajb.1100340)

This is an older analysis, but the figure shows how all the tools agree on a subset of the data and differ on other parts.

IN the previous lesson, I mentioned how the differences between aligners tend to occur with edge case read alignments, but generally agree about reads that have a single highly likely alignment.  Similarly, DE tools tend to agree on genes that have the largest changes, and disagree on whether smaller expression level changes are significant.

***

## Getting gene counts

First, let's get some data to work on.  I've prepared a set of subdirectories within `project_Gm` with some pre-aligned data and other files we'll need.  The data is a bit big, so we'll want to work in our scratch space.  Once there:

~~~
$ cp -r /proj/seq/data/carpentry/project_Gm/ .
~~~


### Annotation files

When you align data, we're only matching each read to a location on the genome.  The reference genome and the alignment files, SAM or BAM, know nothing about where exons, protein binding sites, or other features exist in the genome.

This sort of meta-data is generally found in GFF or GTF files, which are tab delimited files that contain information on features in a genome, their chromosome, start and stop locations, and other information.

For all the genomes we provide in `/proj/seq/data/` there are corresponding annotation files.

~~~
$ cd /proj/seq/data/
~~~

Let's look in the `HG19_UCSC` directory:

~~~
$ cd HG19_UCSC
$ ls
~~~

~~~
Annotation    Homo_sapiens_UCSC_hg19.tar  README.txt
Homo_sapiens  NOTE                        Sequence
~~~

Recall the index files we use to align are in the `Sequence` directory.  The GTF files are in `Annotation` and its subdirectories, and for today's lesson we'll just be using the `Genes` annotations.

~~~
$ cd Annotation
$ ls
$ cd Genes
$ ls
~~~

For today's lesson, we'll be using `genes.gtf`, a detailed explanation of GTF files is here:

[GTF format](http://mblab.wustl.edu/GTF22.html)

Briefly, it takes the following form, tab delimited fields:

`<seqname> <source> <feature> <start> <end> <score> <strand> <frame> [attributes] [comments]`

Let's take a look at the one we'll be using.

~~~
$ less -S genes.gtf 
~~~

The `-S` is a useful option for `less` when viewing structured files, it disables line wrapping allowing us to view the data columns in the GTF file more clearly.


Just as a reminder, let's see where we are:

~~~
$ pwd
~~~

~~~
/proj/seq/data/HG19_UCSC/Annotation/Genes
~~~

And the full path to the GTF is:

~~~
/proj/seq/data/HG19_UCSC/Annotation/Genes/genes.gtf
~~~

We'll see this path in the script we use below.

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
featureCounts -a $gtf -o Gm_counts_all.txt -T 4 -g gene_id -p -s 2 $bam1 $bam2 $bam3 $bam4 $bam5 $bam6 $bam7 $bam8 $bam9 $bam10
~~~

`-p` tells `featureCounts` this is paired end data, if you don't use this option with paired data, the counts will be weird
`-s 2` indicates the sequencing libraries were reverse stranded.  This is standard from UNC HTSF, you may have to inquire from a sequencing facility which library kits they used.  `-s 0` indicates unstranded, and `-s 1` regular stranded.

If your read counts are low, try with `0`, as it will count in both directions, and then you can figure out if the library was actually in the other direction.

`-T 4` specifies to use 8 threads, which should have the same value as the `-n` for the slurm script

Finally, there is the list of paths to all the bams we want to count.  You can modify this script by changing the path/names of the variable assignments for the variables `bam1`, `bam2`, etc.

To run this script, we just `sbatch` it, since all values it needs is coded into the script itself

~~~
$ sbatch Gm_fcounts.slurm.sh
~~~

***
