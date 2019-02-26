# QC and trimming of your data


## Modules

The Longleaf cluster uses a system called modules.  Unlike the "basic" commands of the shell, modules are used for more specific purpose software, often whole packages.

Why use modules?  Well, as we'll see, there's usually multiple versions of the same package available at once.



~~~
$ module
~~~

A lot of stuff scrolls by, and it doesn't look like the usual man page or other help info a command might provide.

`module` is a program that groups several functions, and you decide which you want to use with a subcommand:

`module <subcommand>'

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

Updates to packages may fix earlier bugs, improve an algorithm, or introduce new bugs or behaviors.  Its important to document which version you're using.
Sometimes you may do an analysis months or years after an initial analysis - you may not want to rerun the originals with a newer version of the software.

Also, if comparing to results from another lab's paper, you may want to use the same version of the program.

Go ahead and hit enter so we've loaded the `fastqc` module.

~~~
$ module list
~~~

~~~
Currently Loaded Modules:
  1) fastqc/0.11.7
~~~


Its important to note that the modules are only loaded into the current session, and that like shell variables are gone after logging out.  You can have modules automatically loaded when you login, but I prefer to always explicitly load the modules that I need.  You're less likely to be surprised by a version change or cross module conflict.

~~~
$ mkdir trim
$ cd trim
~~~
~~~
$ cp /proj/seq/data/carpentry/ecoli/ecoli_ref-5m.fastq.gz .
~~~

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

(note, you'd be using this command from a terminal window that is not logged into longleaf, ie operating on your own computer)

~~~
$ scp tristand@longleaf.unc.edu:/pine/scr/t/r/tristand/trim/ecoli_ref-5m_fastqc.html .
~~~

This operates like the `cp` command, only you're copying across a network, and you'll have to enter your onyen password.  You are essentially doing a temporary login, copying the files specified, and then logging out.  On Macs, using `pwd` and the native copy/paste is very useful if you have long paths you need to enter.  Note that by default `scp` begins in your home directory, so you need to supply it the correct relative path or give it an absolute path (which is usually safer, since you can't tab complete).

* Now we'll go over the FastQC report *

~~~
$
~~~
~~~
$
~~~

~~~
$
~~~
~~~
$
~~~
~~~
$
~~~
## Trimming

For this exercise, we're going to use the trimming provided in the `bbmap module`.  `bbmap` is also the module we'll be using for alignment.  There are a number of other trimming programs available, for example:

~~~
trim_galore/0.4.3
trimmomatic/0.36
~~~

BBmap's trimming utility is `bbduk`, and its primary purpose is to trim adapter content, which tends to be the major purpose of trimming nowadays.

~~~
$
~~~
~~~
$
~~~
~~~
$
~~~
~~~
$
~~~


## Re-QCing

Now, let's see what our trimming did.  We'll want to be specific, since we already ran `fastqc` on the original file:

~~~
$
~~~

Once its done, let's transfer it locally and compare the two.

~~~
$
~~~

## MultiQC

MultiQC is a program that aggregates multiple FastQC files into one report.  If you've got more than a just a few sequencing files, it can give you a quick overview of the major QC stats.  The individual FastQC reports still give better details, but looking at MultiQC, you can narrow down which fastq files you want to look at the QC stats in more detail.

