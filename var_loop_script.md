# More on Variables and Loops, and finally Scripts!


## Variables

As we saw before, a variable simply stores a value:

~~~
$ id=sample01
$ echo $id
~~~

~~~
sample01
~~~

If we use a space, the shell is not happy:

~~~
$ id=sample 01
~~~

~~~
-bash: 01: command not found
~~~

Instead, we'd need to use quotes

~~~
$ id="sample 01"
$ echo $id
~~~

~~~
sample 01
~~~

However, in general, its not a good idea to have spaces unless you have a specific reason.

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

The shell interprets `$name_data` as a variable named 'name_data', not the variable 'name' appended to '_data'

****
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

Last week, we used this loop header:

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

There are a few ways we could supply this list or set of items to loop over.  `{X..Y}` will produce a range of values incrementing from X to Y

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

Also, if we want to use the output of a shell command as the list, we would surround the command with $()

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

Hmmm, nothing happened?  This is because invoking the shell with the `bash` command creatures a new temporary environment, and the `pattern` variable is not defined.  A different way of running a script with with `source`, which runs the script in the current environment:

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



looping with variable?



