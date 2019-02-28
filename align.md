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

I've made a simple alignment script using `bbmap`, let's copy it and take a look:

~~~
$ cp /proj/seq/data/carpentry/scripts/slurm_bbmap_basic.sh .
$ less slurm_bbmap_basic.sh 
~~~


~~~
$ sbatch slurm_bbmap_basic.sh ecoli_ref-5m index_ecoli/
~~~

(a lot of these make some time to queue, and then a few minutes to run, so we'll go ahead and submit a few and look at the results once all have finished)

Now the more aggressively trimmed file:

~~~
$ sbatch slurm_bbmap_basic.sh ecoli_ref-5m.trim_q20 index_ecoli/
~~~


Let's try using the premade index in `/proj/seq/data/Ecoli_K12_DH10B_ENSEMBL`

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



The `bbmap` log files are very useful, it reports all sorts of info on how well the alignments we to standard out:

~~~
$ less bbmap.14281627.err
~~~

That's a lot of info.

Now say you setup a lot of fastq files to be aligned overnight, and want a quick check to see how they did.  We can use `grep` to pull out specific lines from all the `.err` files at once:

~~~
$ grep "mated pair" *err
~~~

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

***  

## Practice

For homework(!) you can try sorting and indexing the other files.  Also think about how you might put these in scripts, or perhaps just one script to get all your aligning and sorting done in one submission to the queue.
