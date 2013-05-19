---
layout: post
title: "A Plan to Read the Ruby Standard Library"
date: 2013-05-03 20:39
comments: true
categories: 
published: true
---
* Table of Content
{:toc}

## Why Bother Reading the Source Code?
For the past two years, I've built the skill to craft software with various frameworks and tools. To move forward, I need to have more control over the system, the language and the libraries I work with. Merely reading the documentation doesn't fulfil this requirement.

For any craftmanship, learning from the masterpiece is always a good way to improve one's skills. Writers make improvement by reading other writers' work, so should us - coders.

Inspired by [this][read-stdlib] [two][code-reading] articles, I decided to read all the source code of ruby standard libraries in the next four months.

It might be hard to begin with - lots of unknown idioms, unknown concepts and large code bases - but if I survive, I will improve the ability to dive into the source code directly and make sense out of it. This skill will help me become a better programmer in the future.

## Resources

Most of the studies will take place in the [`ruby 1.9.3`][ruby-1-9-3] source code.  I decided to read 1.9.3 instead of 2.0 because most of my work still runs in 1.9.3 that and it would be living for a while. It would be good I could adapt what I've learned into my work quickly.

I decided to leave out some of the ['default gems'][default-gem] in the ruby trunk as they have either an indenpedent or a large code base and deserve dedicate attention in the future. So are some domain-specific parser or gui libraries such as `syck`, `psych`, `rexml`, `rss`, `tk` and `tkextlib`.

Along the way, [`ruby-spec`][ruby-spec] and [Programming Ruby][programming-ruby] will be the guide book to help me dive in the source code.

## Work of Load
Leaving out the complex libraries or gems, there are **92 libraries** I need to study in. 

I calculated the total lines of code I needed to go through...

    cloc --not-match-f='^(tk|psych).+' --exclude-dir=psych,rake,rubygems,rdoc,rexml,tk,tkextlib,rss .

...and get 287 files, 51770 locs, 15160 effective lines of code(loc).

    287 text files.
    287 unique files.
    731 files ignored.

    http://cloc.sourceforge.net v 1.56  T=2.0 s (141.5 files/s, 44190.0 lines/s)
    -------------------------------------------------------------------------------
    Language                     files          blank        comment           code
    -------------------------------------------------------------------------------
    Ruby                           283           9649          26961          51770
    -------------------------------------------------------------------------------
    SUM:                           283           9649          26961          51770
    -------------------------------------------------------------------------------

To be more specific, I want to know every files I also listed all the files I needed to go through with:

    tree -I 'x86_64-darwin12.3.0|psych*|rake|rubygems|rdoc|rexml|tk*|rss' --prune

...and it come out with [this list](#track-of-my-studies).

## Goals and Plans
I want to finish this study in 120 days, from `Fri, 03 May 2013` - `Sat, 31 Aug 2013`.  That means I need to go through around 450 locs every day, might take me 1 to 2 hours to finish this work.
   
Reading is also about quality and understanding, so for each library I will:

- write some example code to use this library
- do researches about unique techniques I found within the source code

All learning progress will be recorded here.

## Track of My Studies

    [x] ├── English.rb
    [x] ├── abbrev.rb
    [ ] ├── base64.rb
    [ ] ├── benchmark.rb
    [x] ├── bigdecimal
    [x] │   ├── jacobian.rb
    [x] │   ├── ludcmp.rb
    [x] │   ├── math.rb
    [x] │   ├── newton.rb
    [x] │   └── util.rb
    [ ] ├── cgi
    [ ] │   ├── cookie.rb
    [ ] │   ├── core.rb
    [ ] │   ├── html.rb
    [ ] │   ├── session
    [ ] │   │   └── pstore.rb
    [ ] │   ├── session.rb
    [ ] │   └── util.rb
    [ ] ├── cgi.rb
    [ ] ├── cmath.rb
    [ ] ├── complex.rb
    [ ] ├── count
    [ ] ├── csv.rb
    [ ] ├── date
    [ ] │   └── format.rb
    [ ] ├── date.rb
    [ ] ├── debug.rb
    [ ] ├── delegate.rb
    [ ] ├── digest
    [ ] │   ├── hmac.rb
    [ ] │   └── sha2.rb
    [ ] ├── digest.rb
    [ ] ├── dl
    [ ] │   ├── callback.rb
    [ ] │   ├── cparser.rb
    [ ] │   ├── func.rb
    [ ] │   ├── import.rb
    [ ] │   ├── pack.rb
    [ ] │   ├── stack.rb
    [ ] │   ├── struct.rb
    [ ] │   ├── types.rb
    [ ] │   └── value.rb
    [ ] ├── dl.rb
    [ ] ├── drb
    [ ] │   ├── acl.rb
    [ ] │   ├── drb.rb
    [ ] │   ├── eq.rb
    [ ] │   ├── extserv.rb
    [ ] │   ├── extservm.rb
    [ ] │   ├── gw.rb
    [ ] │   ├── invokemethod.rb
    [ ] │   ├── observer.rb
    [ ] │   ├── ssl.rb
    [ ] │   ├── timeridconv.rb
    [ ] │   └── unix.rb
    [ ] ├── drb.rb
    [ ] ├── e2mmap.rb
    [ ] ├── erb.rb
    [ ] ├── expect.rb
    [ ] ├── fiddle
    [ ] │   ├── closure.rb
    [ ] │   └── function.rb
    [ ] ├── fiddle.rb
    [ ] ├── fileutils.rb
    [ ] ├── find.rb
    [x] ├── forwardable.rb
    [ ] ├── getoptlong.rb
    [ ] ├── gserver.rb
    [ ] ├── io
    [ ] │   └── console
    [ ] │       └── size.rb
    [ ] ├── ipaddr.rb
    [ ] ├── irb
    [ ] │   ├── cmd
    [ ] │   │   ├── chws.rb
    [ ] │   │   ├── fork.rb
    [ ] │   │   ├── help.rb
    [ ] │   │   ├── load.rb
    [ ] │   │   ├── nop.rb
    [ ] │   │   ├── pushws.rb
    [ ] │   │   └── subirb.rb
    [ ] │   ├── completion.rb
    [ ] │   ├── context.rb
    [ ] │   ├── ext
    [ ] │   │   ├── change-ws.rb
    [ ] │   │   ├── history.rb
    [ ] │   │   ├── loader.rb
    [ ] │   │   ├── math-mode.rb
    [ ] │   │   ├── multi-irb.rb
    [ ] │   │   ├── save-history.rb
    [ ] │   │   ├── tracer.rb
    [ ] │   │   ├── use-loader.rb
    [ ] │   │   └── workspaces.rb
    [ ] │   ├── extend-command.rb
    [ ] │   ├── frame.rb
    [ ] │   ├── help.rb
    [ ] │   ├── init.rb
    [ ] │   ├── input-method.rb
    [ ] │   ├── inspector.rb
    [ ] │   ├── lc
    [ ] │   │   ├── error.rb
    [ ] │   │   ├── help-message
    [ ] │   │   └── ja
    [ ] │   │       ├── encoding_aliases.rb
    [ ] │   │       ├── error.rb
    [ ] │   │       └── help-message
    [ ] │   ├── locale.rb
    [ ] │   ├── magic-file.rb
    [ ] │   ├── notifier.rb
    [ ] │   ├── output-method.rb
    [ ] │   ├── ruby-lex.rb
    [ ] │   ├── ruby-token.rb
    [ ] │   ├── slex.rb
    [ ] │   ├── src_encoding.rb
    [ ] │   ├── version.rb
    [ ] │   ├── workspace.rb
    [ ] │   ├── ws-for-case-2.rb
    [ ] │   └── xmp.rb
    [ ] ├── irb.rb
    [ ] ├── json
    [ ] │   ├── add
    [ ] │   │   ├── complex.rb
    [ ] │   │   ├── core.rb
    [ ] │   │   └── rational.rb
    [ ] │   ├── common.rb
    [ ] │   ├── ext.rb
    [ ] │   └── version.rb
    [ ] ├── json.rb
    [ ] ├── kconv.rb
    [ ] ├── logger.rb
    [ ] ├── mathn.rb
    [ ] ├── matrix
    [ ] │   ├── eigenvalue_decomposition.rb
    [ ] │   └── lup_decomposition.rb
    [ ] ├── matrix.rb
    [ ] ├── minitest
    [ ] │   ├── autorun.rb
    [ ] │   ├── benchmark.rb
    [ ] │   ├── mock.rb
    [ ] │   ├── pride.rb
    [ ] │   ├── spec.rb
    [ ] │   └── unit.rb
    [ ] ├── mkmf.rb
    [ ] ├── monitor.rb
    [ ] ├── multi-tk.rb
    [ ] ├── mutex_m.rb
    [ ] ├── net
    [ ] │   ├── ftp.rb
    [ ] │   ├── http.rb
    [ ] │   ├── https.rb
    [ ] │   ├── imap.rb
    [ ] │   ├── pop.rb
    [ ] │   ├── protocol.rb
    [ ] │   ├── smtp.rb
    [ ] │   └── telnet.rb
    [ ] ├── observer.rb
    [ ] ├── open-uri.rb
    [ ] ├── open3.rb
    [ ] ├── openssl
    [ ] │   ├── bn.rb
    [ ] │   ├── buffering.rb
    [ ] │   ├── cipher.rb
    [ ] │   ├── config.rb
    [ ] │   ├── digest.rb
    [ ] │   ├── ssl-internal.rb
    [ ] │   ├── ssl.rb
    [ ] │   ├── x509-internal.rb
    [ ] │   └── x509.rb
    [ ] ├── openssl.rb
    [ ] ├── optparse
    [ ] │   ├── date.rb
    [ ] │   ├── shellwords.rb
    [ ] │   ├── time.rb
    [ ] │   ├── uri.rb
    [ ] │   └── version.rb
    [ ] ├── optparse.rb
    [ ] ├── ostruct.rb
    [ ] ├── pathname.rb
    [ ] ├── pp.rb
    [ ] ├── prettyprint.rb
    [ ] ├── prime.rb
    [ ] ├── profile.rb
    [ ] ├── profiler.rb
    [x] ├── pstore.rb
    [ ] ├── racc
    [ ] │   └── parser.rb
    [ ] ├── rake.rb
    [ ] ├── rational.rb
    [ ] ├── rbconfig
    [ ] │   ├── datadir.rb
    [ ] │   └── obsolete.rb
    [ ] ├── rdoc.rb
    [ ] ├── remote-tk.rb
    [ ] ├── resolv-replace.rb
    [ ] ├── resolv.rb
    [ ] ├── rinda
    [ ] │   ├── rinda.rb
    [ ] │   ├── ring.rb
    [ ] │   └── tuplespace.rb
    [ ] ├── ripper
    [ ] │   ├── core.rb
    [ ] │   ├── filter.rb
    [ ] │   ├── lexer.rb
    [ ] │   └── sexp.rb
    [ ] ├── ripper.rb
    [ ] ├── rss.rb
    [ ] ├── rubygems.rb
    [ ] ├── scanf.rb
    [ ] ├── securerandom.rb
    [ ] ├── set.rb
    [ ] ├── shell
    [ ] │   ├── builtin-command.rb
    [ ] │   ├── command-processor.rb
    [ ] │   ├── error.rb
    [ ] │   ├── filter.rb
    [ ] │   ├── process-controller.rb
    [ ] │   ├── system-command.rb
    [ ] │   └── version.rb
    [ ] ├── shell.rb
    [ ] ├── shellwords.rb
    [ ] ├── singleton.rb
    [ ] ├── socket.rb
    [ ] ├── sync.rb
    [ ] ├── tags
    [ ] ├── tcltk.rb
    [ ] ├── tempfile.rb
    [ ] ├── test
    [ ] │   ├── unit
    [ ] │   │   ├── assertions.rb
    [ ] │   │   ├── parallel.rb
    [ ] │   │   └── testcase.rb
    [ ] │   └── unit.rb
    [ ] ├── thread.rb
    [ ] ├── thwait.rb
    [ ] ├── time.rb
    [ ] ├── timeout.rb
    [ ] ├── tmpdir.rb
    [ ] ├── tracer.rb
    [ ] ├── tsort.rb
    [ ] ├── ubygems.rb
    [ ] ├── un.rb
    [ ] ├── uri
    [ ] │   ├── common.rb
    [ ] │   ├── ftp.rb
    [ ] │   ├── generic.rb
    [ ] │   ├── http.rb
    [ ] │   ├── https.rb
    [ ] │   ├── ldap.rb
    [ ] │   ├── ldaps.rb
    [ ] │   └── mailto.rb
    [ ] ├── uri.rb
    [ ] ├── weakref.rb
    [ ] ├── webrick
    [ ] │   ├── accesslog.rb
    [ ] │   ├── cgi.rb
    [ ] │   ├── compat.rb
    [ ] │   ├── config.rb
    [ ] │   ├── cookie.rb
    [ ] │   ├── htmlutils.rb
    [ ] │   ├── httpauth
    [ ] │   │   ├── authenticator.rb
    [ ] │   │   ├── basicauth.rb
    [ ] │   │   ├── digestauth.rb
    [ ] │   │   ├── htdigest.rb
    [ ] │   │   ├── htgroup.rb
    [ ] │   │   ├── htpasswd.rb
    [ ] │   │   └── userdb.rb
    [ ] │   ├── httpauth.rb
    [ ] │   ├── httpproxy.rb
    [ ] │   ├── httprequest.rb
    [ ] │   ├── httpresponse.rb
    [ ] │   ├── https.rb
    [ ] │   ├── httpserver.rb
    [ ] │   ├── httpservlet
    [ ] │   │   ├── abstract.rb
    [ ] │   │   ├── cgi_runner.rb
    [ ] │   │   ├── cgihandler.rb
    [ ] │   │   ├── erbhandler.rb
    [ ] │   │   ├── filehandler.rb
    [ ] │   │   └── prochandler.rb
    [ ] │   ├── httpservlet.rb
    [ ] │   ├── httpstatus.rb
    [ ] │   ├── httputils.rb
    [ ] │   ├── httpversion.rb
    [ ] │   ├── log.rb
    [ ] │   ├── server.rb
    [ ] │   ├── ssl.rb
    [ ] │   ├── utils.rb
    [ ] │   └── version.rb
    [ ] ├── webrick.rb
    [ ] ├── xmlrpc
    [ ] │   ├── base64.rb
    [ ] │   ├── client.rb
    [ ] │   ├── config.rb
    [ ] │   ├── create.rb
    [ ] │   ├── datetime.rb
    [ ] │   ├── httpserver.rb
    [ ] │   ├── marshal.rb
    [ ] │   ├── parser.rb
    [ ] │   ├── server.rb
    [ ] │   └── utils.rb
    [ ] ├── yaml
    [ ] │   ├── dbm.rb
    [ ] │   └── store.rb
    [x] └── yaml.rb


[well-ground-ruby]: http://www.amazon.com/The-Well-Grounded-Rubyist-David-Black/dp/1933988657
[programming-ruby]: http://www.amazon.com/Programming-Ruby-1-9-Pragmatic-Programmers/dp/1934356085
[ruby-spec]: https://github.com/rubyspec/rubyspec
[ruby-1-9-3]: https://github.com/ruby/ruby/tree/ruby_1_9_3
[default-gem]: https://bugs.ruby-lang.org/projects/ruby/wiki/StdlibGem
[read-stdlib]: http://blog.rubybestpractices.com/posts/gregory/005-code-reading-stdlib.html
[code-reading]: http://on-ruby.blogspot.jp/2009/05/questions-five-ways-code-reading.html
