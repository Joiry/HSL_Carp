# Working with Files
In today's lesson:
 * Learn more about using `less` and some other tools for looking at files.
 * Go into more depth in copying files, as well as creating directories and moving files around
 * Learn about file permissions and how to change them
 * See how to delete files


## Examining Files

We now know how to switch directories and look at the contents of directories.  And briefly how we can look at the contents of files.

Let's review the `less` command, first returning to a directory with some files:

~~~
$ cd cd shell_data/untrimmed_fastq/
$ ls
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

`less` also gives you a way of searching through files. Use the "/" key to begin a search. Enter the word you would like
to search for and press `enter`. The screen will jump to the next location where that word is found. 

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

****

****

> ## Exercise
>
> From the beginning of the file, search for TCAAAT, then keep repeating the search and count the total number of occurences

****

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
 

> ## Exercise
>
> Starting in the `shell_data/untrimmed_fastq/` directory, do the following:
> 1. Make sure that you have deleted your backup directory and all files it contains.  
> 2. Create a copy of each of your FASTQ files. (Note: You'll either need to do this individually for each of the FASTQ files or use a wildcard pattern copy)  
> 3. Use a wildcard to move all of your backup files to a new backup directory.   
> 4. Change the permissions on all of your backup files to be write-protected.  
