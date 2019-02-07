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

We can also chain together the special `..` notion for the next directory up

~~~
$ cd ../..
~~~

Where are we now?



> ## Challenge - Finding hidden directories
>
> First navigate to the `shell_data` directory. There is a hidden directory within this directory. Explore the options for `ls` to 
> find out how to see hidden directories. List the contents of the directory and 
> identify the name of the text file in that directory.
> 
> Hint: hidden files and folders in Unix start with `.`, for example `.my_hidden_directory`
>


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




> ## Listing practice
> 
> Try to list the contents of the `untrimmed_fastq` directory from the top of your scratch space. 
> 
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
>


## Paths: Full vs. Relative 

Understanding *paths* is key to understanding the Unix shell.  The path, much like its name suggests, is where to find a file or directory.

For example, the `cd` command takes an argument which is a directory name - but we've combined multiple directories as one argument.  Both are examples of *paths*, which can be either a *relative* path or an *absolute* path.  We've seen how the directories on the system are arranged into a hierarchy. 

Imagine the file system as a building.  The front door of the building is like the *root directory*.  To get to a particular room, you could always use the directions from the front door, which floor, which hall, which room.  This would be the absolute path.  You're always navigating from the beginning.

However, if you're already in a room somewhere in the building, you could use directions from that room to another, this is like a relative path.  For example, "go out the door, turn left, go past three doors then take the next door on the right."

The `pwd` command gives you the absolute path to your current directory.  Both of these are absolute paths:

> ~~~
> /nas/longleaf/home/tristand
> ~~~

> ~~~
> /pine/scr/t/r/tristand
> ~~~

The absolute path starts with `/` and has to have the correct hierarchy of directories.  Tab completion is very useful when navigating with absolute paths.

Enter the following command (substituting in your own scratch space)

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
{: .bash}


You can usually use either a full path or a relative path
depending on what is most convenient. If we are in the home directory,
it is more convenient to enter the relative path since it
involves less typing.

Over time, it will become easier for you to keep a mental note of the
structure of the directories that you are using and how to quickly
navigate amongst them.  Always remember the `pwd` command, it can help you figure out relative paths.



### Navigational Shortcuts

There are some shortcuts which you should know about. Dealing with the
home directory is very common. The tilde character, `~`, is a shortcut for your home directory

~~~
$ ls ~
~~~

Will always list the contents of your home directory.

You'll note the following two commands do the same thing

~~~
$ cd
$ cd ~
~~~

So we see the commands `ls` and `cd` have different behaviors when no argument is supplied.
