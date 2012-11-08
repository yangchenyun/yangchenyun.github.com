---
layout: post
title: "Understand Unix Standard Streams, Part 1"
date: 2012-10-01 16:54
comments: true
categories: unix
keywords: unix, io, file, pipe, stream, process substitution
published: false
---

For linux beginners, these commands are powerful but confusing:

```bash
tail /var/log/nginx.log > /tmp/recent.log
node server.js 2>&1 >/var/log/info.log
ls -al | grep '/.rb' | wc -l
tee >(wc -l >&2) < bigfile | gzip > bigfile.gz
```

The shared syntaxes such as `<`, `>`, `>&1` or `|` indicates one of unix's best features - standard IO streams. Standard IO streams has been introduced since [the early days of unix][yt-unix-intro] and has formed a core part in [unix philosophy][unix-philosophy].

> ... Write programs that do one thing and do it well. Write programs to work together. Write programs to handle text streams, because that is a universal interface.

The following sections will explain IO streams of unix and their usage with programs. It will answer the following questions:
- what is standard streams and how does it work with programs?
- how to utilize standard streams in programs?
- what is pipeline and how to use it?
- how to make standard streams work with program arguments?

If your are confused about unix streams, hope the following sections will help you understand the above commands; if you are already familiar with it, [references at the end][#references] might help you unleash more power of it.

* Table of Content
{:toc}

## The Abstract IO Interface for Program
Computer programs need to talk with its environment. It needs to communicate with various hardware such as disk drive, disc, screen, keyboard, mouse, speaker, printer etc. 

In operating system predates Unix, program has to contain the 'knowledge' for specific devices. It need to know the right ways to talk to various devices. To build a program working with three types of different printers, it needs to contain the pieces to translate program instructions to each printer.

![img of program talks to various devices]()

This binding with devices makes program less portable.

To solve this issue, unix invented the **standard IO streams**

Standard IO streams abstract the complexity of reading and writing to different devices with **data stream**. Data stream is merely an ordered sequence of data bytes. Any program could read data stream until the end of a file and it could write data stream without declare the size. This generic ways to read and write data enables program to read input from keyboard devices, files and write data to displays, files, printers etc. without knowing anything about these devices.

![img of program talks to standard io]()

There are three kinds of standard IO streams including standard input, standard output and standard error. Standard input is the data going into a program. By default it is expected from keyboard. Standard output is the where program writes its data to and by default it is the terminal. Standard error is another output stream similar to standard output. It is typically used to output error messages or diagnostics.

Standard streams follows the unix's phisolophy that ['everything is a file'][everything-is-a-file]. So from the program's point of view, these three standard streams have no difference from normal files. As long as a program could read a file, it could read from standard input; as long as it could write a file, it could write the standard output.

The actual actions of 'writing to standard output' is determined by unix itself. It could mean printing to terminal, recording a video tape, burning a disc, sending to printer etc. All of these are handled by the kernel of unix. This feature amplifies the composibility and portablity of a program

![img of program talks to standard io]()

In conslusion, standard IO streams provide an unified input and output interfaces for the programs to communicate with various hardware devices without growing complexities. 

### Example of Program Talks with Standard Streams
With the knowledge about the standard streams, let's take a look at one program example.

Type `cat` in the terminal and now the program will read the default standard input - keyboard. Type `first line` and `<Enter>`, it prints to string to the default standard output - the terminal.

    $ cat
    first line
    >first line

![img of command cat]()

## IO Redirections
Sometimes we want to read data other than the keyboard or write data other than standard output. To alter the default stream source or destination is called **IO Redirection**. This could be archieved through `<` or `>`. Let's first try `>`:

    $ ls -al > /tmp/list.txt

It redirects the standard output to the file `/tmp/list.txt` instead of print to terminal.

`>` has a sibling `>>` which will **append** data to file instead of writing it.

    $ ls -al >> /tmp/list.txt

It appends the result to `/tmp/list.text` which should contain the same results twice. Let's read this file through stdin now:

    $ less < /tmp/list.txt

It reads standard input from `/tmp/list.txt` instead of the keyboard.

![img of IO Redirection]()

To redirect standard error use `2>`:

    $ find this-is-not-exist 2> /tmp/error.txt

The error message is not displayed on the terminal but stored in the `/tmp/error.txt` file.

The meaning of the `2` used in the above command is called **file descriptor**, they are used to access file in one system process just as file name. In the cases when a file doesn't have a name (such as stdin, stdout, pipeline etc.), file descriptor is the usual way to access them. In unix's convension the file descriptors for standard input, standard output, standard error are 0, 1, and 2, respectively.

With the knowledge about file descriptor the above command means: execute the `find this-is-not-exist` command and redirect the file indicated by the file descriptor 2 (which is standard standard error) to `/tmp/error.txt`.

`>` implicitly redirects standard output, the following two commands are the same.

    $ ls -al 1> /tmp/list.txt
    $ ls -al > /tmp/list.txt

Besides using file descriptor to declare the redirection source, it could also be used as redirection destination:

    ls -al 2>&1 >/tmp/result.txt

The above command means: 
- execute `ls -al` 
- redirects file descriptor 2(standard error) to any destination file descriptor 1(standard output) points to 
- redirects standard output to file `/tmp/result.txt`. 

The `&` used here is to distinguish between file name `1` and file descriptor `1`.

![img of file descriptor cat]()

## Piping
Sometimes a program might write out standard output which might later be read by another program as standard input. With the above knowledge it could be written by IO redirections:

    ls -al ~ > /tmp/list.tmp
    grep '\.sh' < /tmp/list.tmp
    rm /tmp/list.tmp

As it is such a common interface between programs, unix comes with a better method - pipeline to accomplish this tasks. Pipeline sends the standard output of one program to the standard input of another program. With pipeline the above example could be rewritten as `ls -al | grep '\.sh'`.



With unix pipeline, it is possible to build one large program composed of small pieces to perform complex operations. You could: 
- do a spell check: `makewords < sentence | lowercase | sort | unique | mismatch ` see [7:00 of this video][yt-unix-intro]

- generate shell statistic stacks such as ▅▆▂▃▂▂▂▅▂▂▅▇▂▂▂▃▆▆▆▅▃▂: `curl http://earthquake.usgs.gov/earthquakes/catalogs/eqs1day-M1.txt --silent | sed '1d' | cut -d, -f9 | spark`. [check out spark](https://github.com/holman/spark)
- send a summary of a file `cat apple.txt | wc | mail -s "The count" nobody@december.com ` 
- ...

Besides the simplified syntax, pipeline has other 
Another benefit of pipeline is that all the pieces of programs start execution as soon as data stream comes in. In the first example where a temporary file is created as a medium, `grep` is not executed until `ls -al` finishes and writes to the `/tmp/list.tmp`. In this case, if the temporary data is significantly big, `grep` needs to wait quitea while. However with pipeline, `grep` is executed once `ls -al` starts to send data through standard output. 

Pipeline takes less space of disk and memory but with faster processing as all the program runs in parellel to process the data stream.


## Pipe Standard Input / Output as arguments
### Difference between standard streams and arguments
Sometimes the program returns the same result when read from standard input or execute with an arguments.

`cat log` has the same result as `cat < log`. But the mechanism is different.
`cat file` pass the file name `log` as an argument to `cat` but `cat < log` passes the content of the `log` file as standard input to `cat`.

### Process substitution

    diff <(sort file1) <(sort file2)

### `xargs`

    ls *.zip | xargs -n 1 -t unzip

## Reference
[Wikipedia Pipepine][wiki-pipepine]

[Wikipedia Standard Stream][wiki-stdstream]

[Wikipedia Process Substitution][wiki-process-sub]

[Redirection Definition][linfo-io-redirection]

[I/O Redirection in Advanced Bash-Scripting Guide][absg-io-redirection]

-----

All the icons used in illustration are from The Noun Project.
[Hard Drive][] and [Computer][] is designed by annonymous designers.
[Computer Keyboard][] is designed by [Andrew Forrester][].
[Mouse][] is designed by [Camila Bertoco][].
[Floppy Disk][] is designed by [Cor Tiemens][].

[wiki-pipepine]: http://en.wikipedia.org/wiki/Pipeline_%28Unix%29
[wiki-stdstream]: http://en.wikipedia.org/wiki/Standard_streams
[wiki-process-sub]: http://en.wikipedia.org/wiki/Process_substitution
[linfo-io-redirection]: http://www.linfo.org/redirection.html
[absg-io-redirection]: http://tldp.org/LDP/abs/html/io-redirection.html
[everything-is-a-file]: http://en.wikipedia.org/wiki/Everything_is_a_file 
[yt-unix-intro]: http://www.youtube.com/watch?v=tc4ROCJYbm0
[unix-philosophy]: http://www.faqs.org/docs/artu/ch01s06.html
[man-bash-process-sub]: http://www.gnu.org/software/bash/manual/bashref.html#Process-Substitution

[Hard Drive]: http://thenounproject.com/noun/hard-drive/#icon-No537
[Computer]: http://thenounproject.com/noun/computer/#icon-No115
[Computer Keyboard]: http://thenounproject.com/noun/computer-keyboard/#icon-No1807
[Andrew Forrester]: http://thenounproject.com/andrewforrester
[Mouse]: http://thenounproject.com/noun/mouse/#icon-No890
[Camila Bertoco]: http://thenounproject.com/cbertoco
[Floppy Disk]: http://thenounproject.com/noun/floppy-disk/#icon-No2476
[Cor Tiemens]: http://thenounproject.com/cortiemens

<a href="http://thenounproject.com/noun/building-block/#icon-No5218" target="_blank">Building Block</a> designed by <a href="http://thenounproject.com/Mikhail1986" target="_blank">Michael Rowe</a> from The Noun Project
<a href="http://thenounproject.com/noun/speaker/#icon-No4106" target="_blank">Speaker</a> designed by <a href="http://thenounproject.com/squintongreen" target="_blank">Samuel Q. Green</a> from The Noun Project
<a href="http://thenounproject.com/noun/database/#icon-No4995" target="_blank">Database</a> designed by <a href="http://thenounproject.com/DmitryBaranovskiy" target="_blank">Dmitry Baranovskiy</a> from The Noun Project
<a href="http://thenounproject.com/noun/document/#icon-No4769" target="_blank">Document</a> designed by <a href="http://thenounproject.com/mariavaragilal" target="_blank">Maria Varagilal</a> from The Noun Project
<a href="http://thenounproject.com/noun/printer/#icon-No109" target="_blank">Printer</a>  from The Noun Project
<a href="http://thenounproject.com/noun/printer/#icon-No1235" target="_blank">Printer</a> designed by <a href="http://thenounproject.com/johncaserta" target="_blank">John Caserta</a> from The Noun Project
<a href="http://thenounproject.com/noun/printer/#icon-No3751" target="_blank">Printer</a> designed by <a href="http://thenounproject.com/bitsnbobs" target="_blank">James Fenton</a> from The Noun Project
<a href="http://thenounproject.com/noun/gears/#icon-No1870" target="_blank">Gears</a> designed by <a href="http://thenounproject.com/daria" target="_blank">Dasha Shevyrenkova</a> from The Noun Project
<a href="http://thenounproject.com/noun/arrow/#icon-No2487" target="_blank">Arrow</a> designed by <a href="http://thenounproject.com/cortiemens" target="_blank">Cor Tiemens</a> from The Noun Project
