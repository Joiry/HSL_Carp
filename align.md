# Aligning Data

***

In this lesson:
 * A brief background on alignment
 * Creating an index for alignment
 * Aligning fastq files
 * Post-alignment processing

***

## But First...

Since the queues may or may not be congested, we'll go ahead and get some alignment jobs running, so we can hopefully see the results by the end of the lesson.  Later in the lesson, we'll go over in detail what we're doing here:

These files will be bigger than the ones we've worked with before, so let's work in our individual scratch spaces

~~~
$ cd /pine/scr/t/r/tristand/
~~~

Remember to use your own info instead of 't/r/tristand'.  We'll now copy some data: 

~~~
$ cp -r /proj/seq/data/carpentry/align/ .
$ cd align/
$ ls
~~~

set a few variable, and submit two alignment runs.

~~~
$ module load bbmap
$ refpath=/proj/seq/data/dm6_UCSC/Sequence/BBMapIndex/
$ base=SRR5457511_1
$ sbatch -J $base slurm_bbmap_basic.sh $base $refpath
~~~

By using a variable for the "base", we can simply change the value of that variable and submit a second alignment with the same command"

~~~
$ base=SRR5457513_1
$ sbatch -J $base slurm_bbmap_basic.sh $base $refpath
~~~

A lot of these may take some time to queue, especially if we're all submitting at once.  Then a several minutes to run, so we'll go ahead and submit a few and look at the results once all have finished.

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
$ cp /proj/seq/data/dm6_UCSC/Sequence/WholeGenomeFasta/genome.fa .
~~~

'genome.fa' is a bit generic of a name, but as we'll see in a bit, the set of genomic data we store in /proj/seq/data follows a structure to make it easy to automate access.  We can rename this file to remind us of what it is.
  
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
$ ls /proj/seq/data/dm6_UCSC/
~~~

That `Sequence` directory looks promising

~~~
$ ls /proj/seq/data/dm6_UCSC/Sequence/
~~~

Here, we've provided the premade indexes for several common alignment programs

~~~
$ ls /proj/seq/data/dm6_UCSC/Sequence/BBMapIndex/
~~~

Now we see the `ref` directory that BBmap likes to see!  We can give the above path to BBmap and it will automatically find the `ref` directory there.

***

## Alignment

At the beginning of the lesson, we used a simple alignment script I made using `bbmap`, let's take a look at it:

~~~
$ less slurm_bbmap_basic.sh 
~~~

There's a few things to take note of:

~~~
base=$1
refpath=$2
~~~

I've written the script to take two arguments, once is a 'basename' of a file, and the other is the path to the reference index.

~~~
#SBATCH -n 4
...
threads=4
~~~

I request 4 nodes with the `-n` option for `sbatch`, and I've told `bbmap` to use 4 threads - these should be equal.  Typically I'd use 8 to 16 nodes (eg '-n 12' and 'threads=12') which will take longer to queue, but run faster overall

~~~
#SBATCH --mem 8g
...
-Xmx8g
~~~

We're going to request 8 gigs of memory from slurm, and you see below we're telling `bbmap` to use the same amount.  For mammalian sized genomes, 32g is usually a good choice.

~~~
in1=${base}.fastq.gz
~~~

`in1=<file_path>` tells `bbmap` what fastq file to use.  If we had paired end data in 2 separate files (which is how data is released by default), we'd have to specify the 'R2' file using `in2=`

`out=` specifies the name of the file we want to write the bam (aka alignment file) to.


***
*Wait for at least one run from everyone to finish*
***
  
The `bbmap` log files are very useful (not all aligners make such convenient summaries), it reports all sorts of info on how well the alignments we to standard out:  
*(note the job ID, the number after SRRxxxxxx, will differ for everyone, so try using tab completion)*

~~~
$ less bbmap.SRR5457511_1.57125268.err
~~~

That's a lot of info.  Notice anything in particular?

Now say you setup a lot of fastq files to be aligned overnight, and want a quick check to see how they did.  We can use `grep` to pull out specific lines from all the `.err` files at once:

~~~
$ grep "mapped" *err
~~~

When aligning paired end reads, the alignment information section will be repeated, once for Read 1, and Read 2, plus a combined section.  When dealing with paired end data aligned with 'bbmap', I find the following to be useful, because it shows how many of R1 and R2 mapped within typical fragment size of each other.
  
~~~
$ grep "mated pair" *err
~~~
  
***

## Samtools

Samtools is a package with a number of subcommands, which you use similar to the `module` command.

~~~
$ samtools
~~~

Shows us a ton of options, but its mostly important to notice this first part:

~~~
Usage:   samtools <command> [options]
~~~

Let's sort one of the bam files.  If we type in one of the subcommands, it'll give us a list of its individual options:

~~~
$ samtools sort
~~~

Take note of this one:

~~~
  -o FILE    Write final output to FILE rather than standard output
~~~

I mentioned in a previous class a lot of bioinformatics tools are designed to have their results piped from one to another.  So by default `samtools sort` is not going to save the sorted bam file, but stream it to standard out.  Instead, we want it in a file, and the `sort` subcommand gives us that option with the `-o` option.

So, to sort a bam you can submit the below to the queue:

~~~
$ sbatch -o bam_sort.out --wrap="samtools sort SRR5457511_1.bam -o SRR5457511_1.sorted.bam"
~~~

Once the sorting is finished, we have the file `SRR5457511_1.sorted.bam` - let's check the file sizes:

~~~
$ du -shc *bam
~~~

~~~
157M	SRR5457511_1.bam
106M	SRR5457511_1.sorted.bam
~~~

The sorted files will get a bit smaller - this is just because they make a little more efficient use of the file structure.

A lot of the programs that require sorted bams also need an index file of the bam.  Note, this is not the same sort of index as aligning needs - this is a more traditional index.

~~~
$ sbatch -o bam_index.out --wrap="samtools index SRR5457511_1.sorted.bam"
~~~

***  

## Practice

For homework(!) you can try sorting and indexing the other files.  Also think about how you might put these in scripts, or perhaps just one script to get all your aligning and sorting done in one submission to the queue.
