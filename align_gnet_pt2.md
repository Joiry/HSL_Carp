* Review the alignment slurm script
* Creating an index for alignment
* Examine some of the BBmap output
* Sorting the alignment files

## Alignment Slurm Script
Let's take a look at the bbmap slurm script:

~~~
$ less s_gnet_bbmap.sh 
~~~

There's a few things to take note of:

~~~
base=$1
refpath=$2
~~~

I've written the script to take two arguments, one is a 'basename' of a file, and the other is the path to the reference index.

~~~
#SBATCH -n 8
...
threads=8
~~~

I request 8 nodes with the `-n` option for `sbatch`, and I've told `bbmap` to use 8 threads - these should be equal.  Typically I'd use 8 to 16 nodes (eg '-n 12' and 'threads=12') which will take longer to queue, but run faster overall

~~~
#SBATCH --mem 32g
...
-Xmx32g
~~~

We're going to request 32 gigs of memory from slurm, and you see below we're telling `bbmap` to use the same amount.  For mammalian sized genomes, 32g is usually a good choice.

~~~
in1=$fq_path/${base}_R1.fastq.gz
in2=$fq_path/${base}_R2.fastq.gz
~~~

`in1=<file_path>` tells `bbmap` what fastq file to use for the first read.  Since this is paired end data in 2 separate files (which is how data is released by default), we specify the 'R2' file using `in2=`


`path=$refpath` the "path=" parameter points bbmap to an existing bbmap index.  If it is not set, bbmap looks for a 'ref/' directory in the current directory as mentioned above.

`out=` specifies the name of the file we want to write the bam (aka alignment file) to.


***

## Making a reference with BBmap

Making a reference is fairly straight forward with `bbmap`.

First let's get a genome to use - usually these aren't just sitting around but we'll grab one from a set we maintain on longleaf.

~~~
$ cd /work/users/t/r/tristand/
$ mkdir align
$ cd align/
$ cp /proj/seq/data/GRCh38_NCBI/Sequence/WholeGenomeFasta/genome.fa .
~~~

`genome.fa` is a bit generic of a name, but as we'll see in a bit, the set of genomic data we store in /proj/seq/data follows a structure to make it easy to automate access.  We can rename this file to remind us of what it is.
  
~~~
$ mv genome.fa GRCh38_NCBI.genome.fa
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
$ sbatch -o GRCh38_NCBI_indexing.out --wrap="bbmap.sh ref=GRCh38_NCBI.genome.fa"
~~~

As we saw with its bbduk trimming sister program, bbmap provides useful information in its logging:

~~~
$ less GRCh38_NCBI_indexing.out
~~~

BBmap will by default write its indexing files to a directory called `ref` in the current directory.  In fact, when aligning, if you don't specify a reference, bbmap will look for the `ref` by default.  Because I've setup the alignment script to point to a directory containing a `ref` directory, let's move the index we just made into a newly made directory.  Why we do this will hopefully become clear later.

~~~
$ mkdir index_GRCh38_NCBI
$ mv ref/ index_GRCh38_NCBI/
~~~

***

## BBmap alignment 

The `bbmap` log files are very useful (not all aligners make such convenient summaries), it reports all sorts of info on how well the alignments we to standard out:  
*(note the job ID, the number after Gmxxxxx, will differ for everyone, so try using tab completion)*

~~~
$ less bbmap.Gm10847.64297282.out
~~~

That's a lot of info.  Notice anything in particular?

Now say you setup a lot of fastq files to be aligned overnight, and want a quick check to see how they did.  We can use `grep` to pull out specific lines from all the `.err` files at once:

~~~
$ grep "mapped" *out
~~~

When aligning paired end reads, the alignment information section will be repeated, once for Read 1, and Read 2, plus a combined section.  When dealing with paired end data aligned with 'bbmap', I find the following to be useful, because it shows how many of R1 and R2 mapped within typical fragment size of each other.
  
~~~
$ grep "mated pair" *out
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
$ sbatch -o bam_sort.out --wrap="samtools sort Gm10847.bam -o Gm10847.sorted.bam"
~~~

Once the sorting is finished, we have the file `Gm10847.sorted.bam` - let's check the file sizes:

~~~
$ du -shc *bam
~~~

~~~
157M	Gm10847.bam
106M	Gm10847.sorted.bam
~~~

The sorted files will get a bit smaller - this is just because they make a little more efficient use of the file structure.

A lot of the programs that require sorted bams also need an index file of the bam.  Note, this is not the same sort of index as aligning needs - this is a more traditional index.

~~~
$ sbatch -o bam_index.out --wrap="samtools index Gm10847.sorted.bam"
~~~

***  
