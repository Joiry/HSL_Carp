# QC and trimming of data

***

In this lesson we'll learn the basics of doing *quality control* on sequencing data, and how to clean up data with low quality reads.  

First, though, we need to use the *modules* system on the compute cluster so we can access more complex applications.

***

## Modules

The Longleaf cluster uses a system called modules.  Unlike the "basic" commands of the shell, modules are used for more specific purpose software, often whole packages.  A package usually groups related programs and they may share a common set of conventions and functionality.

Why use modules?  Well, as we'll see, there's usually multiple versions of the same package available, and a module system let's us be specific about which version we want to use.

~~~
$ module
~~~

A lot of stuff scrolls by, and it doesn't look like the usual man page or other help info a command might provide.

`module` is a program that groups several functions, and you decide which you want to use with a subcommand:

`module <subcommand>`

Let's try this:

~~~
$ module avail
~~~

This subcommand lists out all the available modules, and conveniently pipes it through `less`

You see many of the modules have multiple versions, those with a (D) are the default version.  

'load' is the subcommand that will activate the module for use.  If you typed:

`module load <module_name>`

You would get the default version of that module.  Let's try loading a module, but don't hit <kbd>enter</kbd> quite yet:

~~~
$ module load fastqc<tab><tab>
~~~

By hitting tab, you can see the other versions available.  Why are these other versions there - the short answer is you should treat version numbers of software the same as you might lot numbers of reagents.

Updates to packages may fix earlier bugs, improve an algorithm, or introduce new bugs or behaviors.  **Its important to document which version you're using.**  

Sometimes you may do an analysis months or years after an initial analysis - you will have to decide if you want to rerun the originals with a newer version of the software, or run the newer data with the older version of the software.

Also, if comparing to results from another lab's paper, you may want to use the same version of the program - which is hopefully documented in their methods section or supplemental information.

Go ahead and hit enter so we've loaded the `fastqc` module.

~~~
$ module list
~~~

~~~
Currently Loaded Modules:
  1) fastqc/0.11.7
~~~

FastQC is a nice little program that will compile stats on the read quality of a fastq file and a few other general measures.  We'll look over the results later and talk about what each section of the report means.

Its important to note that the modules are only loaded into the current session, and that like shell variables are gone after logging out.  You can have modules automatically loaded when you login, but I prefer to always explicitly load the modules that I need.  You're less likely to be surprised by a version change or cross module conflict.

We can also see that a new path has been added to the `PATH` variable:

~~~
$ echo $PATH
~~~

***

## Using FastQC

Even though the following can be done in your home directory, we'll use the scratch space `/work/users/<a>/<b>/<onyen>` as if this was a much larger fastq file.

~~~
$ cd /work/users/t/r/tristand
~~~

We'll make a trimming folder and enter it.

~~~
$ mkdir trim
$ cd trim
~~~

Let's copy a test fastq, this is a file of E Coli reads, 5 million in fact.  Its a bit bigger than the previous files we worked with, but still smaller than a typical modern fastq (of course this depends on your experiment)

~~~
$ cp /proj/seq/data/carpentry/ecoli/ecoli_ref-5m.fastq.gz .
$ cp /proj/seq/data/carpentry/ecoli/adapters.fa .
~~~

***

Now we'll run `fastqc` on it.  

~~~
$ sbatch -o fqc.%j.out -e fqc.%j.err --wrap="fastqc *.fastq.gz"
~~~

Even though we only have a single file, this shows fastqc can take multiple inputs and using the wildcard lets us send all the files for analysis.

~~~
$ ls -1
~~~

You should see something like this, but with a different jobID for the `.err` and `.out` files.

~~~
ecoli_ref-5m_fastqc.html
ecoli_ref-5m_fastqc.zip
ecoli_ref-5m.fastq.gz
fqc.14117348.err
fqc.14117348.out
~~~

If we use less to look at the log files, we see `fastqc` uses standard error to print out its completion status in 5% increments, and standard out just to indicate when each individual file completes.

The two main result files are:

~~~
ecoli_ref-5m_fastqc.html
ecoli_ref-5m_fastqc.zip
~~~

Both contain essentially the same information.  The `.html` file is an embedded html file, meaning all the images are integrated into the one file, and you can transfer it around easily.  The `.zip` contains a more tradition html file with separate images and some extra files.

Let's transfer the `.html` file to our local computer and take a look at it.  Some terminal programs provide a GUI interface to do this, or you could use a program like Filezilla.  Also, if you have a native terminal, like on MacOS X, you could transfer it using scp

*(note, you'd be using this command from a terminal window that is not logged into longleaf, ie operating on your own computer)*

If working in your scratch space (replacing with your own onyen and scratch path:  
~~~
$ scp tristand@longleaf.unc.edu:/work/users/t/r/tristand/trim/ecoli_ref-5m_fastqc.html .
~~~

...or, somewhat simpler, from your home directory:

~~~
$ scp tristand@longleaf.unc.edu:trim/ecoli_ref-5m_fastqc.html .
~~~

This operates like the `cp` command, only you're copying across a network, and you'll have to enter your onyen password when requested.  You are essentially doing a temporary login, copying the files specified, and then logging out.  On Macs, using `pwd` and the native copy/paste is very useful if you have long paths you need to enter.  Note that by default `scp` begins in your home directory, so you need to supply it the correct relative path or give it an absolute path (which is usually safer, since you can't tab complete).

***

**Now we'll go over the FastQC report**

***  



## Trimming

For this exercise, we're going to use the trimming provided in the `bbmap module`.  `bbmap` is also the module we'll be using for alignment.  There are a number of other trimming programs available, for example:

~~~
trim_galore/0.4.3
trimmomatic/0.36
~~~

~~~
$ module load bbmap
~~~

BBmap's trimming utility is `bbduk`, and its primary purpose is to trim adapter content, which tends to be the major purpose of trimming nowadays.  However, as an exercise we'll use it to do simpler quality trimming first.

The simplest method is to set a hard quality cutoff.

~~~
$ sbatch -o bbduk.%j.out --wrap="bbduk.sh -Xmx1g in=ecoli_ref-5m.fastq.gz out=ecoli_ref-5m.trim_q20.fastq.gz qtrim=rl trimq=20"
~~~

Don't worry too much about the `-Xmx1g` for the moment, these are parameters for java, which the bbmap programs are written in.  Most important is `1g` tells java use 1 gig of memory.  As we'll see in the next lesson, this will become more important for bbmap's alignment program.

`in=` is the file you want to trim reads from 
`out=` specifies the name of the file to write the trimmed to
`qtrim=rl` indicates to trim from both the left and right ends.  `qtrim=r` would mean just trim from the right side.
`trimq=20` tells bbduk the cutoff, reads below quality 20 will be trimmed.

Its important to note the trimming proceeds from whichever side until it reaches a bp about the cutoff, and stops there.  Potentially one good read could protect worse reads further in, so another strategy is to use a window, where the cutoff is triggered based on the average quality of several reads.

*(in the case of bbduk, the basic side trimming is a bit more sophisticated already than other trimmers)*

~~~
$ sbatch -o bbduk.%j.out --wrap="bbduk.sh -Xmx1g in=ecoli_ref-5m.fastq.gz out=ecoli_ref-5m.trim_win20.fastq.gz qtrim=w trimq=20 minlen=50"
~~~


As stated above, the main need for trimming is to remove adapter content, and we tend to be less concerned with quality scores these days.  There are still cases where quality trimming is needed - for example transcriptome assembly.

~~~
$ sbatch -o bbduk.%j.out --wrap="bbduk.sh -Xmx1g in=ecoli_ref-5m.fastq.gz out=ecoli_ref-5m.trim.adapt.fastq.gz minlen=50 ref=adapters.fa ktrim=r k=23 mink=11 hdist=1 tpe tbo"
~~~

What do these new options mean?  Use bbduk's help to figure them out as a quick exercise:

~~~
$ bbduk.sh --help | less
~~~

We're in `less`, and if you recall, there's a way to search...

~~~
ref=adapters.fa
ktrim=r
k=23
mink=11
hdist=1
tpe
tbo
~~~

***

## Re-QCing

Now, let's see what our trimming did.  We'll want to be more specific, since we already ran `fastqc` on the original file.  By including 'trim' in all our trimmed fastq files, we have an easy filter:

~~~
$ sbatch -o fqc.%j.out -e fqc.%j.err --wrap="fastqc *trim*.fastq.gz"
~~~

Once its done, let's transfer it locally and compare what the different trimmings did.  

But first, this directory is getting a bit messy.

~~~
$ mkdir fastqc
$ mv *fastqc* fastqc
$ ls fastqc/
~~~

Notice the error?  This one is rather benign, since all the other files get moved, but we can't move a directory into itself.

If we go in this directory and use `pwd`, it can give us a useful path to help in copying the files to our local machine.

~~~
$ cd fastqc/
$ pwd
~~~

~~~
/work/users/t/r/tristand/trim/fastqc
~~~

If using `scp` from a local terminal, this helps me build a path to specify the download target.

*(remember, this is for use on your local terminal)*

From scratch space:

~~~
$ scp tristand@longleaf.unc.edu:/work/users/t/r/tristand/trim/fastqc/*.html .
~~~

or home directory:

~~~
$ scp tristand@longleaf.unc.edu:trim/fastqc/*html .
~~~

***

## MultiQC

MultiQC is a program that aggregates multiple FastQC files into one report.  If you've got more than a just a few sequencing files, it can give you a quick overview of the major QC stats.  The individual FastQC reports still give better details, but looking at MultiQC, you can narrow down which fastq files you want to look at the QC stats in more detail.


MultiQC looks for all the `*fastqc.zip` files in a directory to build up its report.

~~~
$ module load multiqc
$ sbatch -o multiqc.%j.out --wrap="multiqc fastqc/"
~~~

Once its finished, checking the queue, let's see what the logging info has to say:

(substituting in the proper JobID)
~~~
$ less multiqc.14274947.out
~~~

Now let's download the `multiqc_report.html` file and take a look at it.

In this case, we're looking at the same data in multiple trimming states.  But typically you'd fastqc an entire folder and run MultiQC on them all.  You can look at the combined report to get an overall view, and then decide which specific Fastqc reports you want to look at in detail.

***


