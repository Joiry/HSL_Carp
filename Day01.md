# Introduction to the Shell

## Login

from command line...
onyen   <onyen>

`ssh <onyen>@longleaf.unc.edu`
  
for me would be:  
  
`ssh tristand@longleaf.unc.edu`


from terminal program...

`$` is the prompt, telling you the shell is ready to take a command.

`[tristand@longleaf-login2 ~]$`

~~~
$ pwd
~~~

~~~
/nas/longleaf/home/tristand
~~~

This is your home directory, where you will always be when logging in.  However, there isn't much disk space allocated to individual home directories.  For genomics data sets, you'll have to work in the scratch space or your groups `/proj/<labname/` directory.

~~~
$ cd shell_data
~~~


We can see files and subdirectories are in this directory by running `ls`,
which stands for "listing":

~~~
$ ls
~~~


~~~
sra_metadata  untrimmed_fastq
~~~


`ls` prints the names of the files and directories in the current directory in
alphabetical order,
arranged neatly into columns.
We can make its output more comprehensible by using the **flag** `-F`,
which tells `ls` to add a trailing `/` to the names of directories:

~~~
$ ls -F
~~~


~~~
sra_metadata/  untrimmed_fastq/
~~~


Anything with a "/" after it is a directory. Things with a "*" after them are programs. If
there are no decorations, it's a file.

`ls` has lots of other options. To find out what they are, we can type:

~~~
$ man ls
~~~


Some manual files are very long. You can scroll through the file using
your keyboard's down arrow or use the <kbd>Space</kbd> key to go forward one page
and the <kbd>b</kbd> key to go backwards one page. When you are done reading, hit <kbd>q</kbd>
to quit.

> ## Challenge
> Use the `-l` option for the `ls` command to display more information for each item 
> in the directory. What is one piece of additional information this long format
> gives you that you don't see with the bare `ls` command?
>
> > ## Solution
> > ~~~
> > $ ls -l
> > ~~~
> > 
> > ~~~
> > total 8
> > drwxr-x--- 2 dcuser dcuser 4096 Jul 30  2015 sra_metadata
> > drwxr-xr-x 2 dcuser dcuser 4096 Nov 15  2017 untrimmed_fastq
> > ~~~
> > 
> > The additional information given includes the name of the owner of the file,
> > when the file was last modified, and whether the current user has permission
> > to read and write to the file.
> > 

No one can possibly learn all of these arguments, that's why the manual page
is for. You can (and should) refer to the manual page or other help files
as needed.

Let's go into the `untrimmed_fastq` directory and see what is in there.

~~~
$ cd untrimmed_fastq
$ ls -F
~~~


~~~
SRR097977.fastq  SRR098026.fastq
~~~


This directory contains two files with `.fastq` extensions. FASTQ is a format
for storing information about sequencing reads and their quality.
We will be learning more about FASTQ files in a later lesson.

### Shortcut: Tab Completion

Typing out file or directory names can waste a
lot of time and it's easy to make typing mistakes. Instead we can use tab complete 
as a shortcut. When you start typing out the name of a directory or file, then
hit the <kbd>Tab</kbd> key, the shell will try to fill in the rest of the
directory or file name.

Return to your home directory:

~~~
$ cd
~~~


then enter:

~~~
$ cd she<tab>
~~~


The shell will fill in the rest of the directory name for
`shell_data`.

Now change directories to `untrimmed_fastq` in `shell_data`

~~~
$ cd shell_data
$ cd untrimmed_fastq
~~~


Using tab complete can be very helpful. However, it will only autocomplete
a file or directory name if you've typed enough characters to provide
a unique identifier for the file or directory you are trying to access.

If we navigate back to our `untrimmed_fastq` directory and try to access one
of our sample files:

~~~
$ cd
$ cd shell_data
$ cd untrimmed_fastq
$ ls SR<tab>
~~~


The shell auto-completes your command to `SRR09`, because all file names in 
the directory begin with this prefix. When you hit
<kbd>Tab</kbd> again, the shell will list the possible choices.

~~~
$ ls SRR09<tab><tab>
~~~


~~~
SRR097977.fastq  SRR098026.fastq
~~~


Tab completion can also fill in the names of programs, which can be useful if you
remember the begining of a program name. 

~~~
$ pw<tab><tab>
~~~


~~~
pwd         pwd_mkdb    pwhich      pwhich5.16  pwhich5.18  pwpolicy
~~~


Displays the name of every program that starts with `pw`. 
