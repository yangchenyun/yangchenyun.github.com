---
layout: post
title: "Use spy, stub and mock in tests"
date: 2012-07-29 16:51
comments: true
keywords: test, spy, stub, mock
categories: geek
published: true

---

This article will share some lessons I learned through writing tests at work. It will explain why doing unit-test and shed light on the confusing concepts of **spy**, **stub** and **mock** and how to apply them in order to meet the goal in a unit-test.

The concept itself is explained in general and examples will be written in `coffeescript` with `SinonJS`.

* Table of Content
{:toc}

## What unit test?
Spy, stub and mock are all created to accomplish the goal of unit test - *"to isolate each part of the program and show that the individual parts are correct."*[^1]

Isolation distinguishes unit tests from other test methdology. 

For example, in integration testing, one could prepare a collection of HTTP requests with different scenarios and assert the expected HTML/JSON in the HTTP responses. The test takes the application as whole and tests about overall output with certain input. This method closely mimic the end user behavior but it has several disadvantages.

First, when the application grows more complicated, it is costly to cover all the usage scenarios. Because every components in an application contains several alternative paths, when chaining these paths together, the alternatives will be muptipled. 

Second, the complexity of the application make it is hard to zero in on exceptions.

For example, you have this calculator under test, it takes more than two arguments and the calculator will invoke one of another four algebra functions according to the first arguments. 

{% gist 3199164 %}

For the first arguments, you have five variations ('muptiple', 'addition', 'minus', 'divide' and others), for the second arguments you have three variations(<=1, 2, >2), and in total you get 15 test cases if you use the integration testing approach.

Moreover, there are five possible exceptions might be thrown out of the system.

In contrast, if we take the unit test approach, there are 5 test cases for the `calculator`, 2 test cases for each individual algebra functions, in total we will have 13 to cover our application. Moreover, each test will throw their own exception which will be easier to detect bugs in the application code.

Unit test helps to reduce the numbers of test cases and detect buggy parts of application. This benefit will be more significant if the application has a more complex architecture and has more input variations. 

However, we need to **isolate the behavior** of individual module to adapt unit test in our code. When we invoke `calculator()` we only care about its behavior, we don't want to invoke other functions as well. How could we archieve that? 

There is where those spy, stub and mock come into play.

## What are those…spys?
spy, stub and mock are all **test doubles**, they are **objects have the same or parts of the original object' APIs and doesn't necessary behave the same**. 

### Spy, Slient Spy
Test spy is silent, they don't disturb the running of the application. They just stand in the corner and record everything about the object they are ordered to spy on. They are so careful that they could remember whether the object is invoked or not, how many arguments are passed in and what arguments they are.

Here we use a spy to test the calculator when some invalid methods name are passed in. Notice we don't interrupt with the execution of the code but assert the behaviors of the `calculator` function at last.

{% gist 3199577 %}

We could do the same for 'multiple' method:

{% gist 3199633 %}

This time with the careful work of the spy, we could not only assert the invocation of the `multiple` method, but also assert the times it is called and the parameters it is passed in.

With a little refactoring, we could test all the five possible paths for the `calculator()` method based the above two pieces of testings. 

{% gist 3199697 %}

We use mocha's `before` helper to create spies for all our methods and we share this spies through `@`(`this`) across every tests. 

The spy is so dictated to its work that we could assert individual method behaviors. But it comes with a caveat, look the following test:

{% gist 3199904 %}

the following test fail with an exception `Error: minus takes only two operands`. This exception is thrown by the `minus()` method. But we are testing `calculator()` now, as long as it calls the `minus()` method correctly, we shouldn't blame it for exception thrown by other functions. 

Thus we need to **isolate `calculator()` from `minus()`**, that's what stub and mock is good at.

### Stubs, they are dummy objects.
**Stubs are objects with pre-programmed behavior.** Unlike spy, it not only observes the methods but also **alter the output of the method**. In order to isolate the `minus()` method from the `calculator()` method, we need to shutdown the `minus()` method temporarily. 

{% gist 3200012 %} 

This tests pass because the actual `minus()` method is "stubbed" now, and `calculator` behaves exactly as we expect - call the right method, pass the parameters in an array.

One thing to note for the last line, we need to **restore** this method to its original behavior after test.


### Mocks
Sometimes, we want more than stubs, we **expect** stubs also have certain behaviors. Add preprogrammed behavior expectation to stubs, we have mocks. Mocks are be viewed as an combination of stubs and assertions. Mocks observe the method behavior as a spy, re-program method behavior as a stub and verify method behavior when it gets called.

Here is our test rewritten with mocks:

{% gist 3200122 %} 

With mocks, we also change the structure of our test cases, we move up our assertion to the top of each test cases.

[Some Summary]

## Further Readings
- [xUnit Patterns][1] has a table summaries the difference between spies, stubs and mocks. 
- CJ has written a very detailed [book][3] with step-by-step examples about testing in `javascript`.
- The [Sinon.JS][2] Doc he creates is also documented very well.

[^1]: http://en.wikipedia.org/wiki/Unit_tests
[1]: http://xunitpatterns.com/Mocks,%20Fakes,%20Stubs%20and%20Dummies.html
[2]: http://cjohansen.no/en/javascript/javascript_test_spies_stubs_and_mocks
[3]: http://www.amazon.com/Test-Driven-JavaScript-Development-Developers-Library/dp/0321683919/ref=sr_1_1?ie=UTF8&qid=1343580825&sr=8-1&keywords=test+driven+javascript
