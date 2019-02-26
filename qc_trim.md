# QC and trimming of your data


## Modules

The Longleaf cluster uses a system called modules.  Unlike the "basic" commands of the shell, modules are used for more specific purpose software, often whole packages.

Why use modules?  Well, as we'll see, there's usually multiple versions of the same package available at once.



~~~
$ module
~~~

A lot of stuff scrolls by, and it doesn't look like the usual man page or other help info a command might provide.

`module` is a program that groups several functions, and you decide which you want to use with a subcommand:

`module <subcommand>'

Let's try this:

~~~
$ module avail
~~~

This subcommand lists out all the available modules, and conveniently pipes it through `less`

You see many of the modules have multiple versions, those with a (D) are the default version.  

'load' is the subcommand that will activate the module for use.  If you typed:

`module load <module_name>`

You would get the default version of that module.  Let's try loading a module, but don't hit <kbd>enter</kbd> quite yet:

~~~
$ module load fastqc<tab><tab>
~~~

By hitting tab, you can see the other versions available.  Why are these other versions there - the short answer is you should treat version numbers of software the same as you might lot numbers of reagents.

Updates to packages may fix earlier bugs, improve an algorithm, or introduce new bugs or behaviors.  Its important to document which version you're using.
Sometimes you may do an analysis months or years after an initial analysis - you may not want to rerun the originals with a newer version of the software.

Also, if comparing to results from another lab's paper, you may want to use the same version of the program.

Go ahead and hit enter so we've loaded the `fastqc` module.

~~~
$ module list
~~~

~~~
Currently Loaded Modules:
  1) fastqc/0.11.7
~~~

~~~
$
~~~
~~~
$
~~~

## Trimming


~~~
$
~~~
~~~
$
~~~


## Re-QCing


## MultiQC
