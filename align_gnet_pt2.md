If time allow let's take a look at the bbmap slurm script:

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
#SBATCH -n 8
...
threads=4
~~~

I request 8 nodes with the `-n` option for `sbatch`, and I've told `bbmap` to use 4 threads - these should be equal.  Typically I'd use 8 to 16 nodes (eg '-n 12' and 'threads=12') which will take longer to queue, but run faster overall

~~~
#SBATCH --mem 32g
...
-Xmx32g
~~~

We're going to request 32 gigs of memory from slurm, and you see below we're telling `bbmap` to use the same amount.  For mammalian sized genomes, 32g is usually a good choice.

~~~
in1=${base}.fastq.gz
~~~

`in1=<file_path>` tells `bbmap` what fastq file to use.  If we had paired end data in 2 separate files (which is how data is released by default), we'd have to specify the 'R2' file using `in2=`
  
`path=$refpath` the "path=" parameter points bbmap to an existing bbmap index.  If it is not set, bbmap looks for a 'ref/' directory in the current directory as mentioned above.

`out=` specifies the name of the file we want to write the bam (aka alignment file) to.
