---
layout: post
title: "Ack - a better way to search in files"
date: 2012-09-02 16:58
comments: true
categories: unix, tool
keywords: ack, grep
published: true
---
Recently, [ack][1] became the replacement for `grep` when I need to spot editing point in my code bases. It wins over `grep` in terms of:

- A better pattern match syntax with [Perl regular expressions][perlreg]
- Smarter to limit searches in directories or certain file types
- Much prettier display of result
- Config files to make customization permanent

These advantages will be introduced in the the following sections and after reading it I hope you will be comfortable with it.

Let's start by installing `ack`:

Mac OS X with homebrew:

    brew install ack

Debian/Ubuntu

    sudo apt-get install ack-grep; sudo ln -s $(which ack-grep) /usr/local/bin/ack

Then `git clone https://github.com/twitter/bootstrap` to get the directory used in following examples.

* Table of Content
{:toc}

## Search with Modern Regexp Pattern
Let's start with a simple search `ack diff`.

It prints out all the files whose lines contain the string `'diff'` within the current working directory.

In fact `'diff'` could be any valid Perl Regular Expression include the [advanced regular expression syntax][regad], if you are familiar with regular expression in Ruby, Python, javaScript or Perl, you will be far more at home with the syntax than GNU Basic Regular Expression.

For example, we could use `ack 'diff\(.+\)'` to detect a string as a function call such as `'diff(o, n)''` .

It will be common that we will be more interested in `'diff'` as a word than a portion of string. In this case, we could use `ack '\bdiff\b'` but there is a handy option `-w` which force a pattern to match a complete word. So we could send our search result as `ack -w diff`.

As in any regular expression flavors, to match character such as `$` or `.`, we need to escape them in the pattern as `'\$\.proxy'`. In this case, `ack` also comes with a handy way to treat the pattern as string literal. Use `ack -Q $.proxy` to match text against liternal `'$.proxy'`. It is useful to match IP addresses in log file such as `ack aa.bb.cc.dd access.log -Q`.

## Limit Where the Search Happens
You might have already noticed that in the above example, `ack` automatically search under your current working directory.

In order to alter this behavior, you could specify the file or directory to be searched in.  `ack href js/bootstrap-alert.js` will only match pattern in the `js/bootstrap-alert.js` file.  Similarly, if you specify a directory name as `ack proxy docs`, it will search in the `'docs'` directory recursively.

To cancel the recursive behavior, pass in `-n` or `--no-recurce`. For example `ack post docs -n` will only match pattern in the `'docs'` directory but not its subdirectories.

You can also ignore directory with `--ignore-dir=name` option, `ack post --ignore-dir=docs` will seek `'post'` in all directories other than `'docs'`.

Sometimes the constrain is complicated, then the option `-f` will be helpful as it will print the files to be searched before performing the search. For example `ack --ignore-dir=docs -f` will print all the files `ack` will search in.

Besides reading from arguments for files, `ack` could also reads from STDIN. This makes `ack` a nice candidate in unix pipeline. We could chain multiple `ack` together to zero in on what the text you really care about.  For example `ack postError js | ack message` **first** find matches for `'postError'` in all `'js'` directory and **within the result** it find matches for `'message'`.

## Add Files to be Searched
Try `ack background less` to search files in the `less` directory. It doesn't return the result we are looking for, the `.less` files are ignored by `ack`. There must be something wrong with `ack`, right?

However, this behavior is not an error but is by design by its author Andy Lester:

>... Most codebases have a lot files in them which aren't source files, and grep wastes a lot of time searching through all of those as well and returning matches from those files.  

>That's why ack's behavior of not searching things it doesn't recognize is one of its greatest strengths: the speed you get from only searching the things that you want to be looking at.

`ack` is designed to only search in file types it knows. The file types known by `ack` could be seen with `ack --help-types`.  Through `ack --help-types | ack less` we found that, `*.less` is not known by `ack` and that's the reason why they are ignored.

To add more file types we use `--type-set` and `--type-add`.  With `ack background less --type-set less=.less`, we succeeded in searching in the `*.less` files.  

Sometimes, we don't care about file type and just want search in all files in the directory, in these cases, we use `--all-types` or `-a`. It will search in all files regardless of its type with one exception - the CVS directory such as `.git` or `.svn` is excluded. If we want to count them in, use the ultimate `--unrestricted` or `-u` options to search **everything** within a directory.

## Make Configuration Sticky
In most cases, we want to limit our searches to certain types, then it will be tedious to type `--type-set` or `--type-add` every time we want to search beyond build-in file types. `~/.ackrc` comes into play in these case. This is the configuration file which will be loaded by `ack`. All the options we introduced above could be written to it to make it permanent.

Take my configuration as an example:

```bash
# ~/.ackrc ack configuration file

# Sort files by default
--sort-files

# Use smart-case by default
--smart-case

# Extended File Types
--type-add=css=.less,.scss,.sass
--type-add=ruby=.haml
--type-set=coffee=.coffee
--type-set=markdown=.md,.markdown
--type-set=json=.json
```

One thing to be noticed, instead of use whitespace in `--type-add less=.less` use `=`. Also, line begins with `#` is ignored.

After adding our own file types, we could use `--type less` / `--less` to limit searches in certain file types or `--type noless` or `--noless` to exclude them. 

## More Magic and Conclusion
`ack` comes a lot options for the format of output, you could use `--pager=less -r` to use `less` as pager with color support, or use `-C` to display the lines above/below the matched line as well. For more information, check out `man ack`.

This handy tool is smarter and faster than `grep` and I have been using it heavily in my workflow.  Moreover, the coming [ack2.0][ack2.0] is more powerful in terms of file type filtering and multiple config file support.

Hope this introduction makes you comfortable with `ack`.

Do you know other "improved" substitute for daily unix utility?

[1]: http://betterthangrep.com/
[regad]: http://www.regular-expressions.info/refadv.html
[perlreg]: http://perldoc.perl.org/perlre.html
[ack2.0]: http://stackoverflow.com/questions/9508431/ack-binding-an-actual-file-name-to-a-filetype#answer-9511450
