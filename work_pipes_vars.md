****

A very useful command line option for `less` is `-S`, which makes it much easier to view text data arranged in columns.

Compare using the `-S` with not using it on the `SraRunTable.txt` file one directory over:

~~~
$ less ../sra_metadata/SraRunTable.txt
~~~

You can kind of see that there are columns, but the line wrapping makes it hard to see.

~~~
$ less -S ../sra_metadata/SraRunTable.txt
~~~


****

Another way to examine a file is to print out all of the contents using the program `cat`. 

Enter the following command from within the `untrimmed_fastq` directory: 

~~~
$ cat SRR098026.fastq
~~~

This will print out all of the contents of the `SRR098026.fastq` to the screen. When the file is really big, `cat` can be annoying to use.   

However, it may not seem like it at first, but `cat` is a useful program.  `cat` is short for concatenate, and its primary function is to join files together, but you will often see it used to just stream out individual file.  Next lesson we'll see it can serve several purposes once you've learned a few more advanced techniques. 


There's another way that we can look at files, and in this case, just
look at part of them. This can be particularly useful if we just want
to see the beginning or end of the file, or see how it's formatted.

The commands are `head` and `tail` and they let you look at
the beginning and end of a file, respectively.

~~~
$ head SRR098026.fastq
~~~

> ~~~
> @SRR098026.1 HWUSI-EAS1599_1:2:1:0:968 length=35
> NNNNNNNNNNNNNNNNCNNNNNNNNNNNNNNNNNN
> +SRR098026.1 HWUSI-EAS1599_1:2:1:0:968 length=35
> !!!!!!!!!!!!!!!!#!!!!!!!!!!!!!!!!!!
> @SRR098026.2 HWUSI-EAS1599_1:2:1:0:312 length=35
> NNNNNNNNNNNNNNNNANNNNNNNNNNNNNNNNNN
> +SRR098026.2 HWUSI-EAS1599_1:2:1:0:312 length=35
> !!!!!!!!!!!!!!!!#!!!!!!!!!!!!!!!!!!
> @SRR098026.3 HWUSI-EAS1599_1:2:1:0:570 length=35
> NNNNNNNNNNNNNNNNANNNNNNNNNNNNNNNNNN
> ~~~


~~~
$ tail SRR098026.fastq
~~~


> ~~~
> +SRR098026.247 HWUSI-EAS1599_1:2:1:2:1311 length=35
> #!##!#################!!!!!!!######
> @SRR098026.248 HWUSI-EAS1599_1:2:1:2:118 length=35
> GNTGNGGTCATCATACGCGCCCNNNNNNNGGCATG
> +SRR098026.248 HWUSI-EAS1599_1:2:1:2:118 length=35
> B!;?!A=5922:##########!!!!!!!######
> @SRR098026.249 HWUSI-EAS1599_1:2:1:2:1057 length=35
> CNCTNTATGCGTACGGCAGTGANNNNNNNGGAGAT
> +SRR098026.249 HWUSI-EAS1599_1:2:1:2:1057 length=35
> A!@B!BBB@ABAB#########!!!!!!!######
> ~~~


The `-n` option to either of these commands can be used to print the
first or last `n` lines of a file. 

~~~
$ head -n 1 SRR098026.fastq
~~~


> ~~~
> @SRR098026.1 HWUSI-EAS1599_1:2:1:0:968 length=35
> ~~~


~~~
$ tail -n 1 SRR098026.fastq
~~~

> ~~~
> A!@B!BBB@ABAB#########!!!!!!!######
> ~~~

When you know a file has special information at its beginning, such as meta data headers or column names, head can be especially useful:

~~~
$ head -n 1 ../sra_metadata/SraRunTable.txt
~~~

As with all these commands/programs, we can see it has several options listed on its man page:

~~~
$ man tail
~~~



## Creating, moving, copying, and removing

Now we can move around in the file structure, look at files, and search files. But what if we want to copy files or move
them around or get rid of them? Most of the time, you can do these sorts of file manipulations without the command line,
but there will be some cases (like when you're working with a remote computer like we are for this lesson) where it will be
impossible. You'll also find that you may be working with hundreds of files and want to do similar manipulations to all 
of those files. In cases like this, it's much faster to do these operations at the command line.

### Copying Files

When working with computational data, it's important to keep a safe copy of that data that can't be accidentally overwritten or deleted. 
For this lesson, our raw data is our FASTQ files.  We don't want to accidentally change the original files, so we'll make a copy of them
and change the file permissions so that we can read from, but not write to, the files.

First, let's make a copy of one of our FASTQ files using the `cp` command. 

Navigate to the `shell_data/untrimmed_fastq` directory and enter:

~~~
$ cp SRR098026.fastq SRR098026-copy.fastq
$ ls
~~~


> ~~~
> SRR097977.fastq  SRR098026-copy.fastq  SRR098026.fastq
> ~~~


We now have two copies of the `SRR098026.fastq` file, one of them named `SRR098026-copy.fastq`. We'll move this file to a new directory
called `backup` where we'll store our backup data files.

### Creating Directories

The `mkdir` command is used to make a directory. Enter `mkdir`
followed by a space, then the directory name you want to create.

~~~
$ mkdir backup
~~~


### Moving / Renaming 

We can now move our backup file to this directory. We can
move files around using the command `mv`. 

~~~
$ mv SRR098026-copy.fastq backup
$ ls backup
~~~
 
> ~~~
> SRR098026-copy.fastq
> ~~~


The `mv` command is also how you rename files. Let's rename this file to make it clear that this is a backup.

~~~
$ cd backup
$ mv SRR098026-copy.fastq SRR098026_backup.fastq
$ ls
~~~

> ~~~
> SRR098026_backup.fastq
> ~~~

### Multi copy

We've seen how using the recursive option, `-r` allows you to copy an entire directory and all the subdirectories.  `cp` can also copy multiple files, so long as the destination path given is a directory, and not a filename.

Move back out of the `backup` directory with `cd ..` and try the following command:

~~~
> cp Control*fastq backup/
> ls backup/
~~~

Remember, the wildcard expands out to a list that is passed to `cp`, so the names of all three of those files was used by `cp` - you'll note this line the man page for `cp`

> Copy SOURCE to DEST, or multiple SOURCE(s) to DIRECTORY.



****

## File Permissions

We've now made a backup copy of our file, but just because we have two copies doesn't make us safe. We can still accidentally delete or 
overwrite both copies. To make sure we can't accidentally mess up this backup file, we're going to change the permissions on the file so
that we're only allowed to read (i.e. view) the file, not write to it (i.e. make new changes).

View the current permissions on a file using the `-l` (long) flag for the `ls` command. 

~~~
$ ls -l
~~~


> ~~~
> -rw-r--r-- 1 tristand its_employee_psx 43332 Feb 13 19:21 SRR098026_backup.fastq
> ~~~


The first part of the output for the `-l` flag gives you information about the file's current permissions. There are ten slots in the
permissions list. The first character in this list is related to file type, not permissions, so we'll ignore it for now. The next three
characters relate to the permissions that the file owner has, the next three relate to the permissions for group members, and the final
three characters specify what other users outside of your group can do with the file. We're going to concentrate on the three positions
that deal with your permissions (as the file owner). 


Here the three positions that relate to the file owner are `rw-`. The `r` means that you have permission to read the file, the `w` 
indicates that you have permission to write to (i.e. make changes to) the file, and the third position is a `-`, indicating that you 
don't have permission to carry out the ability encoded by that space (this is the space where `x` or executable ability is stored).

Our goal for now is to change permissions on this file so that you no longer have `w` or write permissions. We can do this using the `chmod` (change mode) command and subtracting (`-`) the write permission `-w`. 

~~~
$ chmod -w SRR098026_backup.fastq
$ ls -l 
~~~


> ~~~
> -r--r--r-- 1 tristand its_employee_psx 43332 Feb 13 19:21 SRR098026_backup.fastq
> ~~~


### Removing

To prove to ourselves that you no longer have the ability to modify this file, try deleting it with the `rm` command.

~~~
$ rm SRR098026_backup.fastq
~~~


You'll be asked if you want to override your file permissions.

> ~~~
> rm: remove write-protected regular file ‘SRR098026_backup.fastq’?
> ~~~


If you enter `n` (for no), the file will not be deleted. If you enter `y`, you will delete the file. This gives us an extra 
measure of security, as there is one more step between us and deleting our data files.

**Important**: The `rm` command permanently removes the file. Be careful with this command. It doesn't
just nicely put the files in the Trash. They're really gone.  **Forever**

By default, `rm` will not delete directories.

~~~
$ cd ..
$ rm backup
~~~

> ~~~
> rm: cannot remove ‘backup/’: Is a directory
> ~~~

This is one of the **few** safety features in Unix.  There is a specific command to delete directories:

~~~
$ rmdir backup
~~~

> ~~~
> rmdir: failed to remove ‘backup’: Directory not empty
> ~~~

`rmdir` will only delete a directory that is empty.

You can tell `rm` to delete a directory using the `-r` (recursive) option. Let's try to delete the backup directory
we just made. (answer with 'n' for now)

~~~
$ rm -r backup
~~~

This will delete not only the directory, but all files within the directory. If you have write-protected files in the directory, you will be asked whether you want to override your permission settings.

**`rm -r` is the most dangerous command to issue.  Especially with wildcards**  Think about using `ls` with the same argument you will be using for `rm -r` to make sure you know what is about to be deleted forever.  **Forever**

Now, let's delete `backup` in a safer manner.  We could answer 'y' to remove the write-protected file, but here's how to restore the write permission

~~~
$ cd backup
$ chmod +w Control_A.fastq 
$ ls -la
~~~

> ~~~
> drwxr-xr-x 2 tristand its_employee_psx 4096 Feb 12 11:00 .
> drwxr-xr-x 3 tristand its_employee_psx 4096 Feb 12 11:00 ..
> -rw-r--r-- 1 tristand its_employee_psx 3990 Feb 12 11:00 Control_A.fastq
> ~~~

We're actually using a bit of shorthand, by default `chmod` assumes you're changing the owner/user permission field.  You can specify which of the permissions sets you are working with: `u` for user, `g` for group, `o` for other, and `a` for all - which is all three of u, g, and o.

So, for example, if you want to add permission for someone else in the group (for me "its_employee_psx"):

~~~
$ chmod g+w Control_A.fastq 
$ ls -la
~~~

~~~
> drwxr-xr-x 2 tristand its_employee_psx 4096 Feb 12 11:00 .
> drwxr-xr-x 3 tristand its_employee_psx 4096 Feb 12 11:00 ..
> -rw-rw-r-- 1 tristand its_employee_psx 3990 Feb 12 11:00 Control_A.fastq
~~~

See the new `w` has popped up in the third set of three.

After that aside, delete the file:

~~~
$ rm SRR098026_backup.fastq 
~~~

We also need to delete the control files.  Much like how we copied these files, we can use a wildcard, but **be warned this is also very dangerous**
When deleting using some wildcard pattern, it is best to check what will be deleted using the same pattern with `ls`

~~~
$ ls Control*fastq
~~~

~~~
> Control_A.fastq  Control_B.fastq  Control_SR.fastq
~~~

Now we know what the pattern matches, we can delete these with peace of mind.  This is a good time to use your history to up arrow and modify the `ls` command
into `rm`, instead of retyping the pattern.

~~~
$ rm Control*fastq
~~~

And now we can delete the directory safely:

~~~
$ cd ..
$ rmdir backup/
~~~

This time `rmdir` should work without any complaints from the system.

********

## Searching files

We discussed in a previous lesson how to search within a file using `less`. We can also
search within files without even opening them, using `grep`. `grep` is a command-line
utility for searching plain text files for lines matching a specific set of 
characters (sometimes called a string) or a particular pattern 
(which can be specified using something called regular expressions). We're not going to work with 
regular expressions in this lesson, and instead will use specific strings or with shell wildcards.


> Fastq Format
> There are 4 lines per read in a fastq file:
>
> @ (a line starting with @ has header info, usually technical info about the sequencer and the read itself)  
> ATCGCG... (the read sequence)  
> \+ (a line starting with +, that in older fastq files duplicated the @ line, but now is usually blank)    
> !!!#@$+$ (a line with numbers and symbols that are a code for the quality scores of each base above)  


We'll search for strings inside of our fastq files. Let's first make sure we are in the correct directory.

~~~
$ cd ~/shell_data/untrimmed_fastq
~~~


Suppose we want to see how many reads in our file have really bad segments containing 10 consecutive unknown nucleoties (Ns). Let's search for the string NNNNNNNNNN in the SRR098026 file.

> ## Determining quality
> 
> In this lesson, we're going to be manually searching for strings of `N`s within our sequence
> results to illustrate some principles of file searching. It can be really useful to do this
> type of searching to get a feel for the quality of your sequencing results, however, in you 
> research you will most likely use a bioinformatics tool that has a built-in program for
> filtering out low-quality reads.


~~~
$ grep NNNNNNNNNN SRR098026.fastq
~~~

This command returns a lot of output to the terminal. Every single line in the SRR098026 
file that contains at least 10 consecutive Ns is printed to the terminal, regardless of how long or short the file is. 
We may be interested not only in the actual sequence which contains this string, but 
in the name (or identifier) of that sequence. We discussed in a previous lesson 
that the identifier line immediately precedes the nucleotide sequence for each read
in a FASTQ file. We may also want to inspect the quality scores associated with
each of these reads. To get all of this information, we will return the line 
immediately before each match and the two lines immediately after each match.

We can use the `-B` argument for grep to return a specific number of lines before
each match and the `-A` argument to return a specific number of lines after each matching line. Here we want the line before and the two lines after each 
matching line so we add `-B1 -A2` to our grep command.

~~~
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq
~~~

One of the sets of lines returned by this command is: 

> ~~~
> @SRR098026.177 HWUSI-EAS1599_1:2:1:1:2025 length=35
> CNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
> +SRR098026.177 HWUSI-EAS1599_1:2:1:1:2025 length=35
> #!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
> ~~~


> ## Exercise
>
> 1. Search for the sequence `GNATNACCACTTCC` in the `SRR098026.fastq` file.
> Have your search return all matching lines and the name (or identifier) for each sequence
> that contains a match.
> 
> 2. Search for the sequence `AAGTT` in both FASTQ files.
> Have your search return all matching lines and the name (or identifier) for each sequence
> that contains a match.

****

****

****

> > ## Solution  
> > 1. `grep -B1 GNATNACCACTTCC SRR098026.fastq`  
> > 2. `grep -B1 AAGTT *.fastq`

## Redirecting output

`grep` allowed us to identify sequences in our FASTQ files that match a particular pattern. 
All of these sequences were printed to our terminal screen, but in order to work with these 
sequences and perform other opperations on them, we may need to capture that output in some
way. 

We can do this with something called "redirection". The idea is that
we are taking what would ordinarily be printed to the terminal screen and redirecting it to another location. 
In our case, we want to stream this information into a file so that we can look at it later and 
use other commands to analyze this data.

The operator for redirecting output to a file is `>`.

Let's try out this command and copy all the records (including all four lines of each record) 
in our FASTQ files that contain 'NNNNNNNNNN' to another file called `bad_reads.txt`.

~~~
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq > bad_reads.txt
~~~


The prompt should sit there a little bit, and then it should look like nothing
happened. But type `ls`. You should see a new file called `bad_reads.txt`. 

We can check the number of lines in our new file using a command called `wc`. 
`wc` stands for **word count**. This command counts the number of words, lines, and characters
in a file. 

~~~
$ wc bad_reads.txt
~~~

> ~~~
>   537  1073 23217 bad_reads.txt
> ~~~

This will tell us the number of lines, words and characters in the file. If we
want only the number of lines, we can use the `-l` flag for `lines`.

~~~
$ wc -l bad_reads.txt
~~~

> ~~~
> 537 bad_reads.txt
> ~~~

Because we asked `grep` for all four lines of each FASTQ record, we need to divide the output by
four to get the number of sequences that match our search pattern.

> ## Exercise
>
> How many sequences in `SRR098026.fastq` contain at least 3 consecutive Ns?

****

****

****

>> ## Solution
>>  
>>
>> ~~~
>> $ grep NNN SRR098026.fastq > bad_reads.txt
>> $ wc -l bad_reads.txt
>> ~~~
>> 
>>> ~~~
>>> 249
>>> ~~~


We might want to search multiple FASTQ files for sequences that match our search pattern.
However, we need to be careful, because each time we use the `>` command to redirect output
to a file, the new output will replace the output that was already present in the file. 
This is called "overwriting" and, just like you don't want to overwrite your video recording
of your kid's first birthday party, you also want to avoid overwriting your data files.

~~~
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq > bad_reads.txt
$ wc -l bad_reads.txt
~~~


> ~~~
> 537 bad_reads.txt
> ~~~


~~~
$ grep -B1 -A2 NNNNNNNNNN SRR097977.fastq > bad_reads.txt
$ wc -l bad_reads.txt
~~~


> ~~~
> 0 bad_reads.txt
> ~~~


Here, the output of our second  call to `wc` shows that we no longer have any lines in our `bad_reads.txt` file. This is 
because the second file we searched (`SRR097977.fastq`) does not contain any lines that match our
search sequence. So our file was overwritten and is now empty.

We can avoid overwriting our files by using the command `>>`. `>>` is known as the "append redirect" and will 
append new output to the end of a file, rather than overwriting it.

~~~
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq > bad_reads.txt
$ wc -l bad_reads.txt
~~~


> ~~~
> 537 bad_reads.txt
> ~~~


~~~
$ grep -B1 -A2 NNNNNNNNNN SRR097977.fastq >> bad_reads.txt
$ wc -l bad_reads.txt
~~~

> ~~~
> 537 bad_reads.txt
> ~~~

The output of our second call to `wc` shows that we have not overwritten our original data. 

Well, nothing got added, so let's try a search pattern that will:

~~~
$ grep -B1 -A2 AAAAAA SRR097977.fastq >> bad_reads.txt
$ wc -l bad_reads.txt
~~~

> ~~~
> 583 bad_reads.txt
> ~~~


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



We've now created two separate files to store the results of our search for reads matching 
particular criteria. Since we might have multiple different criteria we want to search for, 
creating a new output file each time has the potential to clutter up our workspace. We also
so far haven't been interested in the actual contents of those files, only in the number of 
reads that we've found. We created the files to store the reads and then counted the lines in 
the file to see how many reads matched our criteria. There's a way to do this, however, that
doesn't require us to create these intermediate files - the pipe command (`|`).

This is probably not a key on
your keyboard you use very much, so let's all take a minute to find that key. 
What `|` does is take the output that is
scrolling by on the terminal and uses that output as input to another command. 
When our output was scrolling by, we might have wished we could slow it down and
look at it, like we can with `less`. Well it turns out that we can! We can redirect our output
from our `grep` call through the `less` command.

~~~
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq | less
~~~

We can now see the output from our `grep` call within the `less` interface. We can use the up and down arrows 
to scroll through the output and use `q` to exit `less`.

Redirecting output is often not intuitive, and can take some time to get used to. Once you're 
comfortable with redirection, however, you'll be able to combine any number of commands to
do all sorts of exciting things with your data!

None of the command line programs we've been learning
do anything all that impressive on their own, but when you start chaining
them together, you can do some really powerful things very
efficiently. Let's take a few minutes to practice. 

> ## Exercise
>
> Now that we know about the pipe (`|`), write a single command to find the number of reads 
> in the `SRR098026.fastq` file that contain at least two regions of 5 unknown
> nucleotides in a row, separated by any number of known nucleotides. Do this without creating 
> a new file.

****

****

****

>> ## Solution
>> 
>> ~~~
>> $ grep "NNNNN*NNNNN" SRR098026.fastq | wc -l
>> ~~~
>>
>>> ~~~
>>> 186
>>> ~~~

## Streams
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
$ history | grep "NNNN"
~~~

> ~~~
> 103  grep NNNNNNNNNN SRR098026.fastq 
> 104  grep -B1 -A2 NNNNNNNNNN SRR098026.fastq 
> 107  grep -B1 -A2 NNNNNNNNNN SRR098026.fastq > bad_reads.txt
> 118  grep -B1 -A2 NNNNNNNNNN SRR098026.fastq > bad_reads.txt
> 121  grep -B1 -A2 NNNNNNNNNN SRR097977.fastq > bad_reads.txt
> 123  grep -B1 -A2 NNNNNNNNNN SRR098026.fastq > bad_reads.txt
> 125  grep -B1 -A2 NNNNNNNNNN SRR097977.fastq >> bad_reads.txt
> ~~~

Since `grep` is also sending its results to the output stream, you can keep on piping:

~~~
$ history | grep "NNNN" | less
~~~

So long as a command or program outputs to the out stream, you can keep piping.  This is what a lot of bioinformatics utilities do, and are designed to chain a series of manipulations.

Eg:
`program1 -i input.data | program2 [some options] | program3 > results_file.format`

Or with some of the commands we've used:

~~~
$ history | grep "basename" > basename_uses.txt
$ less basename_uses.txt
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
