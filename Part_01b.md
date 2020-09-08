# Second steps

## More moving around the file system

In the first lesson, we learned how to use `pwd` to find our current location within our file system, 
and also how to use `cd` to change locations and `ls` to list the contents of a directory.  Additionally,
we learned of the `man` command that gives you detailed information on most shell commands.  All this leading up
to a discussion of paths and how integral they are to thinking in a Unix mindframe.

Now we'll go more in depth in moving around within a file system.  Previously, storage locations other than
your home directory were mentioned.  We can't teach lesssons with /proj/<lab>/ spaces, since these are private 
to the individual labs.  But we can look at your scratch space.
  
Scratch is a special area where you can work with large files, but files are deleted after 36 days.  We'll dive in with a slightly complicated *absolute path*, it requires a few user specific substitutions:

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

If you do an `ls` here, it'll probably be empty unless you've used the scratch space before.

Now, let's try using the `cd` command without any arguments

~~~
$ cd
~~~

Where are we now?

****

The `cd` command without any arguments takes you back to your home directory.  You're no longer in your scratch space and you'd have to navigate back to it as we did before to work in it.  However, instead of typing out that command again, we can use the shell's *history*.  The history is a log of the commands you've typed in.

Use the up arrow key to scroll back through your history until you find:  
(you can use the down arrow key to scroll forward, in case you skip by the command you want)

~~~
$ cd /pine/scr/t/r/tristand/
~~~

Except with your own info.  Hit enter.  You've exectued the command just as if you typed it out.


## Conventions

Dealing with the home directory is very common. The *tilde* character, `~`, is a shortcut for the path to your home directory.  This is similar to how `.` is the shortcut for the current directory, and `..` is a shortcut for the next directory up.

~~~
$ ls ~
~~~

Will always list the contents of your home directory.

You'll note the following two commands do the same thing, take you to your home directory (don't do these yet, however).

~~~
$ cd
$ cd ~
~~~

So we see the commands `ls` and `cd` have different behaviors when no argument is supplied.

One of the little secrets of Unix is all these 'commands' we're using are actually just little programs, and the shell is letting you execute them.  And all these commands have been written and modified over many years, by different programmers.

Which leads to another little secret - many things in Unix are simply conventions.  Command options aren't guaranteed to be same between different commands, though often they are - or are thematically very similar.

****

Often there are multiple ways to do something in the shell, but just typing `cd` is easier than adding in the `~`, so why is the tilde useful?  You can use the `~` to create paths.  For example, if you are working in your scratch space and need to see the contents of a directory in your home directory:

~~~
$ ls ~/shell_data/untrimmed_fastq
~~~

Tab completion also works with paths using `~`:

~~~
$ ls ~/shell_data/un<tab>
~~~

Say we wanted to navigate to the `untrimmed_fastq` directory, which is in the `shell_data` directory.  We could do it with three commands (but don't):

~~~
$ cd
$ cd shell_data
$ cd untrimmed_fastq
~~~

or do it with one command, using `~` plus `/`'s to separate each directory in the hierarchy we want to travel.

~~~
$ cd ~/shell_data/untrimmed_fastq
~~~

As you work more and more in the shell, you'll find how useful the `~` is, and likely using it will become second nature.

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


****

****

> ### Challenge - Finding hidden directories
>
> First navigate to the `shell_data` directory. There is a hidden directory within this directory.
> Use `man` to explore the options for `ls` to find out how to see hidden directories. List the contents of the directory and 
> identify the name of the text file in that directory.
> 
> Hint: hidden files and folders in Unix start with `.`, for example `.my_hidden_directory`
>
****
****

You can string multiple command flags together, so long as they are the sort one letter version

~~~
$ ls -l -a
~~~

is the same as:

~~~
$ ls -la
~~~



### Wild cards

Navigate to your `untrimmed_fastq` directory.

~~~
$ cd ~/shell_data/untrimmed_fastq
~~~

Let's take a quick look at the files again:

~~~
$ ls
~~~

> ~~~
> Control_A.fastq  Control_SR.fastq  SRR097977.fastq
> Control_B.fastq  file_sizes.txt    SRR098026.fastq
> ~~~


What if we are interested in looking at only the FASTQ files in this directory. We can list
all files with the .fastq extension using the command:

~~~
$ ls *.fastq
~~~

You should only see these files (the column placement might be slightly different):

> ~~~
> Control_A.fastq  Control_SR.fastq  SRR098026.fastq
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

The `echo` command just prints whatever comes after the command to the screen:

~~~
$ echo This is just stuff I am typing out
~~~

We can use the command `echo` to see how the wildcard character is intepreted by the
shell.  

~~~
$ echo *.fastq
~~~


> ~~~
> Control_A.fastq Control_B.fastq Control_SR.fastq SRR097977.fastq SRR098026.fastq
> ~~~

The `*` is expanded to include any file that ends with `.fastq`.

The `*` can be used anywhere in the expression: beginning, middle, end.

~~~
$ ls Control_*.fastq
~~~

> ~~~
> Control_A.fastq  Control_B.fastq  Control_SR.fastq
> ~~~


`*` works with longer paths as well, let's look in the directory `/usr/bin/`

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


****

****
> ## Exercise
> Do each of the following tasks from your current directory using a single
> `ls` command for each.
> 
> 1.  List all of the files in `/usr/bin` that start with the letter 'y'.
> 2.  List all of the files in `/usr/bin` that end with the letter 'o'. 
> 3.  List all of the files in `/usr/bin` that contain the letter 'q'.

****

****



That last one required a little extra thought.  You can use the wildcard multiple times:

~~~
$ ls *S*fastq
~~~

> ~~~
> Control_SR.fastq  SRR097977.fastq  SRR098026.fastq
> ~~~

This also demostrates that `*` can match nothing.

The `?` character only matches a single character.

~~~
$ ls Control_?.fastq
~~~

> ~~~
> Control_A.fastq  Control_B.fastq
> ~~~

`Control_SR.fastq` is not listed, because it contains two letter between `Control_` and `.fastq`

Can you write a pattern to only show files with only 2 characters after `Control_` and before the file extension?




## Command History

As we saw earlier, you can repeat a command that you've run recently, accessing previous
commands using the up arrow on your keyboard to go back to the most recent
command. Likewise, the down arrow takes you forward in the command history.

A few more useful shortcuts: 

- <kbd>Ctrl</kbd>+<kbd>C</kbd> will cancel the command you are executing, and give you a 
fresh prompt.  *This is very useful if you've accidently entered a command that takes up too many resources for the login node*
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

****

> ## Exercise
> Find the line number in your history for the command that listed all the .fastq
> files that contained an 'S'. Rerun that command.

****

****

****

> > ## Solution
> > First type `history`. Then use `!` followed by the line number to rerun that command.

****


## Examining files

Let's head back to the `untrimmed` directory and look at a new command, `less`

~~~
$ less SRR098026.fastq
~~~

`less` is a program that allows us to inspect the contents of a file.  Technically, this isn't the first time you've used `less`, because `man` uses less to display the text of the manual entry.  You use the same commands to navigate.

Use the <kbd>q</kbd> key to quit less.

Let's jump up one directory:

~~~
$  cd ..
~~~

and now use less again:

~~~
$ less untrimmed_fastq/SRR098026.fastq
~~~

As we saw before, commands that take a file or directory as an argument can use any valid path.  So you can access files all across the filesystem.


## More on copying files

Finally, let's revisit the `cp`, aka `copy` command.  I had you use it with minimal explanation, but now let's take a deeper look.  Don't re-execute it, we'll just look at what we did:

`cp -r /proj/seq/data/carpentry/shell_data/ .`

Hopefully it makes more sense now.  We're copying from the absolute path `/proj/seq/data/carpentry/shell_data/` to our current location using the `.` shorthand.  But what is that `-r`?  Use the `man` command to find out.

Here's an extra trick, while in the man page, hit the <kbd>/</kbd> key.  You'll get a cursor in the bottom corner, now type '-r', and voila, you skip to the first time '-r' appears in the man page.  This is a general search feature, not just for the different command options or flags.  If you typed in 'recursively', it would have jumped to the next line down.

`-r`, also often `-R` in some commands, is one of the most powerful, and thus **dangerous**, options for commands that use it.  For example, look at the manual page of `ls`

~~~
$ man ls
~~~

Hit `space` bar a few times to scroll down, and see the different meanings of `-r` and `-R`, which aren't quite the same as the `cp` command.

In its basic usage, `cp` copies just one file to a new file.  Let's go back to the `untrimmed` directory and copy one of the files there.

~~~
$ cp SRR098026.fastq SRR098026_copy.txt
~~~

Since we are copying the file to the same location within the file system, we need to give it a new name, so for example we've added '\_copy' - which by itself would have been sufficient.  We've also changed the extension.  Most of the modern GUI based operating systems would pop up a warning over this.

Unix actually doesn't inherently care how you name your files.  File extensions are one of those many conventions I mentioned earlier.  They are not meaningless however, many commands or other programs take the extensions as hints as to what kind of data might be in the file.

~~~
$ cp SRR098026.fastq SRR.098026.copy.txt.zongaaa
~~~

We can use `less` to look both of these copies and see they are both still just the same text file as the original .fastq file.

****

## What we learned today

 * The main takeaway is that much of how the shell works is based on conventions, and not hard and fast rules
   * all the shell 'commands' are just little programs, written and modified by different programmers over the years
   * similar inputs to different commands don't always produce logically similar behavior
 * `~`, `.`, `..` are all shorthands for various paths
 * `*` and `?` are *wildcards*, they match, respectively, any/no characters or just a single one.
 * The shell keeps a log of all the commands you type, known as your *history*.


That's all for today.  Next class we'll get into more advanced manipulation of files with more specialized shell tools.
