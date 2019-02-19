


## Introduction to Slurm and compressed files ****

Usually data is deliver in compressed files, with .gz extension
Its fine to keep these compressed, because most bioinformatics tools now can work with compressed files
But let's compress these as an exercise in how to use slurm

first, let's back a backup of the data we downloaded, and create a copy to practice on

~~~
$ mkdir backup
$ mv *fastq backup
$ ls
~~~

now let's make a copy of that directory

~~~
$ cp -r backup/ practice
$ ls
~~~

let's compare the size of the two directories

~~~
$ du -shc *
~~~


du stands for disk usage, what are the options we used?

~~~
$ man du
~~~

let's look at all the files on directory down

~~~
$ du -shc */*
~~~

this is a fast, but not complete way to check all the files are fully copied

~~~
$ cd practice
~~~

these are small files, but we'll compress them using the queueing system
this is very important, never run anything processor intensive on the head nodes!

normally, don't do this, instead submit to a queue

date; gzip Control_A.fastq; date

here I've used the date command and ';' to execute 3 commands, time stamping the compression duration

imagine these files were much larger, we'd want to submit them to the queueing system
sbatch is the main way to submit a job
most of the slurm commands start with 's'

~~~
$ sbatch --wrap="gzip Control_B.fastq"
~~~

if you're really fast, you may even see this short job in the queue

Eg, I would type:

~~~
$ squeue -u tristand
~~~

~~~
$ squeue -u <your_onyen>
~~~

sbatch is the command to submit a job, and squeue let's us see what's in the queue
if you don't specify a user with -u, you'll see all the jobs from everyone currently running

warning, if you type this, a lot of stuff will spew across your screen:

~~~
$ squeue
~~~

let's look at what's in our directory now

~~~
$ ls
~~~

we can also look at file sizes, to see how the compression is doing:

~~~
$ du -shc *
~~~

what's this slurm file, is it useful?  Take a look in it

we can tell slurm to save the standard out and standard error to a specific filename
note we're adding in the -v option to gzip

~~~
$ sbatch -o compress.out -e compress.err --wrap="gzip -v Exp_1.fastq"
~~~

use the up arrow and your history to get to your previous squeue -u <your_onyen> quickly

now let's look at the resulting files

~~~
$ ls -1
~~~

use less to look at both compress.out and compress.err - which output stream is gzip reporting with?

slurm has some special characters, and we can make the log .out and .err log files a bit more unique 

sbatch -o compress.%j.out -e compress.%j.err --wrap="gzip -v Exp_2.fastq"

again, you can up arrow to get to your squeue -u command to try and catch the job running

~~~
$ ls -1
~~~

we've got new compress.* files, what is different?  If you managed to see your job running in the queue, what do you notice about the names for the log files?

This may all seem exciting now as your learn, but you see the tedium in your future?
Let's use shell variables to make things bit easier, and more useful

~~~
$ SAMPLE_NAME=Exp_10
$ echo $SAMPLE_NAME
~~~

now that we've set the shell variable, we can use it:

~~~
$ sbatch -o $SAMPLE_NAME.gzip.%j.out -e $SAMPLE_NAME.gzip.%j.err --wrap="gzip -v $SAMPLE_NAME.fastq"
~~~

# once finished, let's look at the resulting files:
~~~
$ ls -1
~~~

note, I replaced "compress" with "gzip" - this isn't necessary, but I like to include what command, program or script I used in the name of the log files


before moving to the next part of the slurm lesson, let's take a look at these compressed files
some commands and programs can decompress gzip files on the fly

~~~
$ less Control_A.fastq.gz
~~~

others cannot, let's make sure to pipe the output to less to not gum up our screens

~~~
$ cat Control_A.fastq.gz | less
~~~

this is the output of cat, which is binary data.  Why isn't less taking care of this?

there's a version of cat, zcat, which can read gzipped files

zcat Control_A.fastq.gz | less

does grep work on gzip files?

~~~
$ grep "CTGGTGGTACTNTCTGTGGCG" Control_A.fastq.gz
~~~

That's the first sequence in the file, no luck?
We need to pipe the output of zcat into grep

~~~
$ zcat Control_A.fastq.gz | grep "CTGGTGGTACTNTCTGTGGCG"
~~~


## How to really use Slurm


this whole --wrap="<command you want to run" may seem really awkward, and it is
slurm is really intended for the user to submit a slurm script to run jobs

copy over this slurm script from my scratch space

~~~
$ cp /pine/scr/t/r/tristand/slurm_gzip.sh .
~~~

let's take a look at it

~~~
$ less slurm_gzip.sh
~~~

we can see options we've already used, plus some extra

there are some defaults that Research Computing has set, so no all the options need be specified
https://help.unc.edu/help/getting-started-on-longleaf/
https://help.unc.edu/help/longleaf-slurm-examples/

~~~
$ sbatch slurm_gzip.sh Exp_011
~~~

let's use a pointless time wasting job to see a few extra slurm commands

~~~
$ cp /pine/scr/t/r/tristand/slurm_time_waster.sh .
~~~

do an squeue -u <your_onyen> to see its job id if you had run it in a previous session
substitute the job id for <jobid> below

~~~
$ sacct -j <jobid> --format=User,JobID,MaxRSS,Start,End,Elapsed
~~~

this gives us info on a running job, or past jobs

now let's kill this job

~~~
$ scancel <jobid>
~~~

we can look up its info still with the sacct command above








