## Moving around the file system

We've learned how to use `pwd` to find our current location within our file system. 
We've also learned how to use `cd` to change locations and `ls` to list the contents
of a directory. Now we're going to learn some additional commands for moving around 
within our file system.

Use the commands we've learned so far to navigate to the `shell_data/untrimmed_fastq` directory, if
you're not already there. 

~~~
$ cd shell_data
$ cd untrimmed_fastq
~~~


What if we want to move back up and out of this directory and to our top level 
directory? Can we type `cd shell_data`? Try it and see what happens.

~~~
$ cd shell_data
~~~


> ~~~
> -bash: cd: shell_data: No such file or directory
> ~~~


Your computer looked for a directory or file called `shell_data` within the 
directory you were already in. It didn't know you wanted to look at a directory level
above the one you were located in. 

We have a special command to tell the computer to move us back or up one directory level. 

~~~
$ cd ..
~~~


Now we can use `pwd` to make sure that we are in the directory we intended to navigate
to, and `ls` to check that the contents of the directory are correct.

~~~
$ pwd
~~~

~~~
/home/dcuser/shell_data
~~~

~~~
$ ls
~~~

> ~~~
> sra_metadata  untrimmed_fastq
> ~~~

From this output, we can see that `..` did indeed take us back one level in our file system. 

You can chain these together like so:

~~~
$ ls ../../
~~~

prints the contents of `/home`, which is one level up from your root directory. 

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
give `ls` the names of other directories to view. Navigate to your
home directory if you are not already there.

~~~
$ cd
~~~
{: .bash}

Then enter the command:

~~~
$ ls shell_data
~~~
{: .bash}

~~~
sra_metadata  untrimmed_fastq
~~~
{: .output}

This will list the contents of the `shell_data` directory without
you needing to navigate there.

The `cd` command works in a similar way.

Try entering:

~~~
$ cd
$ cd shell_data/untrimmed_fastq
~~~
{: .bash}

This will take you to the `untrimmed_fastq` directory without having to go through
the intermediate directory.

> ## Navigating practice
> 
> Navigate to your home directory. From there, list the contents of the `untrimmed_fastq` 
> directory. 
> 
> > ## Solution
> >
> > ~~~
> > $ cd
> > $ ls shell_data/untrimmed_fastq/
> > ~~~
> > {: .bash}
> > 
> > ~~~
> > SRR097977.fastq  SRR098026.fastq 
> > ~~~
> > {: .output}
> > 
> {: .solution}
{: .challenge}

## Full vs. Relative Paths

The `cd` command takes an argument which is a directory
name. Directories can be specified using either a *relative* path or a
full *absolute* path. The directories on the computer are arranged into a
hierarchy. The full path tells you where a directory is in that
hierarchy. Navigate to the home directory, then enter the `pwd`
command.

~~~
$ cd  
$ pwd  
~~~
{: .bash}

You will see: 

~~~
/home/dcuser
~~~
{: .output}

This is the full name of your home directory. This tells you that you
are in a directory called `dcuser`, which sits inside a directory called
`home` which sits inside the very top directory in the hierarchy. The
very top of the hierarchy is a directory called `/` which is usually
referred to as the *root directory*. So, to summarize: `dcuser` is a
directory in `home` which is a directory in `/`.

Now enter the following command:

~~~
$ cd /home/dcuser/shell_data/.hidden
~~~
{: .bash}

This jumps forward multiple levels to the `.hidden` directory. 
Now go back to the home directory. 

~~~
$ cd
~~~
{: .bash}

You can also navigate to the `.hidden` directory using:

~~~
$ cd shell_data/.hidden
~~~
{: .bash}


These two commands have the same effect, they both take us to the `.hidden` directory.
The first uses the absolute path, giving the full address from the home directory. The
second uses a relative path, giving only the address from the working directory. A full
path always starts with a `/`. A relative path does not.

A relative path is like getting directions from someone on the street. They tell you to
"go right at the stop sign, and then turn left on Main Street". That works great if
you're standing there together, but not so well if you're trying to tell someone how to
get there from another country. A full path is like GPS coordinates. It tells you exactly
where something is no matter where you are right now.

You can usually use either a full path or a relative path
depending on what is most convenient. If we are in the home directory,
it is more convenient to enter the relative path since it
involves less typing.

Over time, it will become easier for you to keep a mental note of the
structure of the directories that you are using and how to quickly
navigate amongst them.



### Navigational Shortcuts

There are some shortcuts which you should know about. Dealing with the
home directory is very common. The tilde character,
`~`, is a shortcut for your home directory. Navigate to the `shell_data`
directory:

~~~
$ cd
$ cd shell_data
~~~
{: .bash}

Then enter the command:

~~~
$ ls ~
~~~
{: .bash}

~~~
shell_data
~~~
{: .output}

This prints the contents of your home directory, without you needing to 
type the full path. 
