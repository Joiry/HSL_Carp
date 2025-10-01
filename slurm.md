## Introduction to Slurm

**Slurm** is a job scheduling system.  You pass slurm a command or shell script, and it will put enter this job into its queue.  Slurm manages the compute nodes on the cluster, farming out jobs as resources become available.  The resources are cpus and memory, which are some of the parameters you pass to slurm.  For today's lesson, we'll be using most of slurm's defaults, but once we get into bioinformatics tools in the next set of lessons, we'll need to request more compute and memory resources for the jobs to complete in a reasonable amount of time.


### Compressed files
Usually data is deliver in compressed files, with a `.gz` extension.  
Its usually fine to keep these compressed, because most bioinformatics tools can now work with compressed files.  
Let's compress these as an exercise in how to use slurm, the job scheduler

First, let's get some new data - you can do this in your home directory or scratch space.  These files are a bit larger, but should still fit in your home directory.

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

On an individual unix system you'd use a command like below (**don't execute this command**), but instead we should submit the command to the queue

`date; gzip Control_A.fastq; date`

Here I'd have used the date command and ';' to execute 3 commands, time stamping the compression duration

Imagining these files were much larger, we'd want to submit them to the queueing system.

The queueing system on Longleaf is called Slurm, and `sbatch` is the main way to submit a job to the system.
Most of the slurm commands start with 's'

~~~
$ sbatch --wrap="gzip Control_A.fastq"
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

There's also a shortcut for yourself:

~~~
$ squeue --me
~~~

**Warning**, if you type this, a lot of stuff will spew across your screen:

~~~
$ squeue
~~~

You can also check specific partitions:
(again, this is still likely to fill your screen)

~~~
$ squeue -p general
~~~

We can get a briefer summary of how full the various partitions are with this command:

~~~
$ sinfo
~~~


****

We can also submit a series of commands:

~~~
$ sbatch --wrap="date; gzip Control_B.fastq; date"
~~~

Let's look at what's in our directory now

~~~
$ ls
~~~

We can also look at file sizes, to see how the compression is doing:

~~~
$ du -shc *
~~~

What are these slurm files, are they useful?  Let's take a look at them with `less`.

These are the default files that slurm writes any standard out or error data to, essentially redirecting the output to this file.


We can tell slurm to save the standard out and standard error to a specific filename.
Note we're adding in the -v option to gzip to get some additional output for logging purposes.  '-v' in many Unix commands tends to mean "verbose", that the program should output more info on what its doing than usual.

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

~~~
$ sbatch -o compress.%j.out -e compress.%j.err --wrap="gzip -v Exp_2.fastq"
~~~

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

Now that we've set the shell variable, we can use it:

~~~
$ sbatch -o $sample_nam.gzip.%j.out -e $sample_nam.gzip.%j.err --wrap="gzip -v $sample_nam.fastq"
~~~

Once finished, let's look at the resulting files:

~~~
$ ls -1
~~~

Note, I replaced "compress" with "gzip" - this isn't necessary, but I like to include what command, program or script I used in the name of the log files

## More on compressed files

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


This whole --wrap="<command you want to run>" may seem really awkward, and it is!

Slurm is really intended for the user to submit a slurm script to run jobs

Copy over these slurm scripts:

~~~
$ cp /proj/seq/data/carpentry/slurm_*sh .
~~~

Let's take a look at the first one we will use:

~~~
$ less slurm_gzip.sh
~~~

We can see options we've already used, plus some extra

There are some defaults that Research Computing has set, so no all the options need be specified:  

[Getting started on Longleaf](https://help.rc.unc.edu/getting-started-on-longleaf/)

[Longleaf Slurm examples/](https://help.rc.unc.edu/longleaf-cluster/)

~~~
$ sbatch slurm_gzip.sh Exp_011
~~~

We can use a pointless time wasting script to see a few extra slurm commands

~~~
$ sbatch slurm_time_waster.sh
~~~

Do an squeue -u <your_onyen> to see its job id as if you had run it in a previous session

Substitute the job id for <jobid> below

~~~
$ sacct -j <jobid> --format=User,JobID,MaxRSS,Start,End,Elapsed
~~~

This gives us info on a running job, or past jobs.  Look up the previous job ids and compare with the currently running one.

We can also stop a job with the `scancel` command:

~~~
$ scancel <jobid>
~~~

We can look up its info still with the sacct command above.


## Looping with slurm submissions

There are two ways you could introduce looping into your use of slurm.  One would be to put the loop inside the script you submit to slurm.  But be aware each time through the loop within that script will be run linearly within the slurm job.  Ideally, you want to submit multiple jobs.

Let's see how we can use the `slurm_gzip.sh` script with what we've learned about loops in the previous lesson.  

First, let's make a new copy of our practice files from the backup director we made.

~~~
$ cp -r backup/ loop_practice
$ ls
~~~

Go into the `loop_practice' directory.  We can either recopy the slurm script, or move it from the practice diretory here:

~~~
$ mv ../practice/slurm_gzip.sh .
~~~

Once you start making a lot of general purpose scripts, you may want to create a scripts directory at the top level of your home directory.  Thus you could easily access your scripts from anywhere with the path `~/scripts/my_script.sh`.

~~~
$ for fq_file in $(ls -1 *fastq); do echo $fq_file; sbatch slurm_gzip.sh $fq_file; done
~~~

The construction `$(ls -1 *fastq)` takes the output of listing all the files ending in 'fastq', listing them one per line, and uses it as the list to loop over.  

Here we use `echo` to get some feedback on all the files the loop is going through.  The sbatch commands sends each job submission to our screen right after the echo.

****

You've now learned all the Unix basics for doing bioinformatics analyses on UNC's compute clusters.  There is tons more Unix to learn, some of it more useful and some less, depending on your needs.  Even experienced Unix users tend to use only a small subset of all the possible commands and options the shell provides.












