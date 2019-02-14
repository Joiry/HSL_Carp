# Working with Files


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

### Wild cards

Navigate to your `untrimmed_fastq` directory.

~~~
$ cd ~/shell_data/untrimmed_fastq
~~~

Let's take a quick look at the files since I've added a few:

~~~
$ ls
~~~

We are interested in looking at the FASTQ files in this directory. We can list
all files with the .fastq extension using the command:

~~~
$ ls *.fastq
~~~

You should only see these files (the column placement might be slightly different):

> ~~~
> Control_A.fastq  Control_SS.fastq  SRR098026.fastq
> Control_B.fastq  SRR097977.fastq
> ~~~

The `*` character is a special type of character called a wildcard, which can be used to represent any number of any type of character. Thus, `*.fastq` matches every file that ends with `.fastq`, but not the file ending in `.txt`

This command: 

~~~
$ ls *977.fastq
~~~


> ~~~
> SRR097977.fastq
> ~~~


lists only the file that ends with `977.fastq`.

We can use the command `echo` to see how the wildcard character is intepreted by the
shell.  (echo just prints whatever comes after the command to the screen)

~~~
$ echo *.fastq
~~~


> ~~~
> Control_A.fastq Control_B.fastq Control_SS.fastq SRR097977.fastq SRR098026.fastq
> ~~~

The `*` is expanded to include any file that ends with `.fastq`.

The `*` can be used anywhere in the expression: beginning, middle, end.

~~~
$ ls Control_*.fastq
~~~

> ~~~
> Control_A.fastq  Control_B.fastq  Control_SS.fastq
> ~~~


`*` works with longer paths as well, let's look in the directory `/usr/bin`

~~~
$ ls /usr/bin/*.sh
~~~


> ~~~
> /usr/bin/gettext.sh             /usr/bin/lprsetup.sh
> /usr/bin/gflags_completions.sh  /usr/bin/rescan-scsi-bus.sh
> /usr/bin/gvmap.sh               /usr/bin/setup-nsssysinit.sh
> /usr/bin/lesspipe.sh            /usr/bin/unix-lpr.sh
> ~~~


Lists every file in `/usr/bin` that ends in the characters `.sh`.




> ## Exercise
> Do each of the following tasks from your current directory using a single
> `ls` command for each.
> 
> 1.  List all of the files in `/usr/bin` that start with the letter 'y'.
> 2.  List all of the files in `/usr/bin` that end with the letter 'o'. 
> 3.  List all of the files in `/usr/bin` that contain the letter 'q'.

~~~~

~~~~

That last one required a little extra thought.  You can use the wildcard multiple times:

~~~
$ ls *S*fastq
~~~

> ~~~
> Control_SS.fastq  SRR097977.fastq  SRR098026.fastq
> ~~~

This also demostrates that `*` can match nothing.


> > Bonus: `ls /usr/bin/*[ac]*`


## Command History

If you want to repeat a command that you've run recently, you can access previous
commands using the up arrow on your keyboard to go back to the most recent
command. Likewise, the down arrow takes you forward in the command history.

A few more useful shortcuts: 

- <kbd>Ctrl</kbd>+<kbd>C</kbd> will cancel the command you are writing, and give you a 
fresh prompt.
- <kbd>Ctrl</kbd>+<kbd>R</kbd> will do a reverse-search through your command history.  This
is very useful.
- <kbd>Ctrl</kbd>+<kbd>L</kbd> or the `clear` command will clear your screen.

You can also review your recent commands with the `history` command, by entering:

~~~
$ history
~~~

to see a numbered list of recent commands. You can reuse one of these commands
directly by referring to the number of that command.

For example, if your history looked like this:
(your numbers will be different)

> ~~~
> 259  ls *
> 260  ls /usr/bin/*.sh
> 261  ls *R1*fastq
> ~~~


then you could repeat command #260 by entering:

~~~
$ !260
~~~

Type `!` (exclamation point) and then the number of the command from your history.
You will be glad you learned this when you need to re-run very complicated commands.

> ## Exercise
> Find the line number in your history for the command that listed all the .sh
> files in `/usr/bin`. Rerun that command.
>
> > ## Solution
> > First type `history`. Then use `!` followed by the line number to rerun that command.


## Examining Files

We now know how to switch directories, run programs, and look at the
contents of directories, but how do we look at the contents of files?

One way to examine a file is to print out all of the
contents using the program `cat`. 

Enter the following command from within the `untrimmed_fastq` directory: 

~~~
$ cat SRR098026.fastq
~~~

This will print out all of the contents of the `SRR098026.fastq` to the screen.


> ## Exercise
> 
> 1. Print out the contents of the `~/shell_data/untrimmed_fastq/SRR097977.fastq` file. What is the last line of the file? 
> 2.  From your home directory, and without changing directories,
> use one short command to print the contents of all of the files in
> the `~/shell_data/untrimmed_fastq` directory.
> 
> > ## Solution
> > 1. The last line of the file is `TC:CCC::CCCCCCCC<8?6A:C28C<608'&&&,'$`.
> > 2. `cat ~/shell_data/untrimmed_fastq/*`

`cat` is a terrific program, but when the file is really big, it can be annoying to use, but can serve several purposes once you've learned a few more advanced techniques. 

Let's review the `less` command:

~~~
$ less SRR097977.fastq
~~~


Some navigation commands in `less`

| key     | action |
| ------- | ---------- |
| <kbd>Space</kbd> | to go forward |
|  <kbd>b</kbd>    | to go backward |
|  <kbd>g</kbd>    | to go to the beginning |
|  <kbd>G</kbd>    | to go to the end |
|  <kbd>q</kbd>    | to quit |

`less` also gives you a way of searching through files. Use the
"/" key to begin a search. Enter the word you would like
to search for and press `enter`. The screen will jump to the next location where
that word is found. 

**Shortcut:** If you hit "/" then "enter", `less` will  repeat
the previous search. `less` searches from the current location and
works its way forward. Note, if you are at the end of the file and search
for the sequence "CAA", `less` will not find it. You either need to go to the
beginning of the file (by typing `g`) and search again using `/` or you
can use `?` to search backwards in the same way you used `/` previously.

For instance, let's search forward for the sequence `TTTTT` in our file. 
You can see that we go right to that sequence, what it looks like,
and where it is in the file. If you continue to type `/` and hit return, you will move 
forward to the next instance of this sequence motif. If you instead type `?` and hit 
return, you will search backwards and move up the file to previous examples of this motif.

> ## Exercise
>
> What are the next three nucleotides (characters) after the first instance of the sequence quoted above?
> 



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


### File Permissions

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
don't have permission to carry out the ability encoded by that space (this is the space where `x` or executable ability is stored, we'll 
talk more about this in [a later lesson](http://www.datacarpentry.org/shell-genomics/05-writing-scripts/)).

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
just nicely put the files in the Trash. They're really gone.

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

**`rm -r` is the most dangerous command to issue.  Especially with wildcards**  Think about using `ls` with the same argument you will be using for `rm -r` to make sure you know what is about to be deleted forever.

Now, let's delete `backup` in a safer manner (answer 'y' to remove the write-protected file)

~~~
$ cd backup
$ rm SRR098026_backup.fastq 
$ cd ..
$ rmdir backup/
~~~

This time `rmdir` should work without any complaints from the system.
 

> ## Exercise
>
> Starting in the `shell_data/untrimmed_fastq/` directory, do the following:
> 1. Make sure that you have deleted your backup directory and all files it contains.  
> 2. Create a copy of each of your FASTQ files. (Note: You'll need to do this individually for each of the two FASTQ files. We haven't 
> learned yet how to do this
> with a wild-card.)  
> 3. Use a wildcard to move all of your backup files to a new backup directory.   
> 4. Change the permissions on all of your backup files to be write-protected.  
