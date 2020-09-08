# Second steps

## More moving around the file system

In the first lesson, we learned how to use `pwd` to find our current location within our file system, 
and also how to use `cd` to change locations and `ls` to list the contents of a directory.  Leading up
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

The `cd` command without any arguments takes you back to your home directory.  You're no longer in your scratch space and you'd have to navigate back to it as we did before to use it.  However, instead of typing out that command again, we can use the shell's *history*.  The history is a log of the commands you've typed in.

Use the up arrow key to scroll back through your history until you find:  
(you can use the down arrow key to scroll forward, in case you skip by the command you want)

~~~
$ cd /pine/scr/t/r/tristand/
~~~

Except with your own info.  Hit enter.  You've exectued the command just as if you typed it out.


## Conventions

Dealing with the home directory is very common. The tilde character, `~`, is a shortcut for the path to your home directory.  This is similar to how `.` is the shortcut for the current directory, and `..` is a shortcut for the next directory up.

~~~
$ ls ~
~~~

Will always list the contents of your home directory.

You'll note the following two commands do the same thing, take you to your home directory.

~~~
$ cd
$ cd ~
~~~

So we see the commands `ls` and `cd` have different behaviors when no argument is supplied.

One of the little secrets of Unix is all these 'commands' we're using are actually just little programs, and the shell is letting you execute them.  And all these commands have been written and modified over many years, by different programmers.

Which leads to another little secret - many things in Unix are simply conventions.  Command options aren't guaranteed to be same between different commands, though often they are - or are thematically very similar.
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


****

****

> ## Challenge - Finding hidden directories
>
> First navigate to the `shell_data` directory. There is a hidden directory within this directory. Explore the options for `ls` to 
> find out how to see hidden directories. List the contents of the directory and 
> identify the name of the text file in that directory.
> 
> Hint: hidden files and folders in Unix start with `.`, for example `.my_hidden_directory`
>

****

****


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





<move this up to after the cd command, also add in using ~/path_in_your_home type usage>



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



Finally, let's revisit the `cp`, aka `copy` command.  I had you use it with minimal explanation, but now let's take a deeper look.  Don't re-execute it, we'll just look at what we did:

`cp -r /proj/seq/data/carpentry/shell_data/ .`

Hopefully it makes more sense now.  We're copying from the absolute path `/proj/seq/data/carpentry/shell_data/` to our current location using the `.` shorthand.  But what is that `-r`?  Use the `man` command to find out.

Here's an extra trick, while in the man page, hit the <kbd>/</kbd> key.  You'll get a cursor in the bottom corner, now type '-r', and voila, you skip to the first time '-r' appears in the man page.  This is a general search feature, not just for the different command options or flags.  If you typed in 'recursively', it would have jumped to the next line down.

`-r`, also often `-R`, is one of the most powerful, and thus **dangerous**, options for commands that use it.

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

<bring in command history and wilcards from 2a to this lesson>

That's all for today.  Next class we'll get into more advanced manipulation of files with more specialized shell tools.
