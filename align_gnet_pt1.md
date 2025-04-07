# Aligning Data

***

In this lesson:
 * A brief background on alignment
 * Creating an index for alignment
 * Start aligning fastq files


***

## What is Alignment?

The typical Illumina HiSeq type run can produce 100s of millions of reads, usually from 50-150bp long.  These reads are from the ends of DNA fragments which are typically 400-600bp long.  These fragments either came directly from harvesting genomic DNA, or the conversion of RNA from cells into cDNA.

Alignment is the task of figuring out where these reads, and consequently the fragments they were sequenced from, originated in the original cells' genome.  This is the gateway step for many types of analyses, from ChIP-seq, RNA-seq, variant calling, and more specialized types of <something>-seq.

### Complications

The task of locating these reads in the genome faces a number of difficulties, major ones include:

* Sequencing errors - a sequencing quality score of 30 corresponds to an estimated error of 1 in 1000, q40 1 in 10000.  A typical run has average quality scores in the mid 30s, but over millions of reads statisically they will crop up.

* Reference genome - you can only align as well as your reference genome is accurate.  More polished references like human or mouse will have less issues than less studied organisms.

* Natural variantion - individual organisms will have SNPs and indels, the later of which can be particularly problematic.

* Computational power - trying to match millions of reads to a full genome is a very large task.


### Indexing

Most alignment algorithms use some form of indexing - essentially a way to quickly 'bookmark' sections of a genome.  

A commonly used technique is the Burrows-Wheeler algorithm.  This algorithm is used in lossless compression.  It is particularly good at finding reoccurring substrings in text and reducing the data to simpler representions.  It was adapted for use in alignment, but essentially reversing its usual usage.  The *index* created by B-W aligners is a reduced representation of the genome, which the program can quickly check parts of a read against.  

Burrows-Wheeler is used in a number of common alignment programs, like BWA and Bowtie (the B and W reference the algorithm name).  

However, we'll be using bbmap, which has a slightly different algorithm which rapidly finds k-mer matches.  But it too needs to create an index, which is used in a similar manner.  

These days, which aligner you use will probably depend more on external factors than "how good" a particular aligner is.  Nearly all the major ones differ in various comparisons by a few percent.  And this is a good time to introduce a bioinformatics rule of thumb:

*Most algorithms tend to agree on the obvious stuff and tend to only differ in edge cases*

Now, those edge cases may be important to your particular experiment, so that is one reason you find certain analyses using one aligner over another.  Another major reason is simply a lab standardized on the aligner the first person in the lab used to align sequencing data. You may also want to use the same aligner used in a paper that does the same or similar experiment you are conducting if you wish to compare results.





***

## Data Organization

As we've seen, things can get quite messy once you start processing and analyzing a lot of files.  Once you get more comfortable with paths you can start to have your commands and programs access data from different directories and write out results to a different set of directories.  For now we'll just continue to use the approach of accessing and writing to the current directory, then sort out files afterwards.

If you recall, the fastq files we produced last lesson were all in the `trim` directory, and no separation of the raw fastq files from the trimmed ones.  How you organize your data is up to you, though remember someone else may need to reconstruct your analysis later - that someone may even be you a few months or years in the future.
  
Usually it's a good idea to keep the files from each stage of processing in their own directories, for example you may have several directories like:
  
~~~
fastq/
fastqc/
trimmed/
align/
~~~

You could have it all organized from the beginning, using relative paths to have each stage (QC, trimming, aligning, counting) write directly to the relevant directories.  Or at each stage you just access files in the current "working" directory and then move all those files to a new directory.    

*(we're not actually doing this, it's example/hypothetical set of commands)*

Imagine we have all our raw fastq files in our current directory, and then we trimmed them, and wanted to put the trimmed versions in a new directory and we included the string "trim" in each of the resulting trimmed fastqs.

~~~
$ mkdir trimmed
$ mv *trim*fastq.gz trimmed/
$ mv trimmed/ ..
$ cd ../trimmed/
~~~

In the last three commands, we move the new `trimmed` directory up one level and then enter it, and lastly make sure all the files we want are here.


***  

## Making a reference with BBmap

Let's pretend we have loaded so many modules we're worried there may be a conflict:

~~~
$ module load bowtie
$ module list
~~~

Now we'll clear all loaded modules:

~~~
$ module purge
$ module list
~~~

`module purge` removes all currently loaded modules.  We could also use `module unload` or `module remove` to remove a specific module.

Now let's cleanly load `bbmap`:

~~~
$ module load bbmap
$ module list
~~~

~~~
Currently Loaded Modules:
  1) samtools/1.9   2) pigz/2.3.4   3) bbmap/38.41
~~~

As might have noticed earlier, extra modules appeared in addition to `bbmap`.  The BBmap package makes use of other modules, and ITS has set up the system so these modules are loaded automatically.  Here's another reason for using the modules system, some packages may depend on very specific versions of other packages.  Samtools is a module we'll be using later - it has a lot of utilities for working with alignment files.  `pigz` is module that let's the BBmap programs access compressed files.


Making a reference is fairly straight forward with `bbmap`.

First let's get a genome to use - usually these aren't just sitting around but we'll grab one from a set we maintain on longleaf.

~~~
$ cd /work/users/t/r/tristand/
$ mkdir align
$ cd align/
$ cp /proj/seq/data/dm6_UCSC/Sequence/WholeGenomeFasta/genome.fa .
~~~

`genome.fa` is a bit generic of a name, but as we'll see in a bit, the set of genomic data we store in /proj/seq/data follows a structure to make it easy to automate access.  We can rename this file to remind us of what it is.
  
~~~
$ mv genome.fa dm6_UCSC.genome.fa
~~~
  
We can look at bbmap's help to get a basic idea of its use:
(here we are piping the output to `less` to make it easier to look over)

~~~
$ bbmap.sh -h | less
~~~

From the help, we can see the instructions for creating an index are:  
`To index:     bbmap.sh ref=<reference fasta>`

So, we can construction this slurm submission:

~~~
$ sbatch -o dm6_indexing.out --wrap="bbmap.sh ref=dm6_UCSC.genome.fa"
~~~

As we saw with its bbduk trimming sister program, bbmap provides useful information in its logging:

~~~
$ less dm6_indexing.out
~~~

BBmap will by default write its indexing files to a directory called `ref` in the current directory.  In fact, when aligning, if you don't specify a reference, bbmap will look for the `ref` by default.  Because I've setup the alignment script to point to a directory containing a `ref` directory, let's move the index we just made into a newly made directory.  Why we do this will hopefully become clear later.

~~~
$ mkdir index_dm6
$ mv ref/ index_dm6/
~~~

***

## Premade references

Since indexes for each reference genome only need to be made once, we've provided a number of common reference genomes for some of the most commonly used aligners.  In fact that's what we used to get our alignments going.

~~~
$ ls /proj/seq/data/
~~~

Let's look in the `dm6_UCSC` directory, which is a build of the Drosophila melanogaster genome.

~~~
$ ls /proj/seq/data/GRCh38_NCBI/
~~~

That `Sequence` directory looks promising

~~~
$ ls /proj/seq/data/GRCh38_NCBI/Sequence/
~~~

Here, we've provided the premade indexes for several common alignment programs

~~~
$ ls /proj/seq/data/GRCh38_NCBI/Sequence/BBMapIndex/
~~~

Now we see the `ref` directory that BBmap likes to see!  We can give the above path to BBmap and it will automatically find the `ref` directory there.

***

## Alignment

***


Since the queues may or may not be congested, we'll go ahead and get some alignment jobs running, so we can hopefully see the results by the end of the lesson.  Later in the lesson, we'll go over in detail what we're doing here:

These files will be bigger than the ones we've worked with before, so let's work in our individual scratch spaces


Remember to use your own info instead of 't/r/tristand'.

~~~
$ cp /proj/seq/data/carpentry/scripts/s_gnet_bbmap.sh .
$ ls
~~~

We're going to build up a bit to submiting two alignment runs.  This is roughly how I might build up a for loop to submit a batch of runs.  Here, we'll use basename, that we saw before, but only giving it a file path, it's default behavior is to strip the path up to just the filename.

~~~
$ fq_path=/proj/seq/data/RNAseq-training/reads/full
$ ls $fq_path
$ for fq_r1_path in $(ls -1 $fq_path/Gm10*R1*fastq.gz); do echo $(basename $fq_r1_path); done
~~~

Echo shows us to files. 

~~~
Gm10847_R1.fastq.gz
Gm10851_R1.fastq.gz
~~~

Let's refine the loop a bit more:

~~~
$ for fq_r1_path in $(ls -1 $fq_path/Gm10*R1*fastq.gz); do fq_r1=$(basename $fq_r1_path); echo $(basename $fq_r1 _R1.fastq.gz); done
~~~

We assgined the fastq stripped of it's path to a variable, then used basename again to remove suffix.

~~~
Gm10847
Gm10851
~~~

~~~
$ refpath=/proj/seq/data/GRCh38_NCBI/Sequence/BBMapIndex/
$ for fq_r1_path in $(ls -1 $fq_path/Gm10*R1*fastq.gz); do fq_r1=$(basename $fq_r1_path); sample=$(basename $fq_r1 _R1.fastq.gz); sbatch -J $sample s_gnet_bbmap.sh $sample $refpath; done
~~~


A lot of these may take some time to queue, especially if we're all submitting at once.


***
