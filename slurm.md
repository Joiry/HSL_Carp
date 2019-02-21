


## Introduction to Slurm and compressed files

Usually data is deliver in compressed files, with a `.gz` extension.  
Its usually fine to keep these compressed, because most bioinformatics tools now can work with compressed files
Let's compress these as an exercise in how to use slurm, the job scheduler

First, let's get some new data - you can do this in your home directory or scratch space.

~~~
$ cp -r /proj/seq/data/carpentry/data_exp .
$ cd data_exp/
$ ls
~~~

~~~
Control_A.fastq  Exp_011.fastq  Exp_1.fastq
Control_B.fastq  Exp_10.fastq   Exp_2.fastq
~~~

We'll make a backup directory and put the data in there.

~~~
$ mkdir backup
$ mv *fastq backup
$ ls
~~~

Now let's make a copy of that directory so we'll be free to practice on it.

~~~
$ cp -r backup/ practice
$ ls
~~~

We can compare the size of the two directories and their contents using `du`

~~~
$ du -shc *
~~~

'du' stands for disk usage, what are the options we used?

~~~
$ man du
~~~

Now look at all the files one directory level down.

~~~
$ du -shc */*
~~~

This is a fast, but not complete, way to check all the files are fully copied.

~~~
$ cd practice
~~~

These are small files, not typical of fastq data files you'll get from sequencing. But we will use them to demostrate both compressing files, and more importantly, using the queueing system.

This is very important, never run anything processor intensive on the head nodes!

On an individual unix system you'd use a command like below (don't do this), but instead we should submit the command to the queue

`date; gzip Control_A.fastq; date`

Here I'd have used the date command and ';' to execute 3 commands, time stamping the compression duration

Imagining these files were much larger, we'd want to submit them to the queueing system.

The queueing system on Longleaf is called Slurm, and `sbatch` is the main way to submit a job to the system.
Most of the slurm commands start with 's'

~~~
$ sbatch --wrap="gzip Control_B.fastq"
~~~

If you're really fast, you may even see this short job in the queue using `squeue`

Eg, I would type:

~~~
$ squeue -u tristand
~~~

But with your own onyen

~~~
$ squeue -u <your_onyen>
~~~

`sbatch` is the command to submit a job, and `squeue` let's us see what's in the queue
if you don't specify a user with -u, you'll see all the jobs from everyone currently running

Warning, if you type this, a lot of stuff will spew across your screen:

~~~
$ squeue
~~~

Let's look at what's in our directory now

~~~
$ ls
~~~

We can also look at file sizes, to see how the compression is doing:

~~~
$ du -shc *
~~~

What's this slurm file, is it useful?  Take a look in it

We can tell slurm to save the standard out and standard error to a specific filename
Note we're adding in the -v option to gzip to get some additional output for logging purposes.

~~~
$ sbatch -o compress.out -e compress.err --wrap="gzip -v Exp_1.fastq"
~~~

You can use the up arrow and your history to get to your previous squeue -u <your_onyen> quickly, possibly catching your job in the act of running.

Now let's look at the resulting files

~~~
$ ls -1
~~~

Use less to look at both compress.out and compress.err - which output stream is gzip reporting with?

Slurm has its own special notation, and we can make the log .out and .err log files a bit more unique 

sbatch -o compress.%j.out -e compress.%j.err --wrap="gzip -v Exp_2.fastq"

Again, you can up arrow to get to your squeue -u command to try and catch the job running

~~~
$ ls -1
~~~

We've got new compress.* files, what is different?  If you managed to see your job running in the queue, what do you notice about the names for the log files?

This may all seem exciting now as your learn, but you see the tedium in your future?
Let's use shell variables to make things bit easier, and more useful

~~~
$ sample_nam=Exp_10
$ echo $sample_nam
~~~

now that we've set the shell variable, we can use it:

~~~
$ sbatch -o $sample_nam.gzip.%j.out -e $sample_nam.gzip.%j.err --wrap="gzip -v $sample_nam.fastq"
~~~

# once finished, let's look at the resulting files:
~~~
$ ls -1
~~~

Note, I replaced "compress" with "gzip" - this isn't necessary, but I like to include what command, program or script I used in the name of the log files


Before moving to the next part of the slurm lesson, let's take a look at these compressed files  
Some commands and programs can decompress gzip files on the fly

~~~
$ less Control_A.fastq.gz
~~~

others cannot, let's make sure to pipe the output to less to not gum up our screens

~~~
$ cat Control_A.fastq.gz | less
~~~

This is the output of cat, which is binary data.  Why isn't less taking care of this?

There's a version of cat, zcat, which can read gzipped files

~~~
$ zcat Control_A.fastq.gz | less
~~~

Does grep work on gzip files?

~~~
$ grep "CTGGTGGTACTNTCTGTGGCG" Control_A.fastq.gz
~~~

That's the first sequence in the file, no luck?
We need to pipe the output of zcat into grep

~~~
$ zcat Control_A.fastq.gz | grep "CTGGTGGTACTNTCTGTGGCG"
~~~


## Slurm Scripts!


This whole --wrap="<command you want to run" may seem really awkward, and it is

slurm is really intended for the user to submit a slurm script to run jobs

Copy over this slurm script from my scratch space

~~~
$ cp /pine/scr/t/r/tristand/slurm_gzip.sh .
~~~

Let's take a look at it

~~~
$ less slurm_gzip.sh
~~~

We can see options we've already used, plus some extra

There are some defaults that Research Computing has set, so no all the options need be specified:  
https://help.unc.edu/help/getting-started-on-longleaf/

https://help.unc.edu/help/longleaf-slurm-examples/

~~~
$ sbatch slurm_gzip.sh Exp_011
~~~

Let's use a pointless time wasting job to see a few extra slurm commands

~~~
$ cp /pine/scr/t/r/tristand/slurm_time_waster.sh .
~~~

Do an squeue -u <your_onyen> to see its job id if you had run it in a previous session

Substitute the job id for <jobid> below

~~~
$ sacct -j <jobid> --format=User,JobID,MaxRSS,Start,End,Elapsed
~~~

This gives us info on a running job, or past jobs

Now let's kill this job

~~~
$ scancel <jobid>
~~~

We can look up its info still with the sacct command above








