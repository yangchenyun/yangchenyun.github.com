---
layout: post
title: "ack - a better way to search in files"
date: 2012-09-02 16:58
comments: true
categories: unix, tool
published: false
---

[ack][1] became my replacement for `grep` when I need spot the lines I need within files. It wins me over through:
- A better pattern match syntax with [Perl regular expressions][perlreg]
- Smart to limit files to search in
- Much prettier output
- Config files to make customization permanent

In the following sections, I will introduce the way to use `ack` effectively and hope at the end you will love it as I am.

You could start by install `ack`:

Mac OS X with homebrew:

    brew install ack

Debian/Ubuntu

    sudo apt-get install ack-grep; sudo ln -s $(which ack-grep) /usr/local/bin/ack

Then `git clone https://github.com/twitter/bootstrap` and this will be the repository used in the examples.

## Pattern Options
First let type `ack diff`

It prints out all your lines with the string 'diff' under your current working directory.

In fact `diff` could be any valid Perl regular expression include [advanced regular expression syntax][regad] and [lookaround][reglr].

For example, if we are more interested in 'diff' as a word but not a portion in a string we could use the `ack -w diff`. `-w` will force a pattern to match a complete word.

Then if want to see if the code a function `$.proxy`, we could express it as `ack '\$\.proxy'` as `$` and `.` are needed to be escaped in regular expression. `ack` also comes with a handy way to treat the pattern as literals. use `ack -Q $.proxy` to achieve the same result. `-Q` comes in handy to filter pattern like IP addresses.

## Know where the search happens
You might notice in the above example, `ack` automatically 

By default, `ack` will search under your current working directory.

### Limit Searches in Directory

You could alter this behavior by specifying the file or directory to be searched in.  `ack 'hr.{1,2}' js/bootstrap-alert.js` will only match pattern in the `js/bootstrap-alert.js` file.
If you specify a directory name as `ack proxy docs`. It will match under the 'docs' directory recursively.

To cancel the recursive behavior, pass in `-n` or `--no-recurce`. For example `ack post docs -n` will only match pattern in the 'docs' directory but not its subdirectories.

You can also ignore directory with `--ignore-dir=name` option, `ack post --ignore-dir=docs` will seek 'post' in all directories other than 'docs'.

To see the directories before `ack` will send its searches in, we could use `-f` to only print the files to be searched.

### From STDIN
Besides reading file from arguments, `ack` also reads from STDIN which makes it a candidate in unix pipeline. We could chain multiple `ack` together to zero in on what we really care about.

`ack postError js | ack message` **first** find matches for 'postError' in all 'js' directory and **within the result** it find match for 'message'.

## Add Files to be Searched
Try `ack background less` to search files in the `less` directory. It doesn't return the result we are looking for, the `.less` files are ignored by `ack`. Why?

This is by design, quote the author Andy Lester:

>... Most codebases have a lot files in them which aren't source files, and grep wastes a lot of time searching through all of those as well and returning matches from those files.  

>That's why ack's behavior of not searching things it doesn't recognize is one of its greatest strengths: the speed you get from only searching the things that you want to be looking at.

`ack` is designed to only search files it knows. The file types known by `ack` could be seen with `ack --help-types`.  Through `ack --help-types | ack less` we found that, `*.less` is not known by `ack` and that's the reason why we couldn't get our results back.

To extend the file types we have `--type-set` and `--type-add` options.  With `ack background less --type-set less=.less`, we succeeded in searching in the `*.less` files.  

Sometimes, we don't care about file type and just want search in all files in the directory, in those cases, we use `--all-types` or `-a`. It will searched in all files regardless of its type with one exception - the CVS directory such as `.git` or `.svn`. If we also want to count them in, use the ultimate `--unrestricted` or `-u` options to search **everything** within the directory.

## Make Configuration Sticky
In most cases, we still want to limit our searches to certain types, but it will be tedious to type `--type-set` or `--type-add` everytime we want to search in files that are not supported by `ack` by default. `~/.ackrc` comes as a help. This is the configuration file which will be loaded by `ack`. All the options we introduced above could be written to it. 

Take my configuration as an example:

```shell
# Sort files by default
--sort-files

# Use smart-case by default
--smart-case

# New file types
--type-add=css=.less,.scss,.sass
--type-add=ruby=.haml
--type-set=coffee=.coffee
--type-set=markdown=.md,.markdown
--type-set=json=.json
```

There is onething to notice, instead of use whitespace in `--type-add less=.less` use `=`. Also, line begins with `#` is ignored.

After this file type registration, we could use `--type less` / `--less` to limit searches in a certain type of files. 

## More Magic and Conclusion
If the result list is too long, add `--pager=less -r` to use `less` as pager with color support.

Sometimes we care about the context of the matched line, add `-C` to display it as well.

`ack` is a handy and smart tool be used to parse text files for editing and the `man ack` comes with more explanation with the options it support. Moreoever, the coming [`ack2.0`][ack2.0] is more powerful in terms of filetype filtering and multiple config file support.

Hope this introduction makes you comfortable with `ack` now and happy hacking.

[1]: http://betterthangrep.com/
[regad]: http://www.regular-expressions.info/refadv.html
[reglr]: http://www.regular-expressions.info/lookaround.html
[perlreg]: http://perldoc.perl.org/perlre.html
[ack2.0]: http://stackoverflow.com/questions/9508431/ack-binding-an-actual-file-name-to-a-filetype#answer-9511450
