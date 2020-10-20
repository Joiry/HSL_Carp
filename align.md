# Aligning Data

***

**Alignment Slides**

***

## Data Organization

As we've seen, things can get quite messy once you start processing and analyzing a lot of files.  Once you get more comfortable with paths you can start to have your commands and programs access data from different directories and write out results to a different set of directories.  For now we'll just continue to use the approach of accessing and writing to the current directory, then sort out files afterwards.

Let's look at the fastq files we produced last lesson in the `trim` directory, and then move all these files to a new directory.    
(normally we might just move only the trimmed ones to align, but we're going to use the original for our alignment practice and demostration)

~~~
$ mkdir align
$ mv *fastq.gz align/
$ mv align ..
$ cd ../align/
$ ls -1
~~~

In the last three commands, we move the new `align` directory up one level and then enter it, and lastly make sure all the files we want are here.


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

`purge` will remove all currently loaded modules.  We could also use `module unload` to remove a specific module.

Now let's cleanly reload `bbmap`:

~~~
$ module load bbmap
$ module list
~~~

~~~
Currently Loaded Modules:
  1) samtools/1.9   2) pigz/2.3.4   3) bbmap/38.41
~~~

As might have noticed earlier, extra modules have appeared in addition to `bbmap`.  The BBmap package makes use of other modules, and ITS has set up the system so these modules are loaded automatically.  Here's another reason for using the modules system, some packages may depend on very specific versions of other packages.  Samtools is a module we'll be using later - it has a lot of utilities for working with alignment files.  `pigz` is module that let's the BBmap programs access compressed files.


Making a reference is fairly straight forward with `bbmap`

First let's get a genome to use - usually these aren't just sitting around but I've made one available.

~~~
$ cp /proj/seq/data/carpentry/ref_genome/ecoli_rel606.fasta .
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
$ sbatch -o ecoli_indexing.out --wrap="bbmap.sh ref=ecoli_rel606.fasta"
~~~

As we saw with its bbduk trimming sister program, bbmap provides useful information in its logging:

~~~
$ less ecoli_indexing.out
~~~

BBmap will by default write its indexing files to a directory called `ref` in the current directory.  In fact, when aligning, if you don't specify a reference, bbmap will look for the `ref` by default.  Because I've setup the alignment script to point to a directory containing a `ref` directory, let's move the index we just made into a newly made directory.  Why we do this will hopefully become clear later.

~~~
$ mkdir index_ecoli
$ mv ref/ index_ecoli/
~~~

***

## Premade references

Since indexes for each reference genome only need to be made once, we've provided a number of common reference genomes for some of the most commonly used aligners.

~~~
$ ls /proj/seq/data/
~~~

Let's look in the `Ecoli_K12_DH10B_ENSEMBL` as an example.

~~~
$ ls /proj/seq/data/Ecoli_K12_DH10B_ENSEMBL/
~~~

That `Sequence` directory looks promising

~~~
$ ls /proj/seq/data/Ecoli_K12_DH10B_ENSEMBL/Sequence/
~~~

Here, we've provided the premade indexes for several common alignment programs

~~~
$ ls /proj/seq/data/Ecoli_K12_DH10B_ENSEMBL/Sequence/BBMapIndex/
~~~

Now we see the `ref` directory that BBmap likes to see!  We can give the above path to BBmap and it will automatically find the `ref` directory there.

***

## Alignment

I've made a simple alignment script using `bbmap`, let's copy it and take a look:

~~~
$ cp /proj/seq/data/carpentry/scripts/slurm_bbmap_basic.sh .
$ less slurm_bbmap_basic.sh 
~~~

There's a few things to take note of:

~~~
base=$1
refpath=$2
~~~

I've written the script to take two arguments, once is a 'basename' of a file, and the other is the path to the reference index.

~~~
#SBATCH -n 8
...
threads=8
~~~

I request 8 nodes with the `-n` option for `sbatch`, and I've told `bbmap` to use 8 threads - these should be equal.

~~~
#SBATCH --mem 32g
...
-Xmx32g
~~~

We're going to request 32 gigs of memory from slurm, and you see below we're telling `bbmap` to use the same amount.

~~~
in1=${base}.fastq.gz
~~~

`in1=<file_path>` tells `bbmap` what fastq file to use.  If we had paired end data in 2 separate files (which is how data is released by default), we'd have to specify the 'R2' file using `in2=`

`out=` specifies the name of the file we want to write the bam (aka alignment file) to.


***

Now let's run the script:

~~~
$ sbatch slurm_bbmap_basic.sh ecoli_ref-5m index_ecoli/
~~~

A lot of these may take some time to queue, especially if we're all submitting at once.  Then a few minutes to run, so we'll go ahead and submit a few and look at the results once all have finished.

Now one of the trimmed files:

~~~
$ sbatch slurm_bbmap_basic.sh ecoli_ref-5m.trim_q20 index_ecoli/
~~~


Next, we'll try using the premade index in `/proj/seq/data/Ecoli_K12_DH10B_ENSEMBL`

To make things a little more clear, and show a possible step towards future automation, let's store the path to the index in a shell variable first:

~~~
$ ecoli_index=/proj/seq/data/Ecoli_K12_DH10B_ENSEMBL/Sequence/BBMapIndex/
$ echo $ecoli_index
~~~

We'll copy the original read file, and test aligning it with the strain indexed in `/proj/seq/data`:

~~~
$ cp ecoli_ref-5m.fastq.gz ecoli_test.fastq.gz
$ sbatch slurm_bbmap_basic.sh ecoli_test $ecoli_index
~~~

*wait for at least one run from everyone to finish*

The `bbmap` log files are very useful (not all aligners make such convenient summaries), it reports all sorts of info on how well the alignments we to standard out:

~~~
$ less bbmap.14281627.err
~~~

That's a lot of info.  Notice anything in particular?

Now say you setup a lot of fastq files to be aligned overnight, and want a quick check to see how they did.  We can use `grep` to pull out specific lines from all the `.err` files at once:

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
$ sbatch -o bam_sort.out --wrap="samtools sort ecoli_ref-5m.bam -o ecoli_ref-5m.sorted.bam"
~~~

Once the sorting is finished, we have the file `ecoli_ref-5m.sorted.bam` - let's check the file sizes:

~~~
$ du -shc *bam
~~~

~~~
521M	ecoli_ref-5m.bam
...
513M	ecoli_test.bam
~~~

The sorted files will get a bit smaller - this is just because they make a little more efficient use of the file structure.

A lot of the programs that require sorted bams also need an index file of the bam.  Note, this is not the same sort of index as aligning needs - this is a more traditional index.

~~~
$ sbatch -o bam_index.out --wrap="samtools index ecoli_ref-5m.sorted.bam"
~~~

***  

## Practice

For homework(!) you can try sorting and indexing the other files.  Also think about how you might put these in scripts, or perhaps just one script to get all your aligning and sorting done in one submission to the queue.
