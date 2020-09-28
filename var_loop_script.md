# Loops, and then, finally Scripts!

## What is a loop

A loop is a repeated set of commands, with some variable or variables changes during each time through the loop, called an iteration.

There are usually two main parts to a loop, a condition, and then a set of commands.  The first is called a loop header and the second the loop body

~~~
<some loop condition or range>  # the loop header
   <command>                      # loop body
   <command>                      # loop body
   ...                            # loop body
   <command>                      # loop body
~~~

Since they allow us to execute commands repeatedly, loops provide productivity improvements through automation. 
Similar to wildcards and tab completion, using loops also reduces the amount of typing and potential typing mistakes. 
For example,  when performing operations on groups of sequencing files, such as moving or running analyses on multiple
files. We will use loops for these purposes in subsequent analyses, but will cover the basics of them for now.

Here, we're going to learn about *for* loops, which iterate over a list of values.  There are other kinds of loops, such as *while* loops that execute until a certain condition is met.  


## Writing a for loop in the Unix shell

When the shell sees the keyword `for`, it knows to repeat a command (or group of commands) once for each item in a list. 
Each time the loop runs an iteration, an item in the list is assigned in sequence to the loop **variable**, and 
the commands inside the loop are executed, before moving on to the next item in the list. Inside the loop, we call for 
the variable's value by putting `$` in front of it just like any other **variable**. Recall, the `$` tells the shell interpreter to treat the **variable**
as a variable name and substitute its value in its place, rather than treat it as text or an external command. 

Let's write a for loop to show us the first two lines of the fastq files we downloaded earlier. You will notice shell prompt changes from `$` to `>` and back again as we were typing in our loop. The second prompt, `>`, is different to remind us that we haven’t finished typing a complete command yet. A semicolon, `;`, can be used to separate two commands written on a single line.

~~~
$ cd ../untrimmed_fastq/
~~~

~~~
$ for filename in *.fastq
> do
> head -n 2 ${filename}
> done
~~~

The for loop begins with the formula `for <variable> in <group to iterate over>`. In this case, the word `filename` is designated 
as the variable to be used over each iteration. In our case `SRR097977.fastq` and `SRR098026.fastq` will be substituted for `filename` 
because they fit the pattern of ending with .fastq in directory we've specified. The next line of the for loop is `do`. The next line is 
the code that we want to excute. We are telling the loop to print the first two lines of each variable we iterate over. Finally, the 
word `done` ends the loop.

After executing the loop, you should see the first two lines of both fastq files printed to the terminal. Let's create a loop that 
will save this information to a file.

~~~
$ for filename in *.fastq
> do
> head -n 2 ${filename} >> seq_info.txt
> done
~~~

Note that we are using `>>` to append the text to our `seq_info.txt` file. If we used `>`, the `seq_info.txt` file would be rewritten
every time the loop iterates, so it would only have text from the last variable used. Instead, `>>` adds to the end of the file.

### Using Basename in for loops

Basename is a function in UNIX that is helpful for removing a uniform part of a name from a list of files. In this case, we will use
basename to remove the `*.fastq` from the files that we've been working with. Inside our for loop, we create a new `name` variable.
We call the `basename` function inside the parenthesis, then give our variable name from the for loop, in this case `$filename`, 
and finally state that `.fastq` should be removed from the file name. It's important to note that we're not changing the actual files,
we're creating and manipulating a new variable called `name`. The line `> echo $name` will print to the terminal the variable `name`
each time the for loop runs. Because we are iterating over two files, we expect to see two lines of output.

~~~
$ for filename in *.fastq
> do
> name=$(basename ${filename} .fastq)
> echo ${name}
> done
~~~

Let's up arrow through our history to either of these for loops.  What do you notice?

All the commands for the loop are on one line, separated by `;` - which is a separator to indicate different commands are being issues, as if you hit <kbd>Enter</kbd> after each on.

~~~
$ cd ..; ls
~~~

Takes us up one level, then executes a listing of the directory.

From here, use the history to slightly modify our previous `for` loop to add `*/` before `in` - what is this causing the loop to do?

~~~
$ for filename in */*.fastq; do name=$(basename ${filename} .fastq); echo ${name}; done
~~~

Now, what happens if we modify the loop further to remove the use of `basename` and only echo out `$filename`

****

## More on Looping

Loops were quickly introduced in the last lesson, let's look at them in a bit more depth.

Conceptually, this is what is happening in a loop

~~~
for <item> in <some_set_of_things>
do
  <command>
  <more commands...>
done
~~~ 

So, for example:
~~~
$ for num in 1 2 3 4 5
> do
> echo $num
> done
~~~

Produces

~~~
1
2
3
4
5
~~~

Above, we used this loop header:

~~~
for filename in *.fastq
~~~

What's happening here is we're using the wildcard expansion to produce the list, which we can demonstrate with the `echo` command

~~~
$ echo *.fastq
~~~

~~~
Control_A.fastq Control_B.fastq Control_SR.fastq SRR097977.fastq SRR098026.fastq
~~~

There are a few ways we could supply this list or set of items to loop over.  The syntax `{X..Y}` will produce a range of values incrementing from X to Y

~~~
$ echo {3..7}
~~~

~~~
3 4 5 6 7
~~~

Substituting this into the previous for loop:

~~~
$ for num in {3..7}; do echo $num; done
~~~

~~~
3
4
5
6
7
~~~

Also, if we want to use the output of a shell command as the list, we would surround the command with $(<shell command>)

(make sure to be in the untrimmed_fastq directory, eg `cd ~/shell_data/untrimmed_fastq/`)

~~~
$ for item in $(cat file_sizes.txt); do echo $item; done
~~~

~~~
34K
Control_A.fastq
34K
Control_B.fastq
34K
Control_XX.fastq
194K
SRR097977.fastq
194K
SRR098026.fastq
490K
total
~~~

Let's do a regular `cat` on the file to see how the file was interpreted by the for loop

~~~
$ cat file_sizes.txt
~~~

We see that all uses of white space, including the line break, separate out each element to be used.



## Scripts

A script is just a collection of commands that will all be excuted at once.  Its a bit like having a bunch of commands separated by `;` only we don't have to type them out every time.

But there's more, combining several things we've learned, scripts can be fairly flexible

### An editor

First, we're going to need the ability to create script files.  Unix has a number of text editors, which range in complexity and power.  We're going to use one called `nano`, which has less features than most, but is designed to be more user friendly and act a bit more like a conventional word processing program.

~~~
$ nano
~~~

Like less, `nano` basically takes over the screen and has its own set of commands.  A quick guide to them is along the bottom two lines of the screen.  The notation here is `^G` means press the `control` key and the `g` key at the same time (you do not need to press shift for a capital 'G')

We can type stuff, and use `^O` to save it.  Then quit out and use `less <filename>` to see what you wrote is in fact in the file.

Now, we'll slowly build up a script we can use.  Run nano again and copy in our old grep search from last class.

Before, we just ran nano and named the file when we first saved, but we could also name a new file when we run nano

~~~
$ nano bad_reads_script.sh
~~~

`grep -B1 -A2 NNNNNNNNNN *.fastq > scripted_bad_reads.txt`

Save and exit.  Let's call the file 

We can execute the script like this:

~~~
$ bash bad_reads_script.sh
~~~

We can use `less` to see the results were written to `scripted_bad_reads.txt`

Scripts can also use variables, which is where their real power starts to kick in.

~~~
$ nano test.sh
~~~

Copy in the following command: `echo $pattern` and then save and exit.

~~~
$ pattern=blah
$ bash test.sh
~~~

Hmmm, nothing happened?  This is because invoking the shell with the `bash` command creatures a new temporary environment, and the `pattern` variable is not defined.  A different way of running a script is with the command `source`, which runs the script in the current environment:

~~~
$ source test.sh
~~~

~~~
blah
~~~

There are reasons why you might want to do one or the other.  But there is another way to get variables into a script.

~~~
$ nano test2.sh
~~~

Now copy this line into the new script: `echo $1`

Save and run with each command like this

~~~
$ bash test2.sh not_sourcing
$ source test2.sh not_bashing
~~~

`$1` is a special notation that means whatever the value that came first after the script name.  It is essentially like when you're using command line arguments.  When the shell executes `cd shell_data`, the value 'shell_data' is passed into the cd command as the equivalent of `$1`

`$2` is the second argument passed in, `$3` the third, etc.  Remember whitespace (other than newline) separates these, so:

~~~
$ source test2.sh not bashing
~~~

Gives:

~~~
not
~~~

'bashing' got passed in as the variable `$2`, but we didn't use it in the script.

Now, armed with this, let's make the grep script a lot more flexible to use.

~~~
$ nano grep_reads_script.sh
~~~

And make the following script:

~~~
pattern=$1
grep -B1 -A2 $pattern *.fastq > grepped_${pattern}_reads.txt
~~~

Assigning the value from `$1` is not strictly necessary, but its a form of self documentation - weeks or years later when you're looking at a script, you'll want a descriptive name to remind you what values you need to pass to the script.  We also use the pattern to create the name of the output file, using the ${} notation since we're embedding it in text.

~~~
$ source grep_reads_script.sh ATAT
$ less grepped_ATAT_reads.txt
~~~

You can use loops in scripts as well.  Let's make the following script:

~~~
$ nano loop_test.sh
~~~

~~~
suffix=$1
for file in *.$suffix
do
echo $file
done
~~~

Admittedly, all these scripts are fairly simple.  But there is no limit to how many commands you can put into them.  You could write a script with an entire bioinformatics workflow, or a set of modular smaller scripts to execute parts of the workflow (this allows for picking up work if one step fails instead of re-running everything).

When we get into some of the bioinformatics tools in the last 2 lessons, the complexity of some of the commands and the arguments you pass to them will make scripting a lot more appealing.



