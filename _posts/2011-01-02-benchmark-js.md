---
layout: default
title: Benchmark
---

# Benchmark.js

A [robust](http://calendar.perfplanet.com/2010/bulletproof-javascript-benchmarks/ "Bulletproof JavaScript benchmarks") benchmarking library that works on nearly all JavaScript platforms, supports high-resolution timers, and returns statistically significant results. As seen on [jsPerf](http://jsperf.com/).

## Installation

``` bash
git clone --recursive https://github.com/bestiejs/benchmark.js.git
cd benchmark.js
```

In a browser or Adobe AIR:

``` html
<script src="benchmark.js"></script>
```

Optionally, expose Java’s nanosecond timer by adding the `nano` applet to the `<body>`:

``` html
<applet code="nano" archive="nano.jar"></applet>
```

Or enable Chrome’s microsecond timer by using the [command line switch](http://peter.sh/experiments/chromium-command-line-switches/#enable-benchmarking):

``` bash
--enable-benchmarking
```

Via [npm](http://npmjs.org/):

``` bash
npm install benchmark
```

In [Node.js](http://nodejs.org/) and [RingoJS v0.8.0+](http://ringojs.org/):

``` js
var Benchmark = require('benchmark');
```

Optionally, use the [microtime module](https://github.com/wadey/node-microtime) by Wade Simmons:

``` bash
npm install microtime
```

In [Narwhal](http://narwhaljs.org/) and [RingoJS v0.7.0-](http://ringojs.org/):

``` js
var Benchmark = require('benchmark').Benchmark;
```

In [Rhino](http://www.mozilla.org/rhino/):

``` js
load('benchmark.js');
```

In [RequireJS](http://requirejs.org/):

{% highlight javascript %}

require({
  'paths': {
    'benchmark': 'path/to/benchmark'
  }
},
['benchmark'], function(Benchmark) {
  console.log(Benchmark.version);
});

// or with platform.js
// https://github.com/bestiejs/platform.js
require({
  'paths': {
    'benchmark': 'path/to/benchmark',
    'platform': 'path/to/platform'
  }
},
['benchmark', 'platform'], function(Benchmark, platform) {
  Benchmark.platform = platform;
  console.log(Benchmark.platform.name);
});

{% endhighlight %}


## Usage

``` js
var suite = new Benchmark.Suite;

// add tests
suite.add('RegExp#test', function() {
  /o/.test('Hello World!');
})
.add('String#indexOf', function() {
  'Hello World!'.indexOf('o') > -1;
})
// add listeners
.on('cycle', function(event, bench) {
  console.log(String(bench));
})
.on('complete', function() {
  console.log('Fastest is ' + this.filter('fastest').pluck('name'));
})
// run async
.run({ 'async': true });

// logs:
// > RegExp#test x 4,161,532 +-0.99% (59 cycles)
// > String#indexOf x 6,139,623 +-1.00% (131 cycles)
// > Fastest is String#indexOf
```
