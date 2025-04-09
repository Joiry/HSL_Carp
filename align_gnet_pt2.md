* Review the alignment slurm script
* Creating an index for alignment
* Examine some of the BBmap output

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

