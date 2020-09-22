
Scratch is a special area where you can work with large files, but files are deleted after 36 days.  Let's navigate there using the `cd` command, which stands for *change directory*.  We'll dive in with a slightly complicated command, it requires a few user specific substitutions:

~~~
cd /pine/scr/<a>/<b>/<onyen>
~~~

Replacing `<a>` with the first letter of your onyen, `<b>` with the second letter of your onyen, and as before your onyen for `<onyen>`.  For me, the command is:

~~~
cd /pine/scr/t/r/tristand/
~~~

If your onyen was 'zjdown', the location of your scratch space would be:

`/pine/scr/z/j/zjdown`

Once there, use `pwd` to see your new location, which for my account is:


> ~~~
> /pine/scr/t/r/tristand
> ~~~

****

Often there are multiple ways to do something in the shell.  Say we wanted to navigate to the `untrimmed_fastq` directory, which is in the `shell_data` directory.  We could use two commands (but don't):

~~~
$ cd shell_data
$ cd untrimmed_fastq
~~~

or do it with one command, using `/` to separate each directory in the hierarchy we want to travel.

~~~
$ cd shell_data/untrimmed_fastq
~~~

What happens if we type `cd shell_data` from here? Try it and see what happens.

~~~
$ cd shell_data
~~~

> ~~~
> -bash: cd: shell_data: No such file or directory
> ~~~

The system looked for a directory or file called `shell_data` within the 
directory you were already in. It didn't know you wanted to look at a directory level
above the one you were located in.  

The shell isn't terribly smart, it can only do what its explicitly told, it can't infer much.

We can also chain together the special `..` notation for the next directory up

~~~
$ cd ../..
~~~

Where are we now?




## Examining the contents of other directories

By default, the `ls` commands lists the contents of the working
directory (i.e. the directory you are in). You can always find the
directory you are in using the `pwd` command. However, you can also
give `ls` the names of other directories to view. Navigate to the top of your scratch space.

Then enter the command:

~~~
$ ls shell_data
~~~

> ~~~
> sra_metadata  untrimmed_fastq
> ~~~

This will list the contents of the `shell_data` directory without you needing to navigate there.


****

> ## Listing practice
> 
> Try to list the contents of the `untrimmed_fastq` directory from the top of your scratch space. 

****

****

> > ## Solution
> >
> > ~~~
> > $ ls shell_data/untrimmed_fastq/
> > ~~~
> > 
> > > ~~~
> > SRR097977.fastq  SRR098026.fastq 
> > > ~~~
> > 


We're going to simplify a bit and work out of our home directory today, since the sample files we're using are fairly small.
So login as we did last week and we're going to recopy the sample data again - but we're doing this in our home directory and not the scratch space.  This will allow us to use the `~` shortcut more.

~~~
$ cp -r /proj/seq/data/carpentry/shell_data/ .
~~~

I've also added a few files to make some of the examples more clear and this will give us a clean copy


### Our data set: FASTQ files

Now that we know how to navigate around our directory structure, lets
start working with our sequencing files. We did a sequencing experiment and 
have two results files, which are stored in our `untrimmed_fastq` directory. 

> ## Exercise
> 
> 1. Print out the contents of the `~/shell_data/untrimmed_fastq/SRR097977.fastq` file. What is the last line of the file? 
> 2.  From your home directory, and without changing directories,
> use one short command to print the contents of all of the files in
> the `~/shell_data/untrimmed_fastq` directory.

****

****

****

> > ## Solution
> > 1. The last line of the file is `TC:CCC::CCCCCCCC<8?6A:C28C<608'&&&,'$`.
> > 2. `cat ~/shell_data/untrimmed_fastq/*`


> ## Nucleotide abbreviations
> 
> The four nucleotides that appear in DNA are abbreviated `A`, `C`, `T` and `G`. 
> Unknown nucleotides are represented with the letter `N`. An `N` appearing
> in a sequencing file represents a position where the sequencing machine was not able to 
> confidently determine the nucleotide in that position. You can think of an `N` as a `NULL` value
> within a DNA sequence. 
