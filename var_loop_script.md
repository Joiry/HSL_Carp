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



## Loops

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

NANO

grep example

passing variables

looping with variable?



