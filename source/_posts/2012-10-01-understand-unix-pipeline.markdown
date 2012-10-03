---
layout: post
title: "Understand Unix Pipeline"
date: 2012-10-01 16:54
comments: true
categories: unix
keywords: io, file, pipe, stream
published: true
---

Here are some powerful linux commands often confusing to linux beginners. 

    tail /var/log/nginx.log > /tmp/recent.log
    node server.js 2>&1 >/var/log/info.log
    ls -al | grep '/.rb' | wc -l
    tee >(wc -l >&2) < bigfile | gzip > bigfile.gz

These commands all share similar syntaxes such as `<`, `>`, `>&1` or `|` which indicates one of unix's best features - standard IO streams. Standard IO streams has been introduced since [the early days of unix][yt-unix-intro] and it forms a core part in [unix philosophy][unix-philosophy].

> ... Write programs that do one thing and do it well. Write programs to work together. Write programs to handle text streams, because that is a universal interface.

The following sections will explain IO streams of unix and their usage with programs. It will answer the following questions:
- what is standard streams and how does it work with programs?
- how to utilize standard streams in programs?
- what is pipeline and how to use it?
- how to make standard streams work with program arguments?

If your are confused about unix streams, hope following sections will help you understand the above commands; if you are already familiar with it, [references at the end][#references] might help you unleash more power of it.

* Table of Content
{:toc}

## Abstract IO from program - standard streams
Computer programs need to talk with its environment. It needs to communicate with various hardware such as disk drive, tape drive, screen, audio stereo, printer etc. 

In operating system predates Unix, program has to contain the 'knowledge' for specific devices. It need to know the right ways to talk to various devices. This bind to specific devices makes program less portable.

![img of program talks to various devices]()

Unix invented the **standard IO streams** to solve this issue. 
Firstly, it uses the data stream to abstract various ways to read and write to different devices. Data stream is an ordered sequence of data bytes. Program could read data stream until the end of a file and it could write data stream without declare the size.
Secondly, it associates input and output for program by default. It saves program's effort to establish connections to its IO devices.

Standard IO streams provide an unified input and output interfaces for the programs to communicate with various hardware devices without the growing complexities. 

![img of program talks to standard io]()

Standard IO streams include standard input, standard output and standard error. 
Standard input is data going into a program and by default is expected from keyboard. 
Standard output is the stream where program writes its data and by default prints to the terminal.
Standard error is another output stream similar to standard output. It is typically used to output error messages or diagnostics.

Standard streams follows the unix's phisolophy that ['everything is a file'][everything-is-a-file]. So from the program's point of view, this standard streams are no difference from normal files. 

### A detailed look at one command execution
With the knowledge about the standard streams, let's take a look at a normal command execution.

Type `cat` in the terminal and now the program will read the default standard input - keyboard. Type `first line` and `<Enter>`, it prints to string to the default standard output - the terminal.

    $ cat
    first line
    >first line

`<ctrl>-c` to quit the program and let's try `find this-is-not-exist`. `find` writes output to the standard error.

    $ find this-is-not-exist
    >find: this-is-not-exist: No such file or directory

The following images demonstrates data stream for the above two examples.
![img of command cat]()

## IO Redirections
Besides the default stream destination, we could also redirect them with `<` or `>`.

    $ ls -al > /tmp/list.txt
will redirect the standard output to the file `/tmp/list.txt` instead of print to terminal.

    $ less < /tmp/list.txt
will read standard input from `/tmp/list.txt` instead of the keyboard.

How about redirect standard error? We could use `2>` here.

    $ find this-is-not-exist 2> /tmp/error.txt
The previous error message is not displayed on the terminal but stored in the `/tmp/error.txt` file this time.

![img of command cat]()

The meaning of the `2` used here is **file descriptor**, they are similar to file name which is used to access file in the operating system.

As standard streams don't have file name, to access them one could use the file descriptor. The file descriptors for standard input, standard output, standard error are 0, 1, and 2, respectively.

So the above command means: execute the `find this-is-not-exist` command and redirect the file indicated by the file descriptor 2 (which is standard standard error) to `/tmp/error.txt`.

File descriptor could also be used as redirection destination:

    ls -al 2>&1 >/tmp/result.txt

This means: execute `ls -al` and redirects file descriptor 2(standard error) to any destination file descriptor 1(standard output) points to and redirects standard output to file `/tmp/result.txt`. The `&` used here is to distinguish between file name `1` and file descriptor `1`.

![img of command cat]()

`>>`


## Piping
Sometimes a program might write standard output data which might be read by another program as standard input. With the above knowledge it could be written by IO redirections:

    ls -al ~ > /tmp/list.tmp
    grep '\.sh' < /tmp/list.tmp
    rm /tmp/list.tmp

But unix comes with a better method named pipeline which sends the standard output of one program to the standard input of another program. With pipeline the above example could be rewritten as `ls -al | grep '\.sh'`.

With pipeline, it is possible to compose one giant pipeline to connect pieces of programs together to perform complex operations like `makewords sentence | lowercase | sort | unique | mismatch ` in [this example(7:00)][yt-unix-intro].

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
[Speaker][] is designed by [Okan Benn][].

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
[Speaker]: http://thenounproject.com/noun/speaker/#icon-No1327
[Okan Benn]: http://thenounproject.com/Okan%20Benn
