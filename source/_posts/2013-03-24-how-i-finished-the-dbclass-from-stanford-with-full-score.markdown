---
layout: post
title: "How I Finished the DBClass from Stanford with full score"
date: 2013-03-24 20:44
comments: true
categories: 
published: false
---
* Table of Content
{:toc}

After submitting the final exam button, there came the end of the 10 weeks' studying of [DBClass online courses][db-class] offered by Standord. 

In this period, I used 3 - 4 hours per week to follow the schedule and at last archieved full score on both the required and optional exercises and both the middle-term and final exams. Check out all the code I wrote in this courses: [relational algebra exercises][ra-ex], [sql related exercises][sql-ex], [xml related exercises][xml-ex].

Moreover, I also got a systematic understanding of the relational database system and I could apply the knowledge I acquired into my work easily.

I'd like to share how I approach the studies required by this course and what I've summarized from this courses.

## How I Approach the Course Differently
I have approached the course differently in three aspects:

- I made a discipline effort to read related chapters in text book.
- I made deliberate effort to setup development environment to ease practices.
- I tried to study real-world examples.

I follow the text book [<The Database System - The Complete Book>][db-book] along the course schedule. This was the text book used in former Stanford DB courses and it is what the course's materials are sourced. I read the related chapters in the text book after watching the videos. Reading reveals details I would ignore in the video and helps form a systemetic view over the topic.

One of the principal rule I applied when learning programming is that to practice help speed up the understanding, an easier practice environment will help the learning progress. Thus I made some deliberate effort to setup and tune the development environment to make it as easy as possible to run examples, test new ideas and work on exercises. I will cover the details of the setup [later][db-setup] in this article.

As I was working on several relational database based projects while studying the course, I applied the knowledge I learned in this course. I wrote a lot of raw SQLs to fetch analytic data against our production database and I read the source code of about the database migrations in [squash][squash-repo].

## What I Have Learned
### A Solid Understanding over SQL
At work, I mainly use ORMs such as [ActiveRecord][ar-repo] to access a relational database. Besides working with the library API, I don't know too much details about the underlying running SQLs. It is impossible for me to tune up the performance of certain SQL or structure complex query against the database.

In this course, I was guided through an enlightening experience. I was introduced the relational algebra algorithm, which is the basic idea behind the SQL language. Then, set operations, joins and subqueries are introduced to structure complex SQL step by step.

### Advanced Usage of SQL

[mindmap summary for assertions and triggers][assert-mm]
[mindmap summary for constrains][constrains-mm]
[mindmap summary for transactions][transactions-mm]

### Working with XML

[mindmap summary for XML][xml-mm]

## Tools I Used to Ease Practices
dbext
bash-wrapper
vim-util inspired by dbext


## What's next?
Other uncovered topics: modules, schemes, tablespaces, procedures.
Extra problems

[db-class]: http://
[db-book]: http://
[db-setup]: http://
[site-huali]: 
[squash-repo]:
[ra-ex]:
[sql-ex]:
[xml-ex]:
[ar-repo]:
[xml-mm]: /images/xml-query.png
[assert-mm]: /images/assertions-triggers.png
[constrains-mm]: /images/constrains.png
[transactions-mm]: /images/transactions.png
