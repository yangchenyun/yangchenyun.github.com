---
layout: post
title: "How I Finished the DBClass from Stanford with Full Score"
date: 2013-03-24 20:44
comments: true
categories: database, study
published: true
---
* Table of Content
{:toc}

For the past ten weeks, I've been working the [DBClass][db-class] offered by Standord. With 40 hours' study I achieved [distinguished statement with full score][db-statement] on all exercises and both exams. 

In this course, I've acquired a solid knowledge about the algebra used behind the SQL language, learned advanced usage of database system and also get to know the basics to work with XML.

## The Summary of Material

You might checkout [all the exercise code][db-ex-repo] I've written for the course and some mindmaps I summarized for the reviews: [assertions and triggers][assert-mm], [constrains][constrains-mm], [transactions][transactions-mm], [XML][xml-mm].

Besides following the class instruction, I added some materials for learning and set up a more convenient development environment. The great learning experience is certainly brought by these improvements which I would like to share.

## How I Approach the Course Differently
I have approached the course differently in three aspects:

- I read all related chapters in text book.
- I studied and wrote real-world examples.
- I made deliberate effort to setup development environment to ease practice and test.

### More Reading and Practice
I followed the text book [The Database System - The Complete Book][db-book] along the course schedule. This is the text book used in former Stanford database courses and it is what the course's material origined. I read related chapters in the text book after watching the videos. Reading would reveal some points I've missed in the video and help build up a systemetic view over the topic.

As I work as a programmer, I applied the knowledge I learned in this course directly to several relational database based projects. 

I wrote a lot of [raw SQLs][raw-sql] to fetch analytic data against our production database, it helped me to understand subqueries, natural joins and aggregations in the real world.  

I substituted the backend with `sqlite` for [one command line tool][ask-repo] I was building at local hackthon.

To understand the advanced usage of triggers and constrains, I also read the source code of [squash][squash-repo], an open source tool we use internally for bug tracking.

### Tune the Tools

Another improvment I made is the development environment. I wanted to make it as easy and quick as possible to test out codes. This is the principal rule I've learned from former programming experience. Thus I made some deliberate effort to tune up the development environment.

As my major editor is `vim`, I use the database plugin [`dbext`][dbext-link] to play with SQL. `dbext` enables me to execute SQLs and get the result back in my editor. Instead of editing the SQL, doing copy-and-paste, printing the result out, I could write, run, edit, run, cleanup, run with one simple key binding. This convenience was very helpful when solving complex problems. I could start with some instinct ideas and tested them out right away and improved based on the result I got back. This fast cycle of feedback s helped to evolve one idea to one correct solution.

![dbext image][dbext-img]

Later, when I was working with XQuery, I missed the convenience I got from `dbext`. So I decided to implement it myself. Together with some [bash script][util-bin] and a [vim function][util-vim], I built the same experience for XQuery editing. One key binding would fire the XQuery expression against the xml file and return the result right in the editor.

![xquery image][xquery-img]

There is also simple [bash script][util-bin] to make working with relational algebra compiler `ra` and xslt compiler easier.

## What's next?
The [extra problems][extra-problem] included in the DBClass is well designed and is aimed to test more practical skills. I will finish them within this week.

DBclass has covered a wide range of topics in such a short period, but it also omits some concepts such as schemes, procedures, functions which are also widely used in modern databases. I will refer to the book and database system documentations to fill the gap.

Thanks for all the class2go team and Professor Jennifer Widom to carry out such an excellent course.

[db-class]: https://class2go.stanford.edu/db/Winter2013
[db-book]: http://www.amazon.com/Database-Systems-Complete-Edition-ebook/dp/B004XJIVIC/ref=sr_1_2
[squash-repo]: https://github.com/SquareSquash/web/blob/master/db/migrate/1_initial_schema.rb
[db-ex-repo]: https://github.com/yangchenyun/dbclass-exercises
[xml-mm]: /images/xml-query.png
[assert-mm]: /images/assertions-triggers.png
[constrains-mm]: /images/constrains.png
[transactions-mm]: /images/transactions.png
[raw-sql]: https://gist.github.com/yangchenyun/5254141
[extra-problem]: https://class2go.stanford.edu/db/Winter2013/pages/extra-problems
[dbext-link]: http://www.vim.org/scripts/script.php?script_id=356
[dbext-img]: /images/dbext-img.png
[xquery-img]: /images/xquery-img.png
[util-vim]: https://github.com/yangchenyun/dbclass-exercises/blob/master/utils/xquery-helper.vim
[util-bin]: https://github.com/yangchenyun/dbclass-exercises/tree/master/utils/bin
[ask-repo]: https://github.com/yangchenyun/ask/compare/cf510240b0f6f1b52d69426c4ba38227c0de2efc...master
[db-statement]: /assets/download/db-statement.pdf
