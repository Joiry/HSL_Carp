# Pipes and Variables

In this lesson:
 * we will learn about a powerful search tool
 * how to use redirection of streams
 * How to use and assign values to variables


## Searching files

We discussed in a previous lesson how to search within a file using `less`. We can also search within files without even opening them, using `grep`. `grep` is a command-line utility for searching plain text files for lines matching a specific set of characters (sometimes called a string) or a particular pattern (which can be specified using something called regular expressions). We're not going to work with regular expressions in this lesson, and instead will use specific strings or with shell wildcards.

Suppose we want to see how many samples in our meta data file are paired end data.

~~~
$ grep PAIRED SraRunTable.txt
~~~

> ~~~
> SAMN00205564	2834	PAIRED	ZDB30	29-May-14	1695	679	25-Mar-11SRR098033	SRS167172	ZDB30	WGS	<not provided>	PRJNA188723	MSU	public	Escherichia coli B str. REL606	ILLUMINA	SRP004752	<not provided>	<not provided>	<not provided>	REL606
> SAMN00205573	2729	PAIRED	ZDB172-PE	29-May-14	1620	635	25-Mar-11	SRR098043	SRS167181	ZDB172	WGS	<not provided>	PRJNA188723	MSU	public	Escherichia coli B str. REL606	ILLUMINA	SRP004752<not provided>	<not provided>	<not provided>	REL606
> ~~~

We can use the `-B` argument for grep to return a specific number of lines before each match and the `-A` argument to return a specific number of lines after each matching line. Here we want the line before and the line after each matching line so we add `-B1 -A1` to our grep command.

~~~
$ grep -B1 -A1 PAIRED SraRunTable.txt
~~~

One of the sets of lines returned by this command is: 

> ~~~
> SAMN00205564	0	SINGLE	<not provided>	29-May-14	311	257	25-Mar-11	SRR098032	SRS167172	ZDB30	WGS	<not provided>	PRJNA188723	MSU	public	Escherichia coli B str. REL606	ILLUMINA	SRP004752<not provided>	<not provided>	<not provided>	REL606
> SAMN00205564	2834	PAIRED	ZDB30	29-May-14	1695	679	25-Mar-11SRR098033	SRS167172	ZDB30	WGS	<not provided>	PRJNA188723	MSU	public	Escherichia coli B str. REL606	ILLUMINA	SRP004752	<not provided>	<not provided>	<not provided>	REL606
> SAMN00205565	0	SINGLE	ZDB83	29-May-14	260	162	25-Mar-11SRR098034	SRS167173	ZDB83	WGS	<not provided>	PRJNA188723	MSU	public	Escherichia coli B str. REL606	ILLUMINA	SRP004752	<not provided>	<not provided>	<not provided>	REL606
> ~~~




## Redirecting output

`grep` allowed us to identify specific line entries in our meta data file that matched a particular pattern.  All of these were printed to our terminal screen, but we may need to capture that output in some way instead of just printing to the screen. 

We can do this with something called "redirection". The idea is that
we are taking what would ordinarily be printed to the terminal screen and redirecting it to another location. 
In our case, we want to stream this information into a file so that we can look at it later and 
use other commands to analyze this data.

The operator for redirecting output to a file is `>`.

Let's try out this command and copy all the records in our SraRunTable.txt file that contain 'PAIRED' to another file called `paired_end_samples.txt`.

~~~
$ grep PAIRED SraRunTable.txt > paired_end_samples.txt
~~~

Type `ls`. You should see a new file called `paired_end_samples.txt`. 

We can check the number of lines in our new file using a command called `wc`.  `wc` stands for **word count**. This command counts the number of words, lines, and characters in a file. 

~~~
$ wc paired_end_samples.txt
~~~

> ~~~
>   2  62 465 paired_end_samples.txt
> ~~~

This will tell us the number of lines, words and characters in the file. If we want only the number of lines, we can use the `-l` flag for `lines`.

~~~
$ wc -l paired_end_samples.txt
~~~

> ~~~
> 2 paired_end_samples.txt
> ~~~


What if we might want to search multiple times in a file for our search pattern?
However, we need to be careful, because each time we use the `>` command to redirect output
to a file, the new output will replace the output that was already present in the file. 
This is called "overwriting" and

~~~
$ grep CZB1 SraRunTable.txt > special_samples.txt
$ wc -l special_samples.txt
~~~

> ~~~
> 3 special_samples.txt
> ~~~


~~~
$ grep REL1 SraRunTable.txt > special_samples.txt
$ wc -l special_samples.txt
~~~

which gives:

> ~~~
> 3 special_samples.txt
> ~~~

Still only at 3 lines, let's take a look again

~~~
$ less -S special_samples.txt
~~~


Here, the output of our second  call to `wc` shows that we no longer have any lines in our `paired_end_samples.txt` file. This is 
because the second time we searched our file was overwritten and now only contains the new matches.

We can avoid overwriting our files by using the command `>>`. `>>` is known as the "append redirect" and will 
append new output to the end of a file, rather than overwriting it.

~~~
$ grep CZB1 SraRunTable.txt >> special_samples.txt
$ wc -l special_samples.txt
~~~

> ~~~
> 6 special_samples.txt
> ~~~

We can verify what got written:

~~~
$ less -S special_samples.txt
~~~


We can also do this with a single line of code by using a wildcard. 

~~~
$ grep -B1 -A2 NNNNNNNNNN *.fastq > bad_reads.txt
$ wc -l bad_reads.txt
~~~

> ~~~
> 537 bad_reads.txt
> ~~~



> ## Making use of file extensions
> 
> This is where we would have trouble if we were naming our output file with a `.fastq` extension. 
> If we already had a file called `bad_reads.fastq` (from our previous `grep` practice) 
> and then ran the command above using a `.fastq` extension instead of a `.txt` extension, `grep`
> would give us a warning. 
> 
> ~~~
> grep -B1 -A2 NNNNNNNNNN *.fastq > bad_reads.fastq
> ~~~
> 
>> ~~~
>> grep: input file ‘bad_reads.fastq’ is also the output
>> ~~~
> 
> `grep` is letting you know that the output file `bad_reads.fastq` is also included in your
> `grep` call because it matches the `*.fastq` pattern. Be careful with this as it can lead to
> some surprising output.
> 


So far we've searched for reads containing a long string of at least 10 unknown nucleotides. 
We might also be interested in finding any reads with at least two shorter strings of 5 unknown 
nucleotides, separated by any number of known nucleotides. Reads with more than one region of 
ambiguity like this might be poor enough to not pass our quality filter. We can search for these
reads using a wildcard within our search string for `grep`. 

> ## Exercise
> 
> How many reads in the `SRR098026.fastq` file contain at least two regions of 5 unknown
> nucleotides in a row, separated by any number of known nucleotides?

****

****

****

>> ## Solution
>> 
>> ~~~
>> $ grep "NNNNN*NNNNN" SRR098026.fastq > bad_reads_2.txt
>> $ wc -l bad_reads_2.txt
>> ~~~
>> 
>>> ~~~
>>> 186 bad_reads_2.txt
>>> ~~~


## Pipes

What if we don't care to save the results of these searches, but also don't want a large number of matches to clutter up the screen?  We ca the pipe operator (`|`).

This is probably not a key on your keyboard you use very much, so let's all take a minute to find that key.  What `|` does is take the output that is scrolling by on the terminal and uses that output as input to another command.  When our output was scrolling by, we might have wished we could slow it down and look at it, like we can with `less`. Well it turns out that we can! We can redirect our output from our `grep` call through the `less` command.

~~~
$ grep SINGLE SraRunTable.txt | less -S
~~~

We can now see the output from our `grep` call within the `less` interface.  This can sort of be likened to how `man` uses less to show us the manual pages of commands.

Redirecting output is often not intuitive, and can take some time to get used to. Once you're comfortable with redirection, however, you'll be able to combine any number of commands to do all sorts of exciting things with your data!

None of the command line programs we've been learning do anything all that impressive on their own, but when you start chaining them together, you can do some really powerful things very efficiently. Let's take a few minutes to practice. 



## Streams - the second 'pillar' to understanding Unix/Linux
The word 'stream' has been used a few times in the lesson.  There are 3 main streams in Unix, and they are a core concept much like paths.  Whenever a file is accessed, its contents are put on the input stream.  Some commands and programs, like `less`, automatically direct the output stream to themselves.  The output stream goes directly to the terminal, to be displayed to the user.  So `less` takes the input stream and formats it a specific way to display it to the user in an interactive form on the output stream.  The third stream, standard error, is like the output stream, only it's a channel meant for error messages.  For example, when the shell reports an error in a commands usage, that is usually on the error stream, even though a user gets no indication whether it was from the output or error streams.  Some bioinformatics programs make use of the error stream to output diagnostics on their operation.

  

## Combining multiple commands with more piping

You've seen a simple way to search back through you history, but its a bit limited.  With pipes and the `grep` command, you can mine your history with a lot more power!  `history` sends its results to the output stream, and can be manipulated by the commands its piped to.

~~~
$ history | less
~~~

Instead of `less` opening a file, it takes as input the piped output stream of `history`.  Similarly, `grep` can take the output stream instead of specifying file(s) for it to operate on:

~~~
$ history | grep "mkdir"
~~~

Your command number will differ, but you should see at least this command in the output:

> ~~~
> 250  mkdir backup
> ~~~

You'll also notice the search itself is added to the history before the output is sent to grep

> ~~~
> 1007  history | grep "mkdir"
> ~~~


You can search on any part of line of the command:

~~~
$ history | grep "CZB1"
~~~

> ~~~
> 993  grep CZB1 SraRunTable.txt > special_samples.txt
> 1007  grep CZB1 SraRunTable.txt >> special_samples.txt
> 1015  history | grep "CZB1"
> ~~~

Since `grep` is also sending its results to the output stream, you can keep on piping:

~~~
$ history | grep "CZB1" | less
~~~

So long as a command or program outputs to the out stream, you can keep piping.  This is what a lot of bioinformatics utilities do, and are designed to chain a series of manipulations.

Eg:
`program1 -i input.data | program2 [some options] | program3 > results_file.format`

Or with some of the commands we've used:

~~~
$ history | grep "CZB1" > CZB1_uses.txt
$ less CZB1_uses.txt
~~~

  
## Variables

A variable is a way of storing a value.  Let's assign a variable a value and then look at it with the `echo` command.

~~~
$ id=sample01
$ echo $id
~~~

~~~
sample01
~~~

Assigning a value to a variable in the the bash shell takes the form of:

variable_name=[value]

However, in the example above, you access the value in the variable using `$` operator in front of the variable.  This may seem a bit odd since the bash
shell also uses `$` as the command prompt.  Try echoing the value stored in `id` without the `$`


If we use a space, the shell is not happy:

~~~
$ id=sample 01
~~~

~~~
-bash: 01: command not found
~~~

Instead, we need to use quotes to include spaces in the variable assignment

~~~
$ id="sample 01"
$ echo $id
~~~

~~~
sample 01
~~~

However, in general, its not a good idea to have spaces unless you have a specific need.

There are a number of predefined variables in the shell.  For example, I've talked a lot about paths, and there is a predefined set of paths the shell looks through when you try to execute a command.

~~~
$ echo $PATH
~~~

~~~
/usr/lib64/qt-3.3/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/ifs/sec/cpc/addhealth/apps/bin:/nas/longleaf/home/tristand/.local/bin:/nas/longleaf/home/tristand/bin
~~~

Each path the shell looks through is separated by `:`, and it looks for the command you type in order from the first until the last.  If the command wasn't found, then you get the error message

By convention, the predefined shell variables use all CAPS to distinguish them.  For now, you probably shouldn't mess with any of them.  But the fact you can modify these shows that Unix is very customizable - and dangerous.  If you overwrite your `$PATH` you won't be able to easily use the basic commands.

You can see all the preset shell variable with

~~~
$ env
~~~

One use of shell variables is to simplify our navigation of the file system.  You could for example record the path to your scratch space:

~~~
$ scr_path=/pine/scr/t/r/tristand
~~~

Again, substituting in your own scratch space.  We can now easily move to our scratch space, or include it in a longer path

~~~
$ pwd
$ cd $scr_path
$ pwd
~~~

If you still have a `shell_data` directory in your scratch space, you could include it to extend the path

~~~
$ cd ~
$ pwd
$ cd $scr_path/shell_data
$ pwd
~~~

The shell can resolve variable names if you are separating them out with `.` or `/`, but it is safest if you enclose the variable like this `${<variable_name}`

~~~
$ name=shell
$ echo $name
$ echo $name_data
$ echo ${name}_data
~~~

The shell interprets `$name_data` as a variable named 'name_data', not the variable 'name', to have its value appended with '_data'

****

