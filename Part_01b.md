## More moving around the file system

We've learned how to use `pwd` to find our current location within our file system. 
We've also learned how to use `cd` to change locations and `ls` to list the contents
of a directory. Now we're going to learn some additional commands for moving around 
within our file system.

Let's try using the `cd` command without any arguments

~~~
$ cd
~~~

Where are we now?

****

The `cd` command without any arguments takes you back to your home directory.  You're no longer in your scratch space and you'll have to navigate back to it.  However, instead of typing out that command again, we can use the shell's *history*.  The history is a log of the commands you've typed in.

Use the up arrow key to scroll back through your history until you find:  
(you can use the down arrow key to scroll forward, in case you skip by the command you want)

~~~
$ cd /pine/scr/t/r/tristand/
~~~

Except with your own info.  Hit enter.  You've exectued the command just as if you typed it out.

Often there are multiple ways to do something in the shell.  Say we wanted to navigate to the `shell_data/untrimmed_fastq` directory, we could use two commands:

~~~
$ cd shell_data
$ cd untrimmed_fastq
~~~

or use one, using `/` to separate each directory in the hierarchy we want to travel.

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



## Paths: Full vs. Relative 

Understanding *paths* is key to understanding the Unix shell.  The path, much like its name suggests, is where to find a file or directory.

For example, the `cd` command takes an argument which is a directory name - but we've combined multiple directories as one argument.  Both are examples of *paths*, which can be either a *relative* path or an *absolute* path, also called the *full* path.  We've seen how the directories on the system are arranged into a hierarchy. 

Imagine the file system as a building.  The front door of the building is like the *root directory*.  To get to a particular room, you could always use the directions from the front door, which floor, which hall, which room.  This would be the absolute path.  You're always navigating from the beginning.

However, if you're already in a room somewhere in the building, you could use directions from that room to another, this is like a relative path.  For example, "go out the door, turn left, go past three doors then take the next door on the right."

The `pwd` command gives you the absolute path to your current directory.  Both of these are absolute paths:

> ~~~
> /nas/longleaf/home/tristand
> ~~~

> ~~~
> /pine/scr/t/r/tristand
> ~~~

The absolute path starts with `/` and is followed by a valid hierarchy of directories.  Tab completion is very useful when navigating with absolute paths.

Enter the following command (substituting in your own scratch space path)

~~~
$ cd /pine/scr/t/r/tristand/shell_data/.hidden/
~~~

This navigates multiple levels to the `.hidden` directory, no matter where you are in the file system.

Now that we are in the `.hidden` directory, we can use a relative path to go to the `untrimmed` directory:

~~~
$ cd ../untrimmed_fastq/
~~~

Now try to use a relative path to get to the `sra_metadata`

~~~
$ cd shell_data/.hidden
~~~


You can usually use either a full path or a relative path
depending on what is most convenient. If we are moving just a few directory levels,
it' oftens more convenient to enter the relative path since it involves less typing.

Over time, it will become easier for you to keep a mental note of the
structure of the directories that you are using and how to quickly
navigate amongst them.  Always remember the `pwd` command, it can help you figure out the path.



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

Which leads to another little secret - many things in Unix are simply conventions.  Command options aren't guaranteed to be same between different commands, though usually they are.

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

Here's an extra trick, while in the man page, hit <kbd>/</kbd>.  You'll get a cursor in the bottom corner, now type '-r', and voila, you skip to the first time '-r' appears in the man page.  This is a general search feature, not just for the different command options or flags.  If you typed in 'recursively', it would have jumped to the next line down.

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
