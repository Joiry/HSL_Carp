# Aligning Data

**Alignment Slides**

## Data Organization

~~~
$ mkdir align
$ mv *fastq.gz align/
$ mv align ..
$ cd ../align/
$ ls -1
~~~

## Making a reference

Let's make sure `bbmap` is loaded:

~~~
$ module list
~~~

Making a reference is fairly straight forward with `bbmap`

First let's get a genome to use - usually these aren't sitting around.

~~~
$ cp /proj/seq/data/carpentry/ref_genome/ecoli_rel606.fasta .
~~~

BBmap will by default write its indexing files to a directory called `ref` in the current directory.  In fact, when aligning, if you don't specify a reference, it will look for the `ref` by default.

~~~
$ less ecoli_indexing.out
~~~

Because I've setup the alignment script to point to a directory containing a `ref` directory, let's move the index we just made into a newly made directory.  Why we do this will hopefully become clear below:

~~~
$ mkdir index_ecoli
$ mv ref/ index_ecoli/
~~~


## Premade references

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

## Alignment


~~~
$ cp /proj/seq/data/carpentry/scripts/slurm_bbmap_basic.sh .
$ less slurm_bbmap_basic.sh 
~~~


~~~
$ sbatch slurm_bbmap_basic.sh ecoli_ref-5m index_ecoli/
~~~

~~~
$ sbatch slurm_bbmap_basic.sh ecoli_ref-5m.trim_q20 index_ecoli/
~~~



Since we know the adapter trimmed fastq file is essentially the same as the untrimmed, let's try using the premade index in `/proj/seq/data/Ecoli_K12_DH10B_ENSEMBL`

To make things a little more clear, and show a possible step towards future automation, let's store the path to the index in a shell variable first:

~~~
$ ecoli_index=/proj/seq/data/Ecoli_K12_DH10B_ENSEMBL/Sequence/BBMapIndex/
$ echo $ecoli_index
~~~

~~~
$ sbatch slurm_bbmap_basic.sh ecoli_ref-5m.trim.adapt $ecoli_index
~~~

~~~
$ grep "mated pair" *err
~~~

## Samtools


~~~
$
~~~

~~~
$
~~~
