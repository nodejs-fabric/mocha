---
layout: default
title: 'Mocha - the fun, simple, flexible JavaScript test framework'
---
Mocha is a feature-rich JavaScript test framework running on [Node.js](https://nodejs.org) and in the browser, making asynchronous testing *simple* and *fun*. Mocha tests run serially, allowing for flexible and accurate reporting, while mapping uncaught exceptions to the correct test cases. Hosted on [GitHub](https://github.com/mochajs/mocha).

<nav class="badges">
  <a href="https://gitter.im/mochajs/mocha"><img src="/images/join-chat.svg" alt="Gitter"></a>
  <a href="#backers"><img src="//opencollective.com/mochajs/backers/badge.svg" alt="OpenCollective backers"></a>
  <a href="#sponsors"><img src="//opencollective.com/mochajs/sponsors/badge.svg" alt="OpenCollective sponsors"></a>
</nav>

{% include backers.md %}
{% include sponsors.md %}

## Features

- [browser support](#running-mocha-in-the-browser)
- [simple async support, including promises](#asynchronous-code)
- [test coverage reporting](#wallabyjs)
- [string diff support](#diffs)
- [javascript API for running tests](#more-information)
- proper exit status for CI support etc
- [auto-detects and disables coloring for non-ttys](#reporters)
- [maps uncaught exceptions to the correct test case](#browser-specific-methods)
- [async test timeout support](#delayed-root-suite)
- [test retry support](#retry-tests)
- [test-specific timeouts](#test-level)
- [growl notification support](#mochaopts)
- [reports test durations](#test-duration)
- [highlights slow tests](#dot-matrix)
- [file watcher support](#min)
- [global variable leak detection](#--check-leaks)
- [optionally run tests that match a regexp](#-g---grep-pattern)
- [auto-exit to prevent "hanging" with an active loop](#--exit----no-exit)
- [easily meta-generate suites](#markdown) & [test-cases](#list)
- [mocha.opts file support](#mochaopts)
- clickable suite titles to filter test execution
- [node debugger support](#-d---debug)
- [detects multiple calls to `done()`](#detects-multiple-calls-to-done)
- [use any assertion library you want](#assertions)
- [extensible reporting, bundled with 9+ reporters](#reporters)
- [extensible test DSLs or "interfaces"](#interfaces)
- [before, after, before each, after each hooks](#hooks)
- [arbitrary transpiler support (coffee-script etc)](#--compilers)
- [TextMate bundle](#textmate)
- [and more!](https://mochajs.org)

## Table of Contents

<!-- AUTO-GENERATED-CONTENT:START (toc:maxdepth=2&bullets=-) -->

- [Installation](#installation)
- [Getting Started](#getting-started)
- [Run Cycle Overview](#run-cycle-overview)
- [Detects Multiple Calls to `done()`](#detects-multiple-calls-to-done)
- [Assertions](#assertions)
- [Asynchronous Code](#asynchronous-code)
- [Synchronous Code](#synchronous-code)
- [Arrow Functions](#arrow-functions)
- [Hooks](#hooks)
- [Pending Tests](#pending-tests)
- [Exclusive Tests](#exclusive-tests)
- [Inclusive Tests](#inclusive-tests)
- [Retry Tests](#retry-tests)
- [Dynamically Generating Tests](#dynamically-generating-tests)
- [Timeouts](#timeouts)
- [Diffs](#diffs)
- [Command-Line Usage](#command-line-usage)
- [Interfaces](#interfaces)
- [Reporters](#reporters)
- [Running Mocha in the Browser](#running-mocha-in-the-browser)
- [Configuring Mocha (Node.js)](#configuring-mocha-nodejs)
- [`mocha.opts`](#mochaopts)
- [The `test/` Directory](#the-test-directory)
- [Editor Plugins](#editor-plugins)
- [Examples](#examples)
- [Testing Mocha](#testing-mocha)
- [More Information](#more-information)

<!-- AUTO-GENERATED-CONTENT:END -->

## Installation

Install with [npm](https://npmjs.org) globally:

```sh
$ npm install --global mocha
```

or as a development dependency for your project:

```sh
$ npm install --save-dev mocha
```

> As of v6.0.0, Mocha requires Node.js v6.0.0 or newer.

## Getting Started

```sh
$ npm install mocha
$ mkdir test
$ $EDITOR test/test.js # or open with your favorite editor
```

In your editor:

```js
var assert = require('assert');
describe('Array', function() {
  describe('#indexOf()', function() {
    it('should return -1 when the value is not present', function() {
      assert.equal([1,2,3].indexOf(4), -1);
    });
  });
});
```

Back in the terminal:

```sh
$ ./node_modules/mocha/bin/mocha

  Array
    #indexOf()
      ✓ should return -1 when the value is not present


  1 passing (9ms)
```

Set up a test script in package.json:

```json
"scripts": {
    "test": "mocha"
  }
```

Then run tests with:

```sh
$ npm test
```

## Run Cycle Overview

A brief outline on the order Mocha's components are executed.
Worth noting that all hooks, `describe` and `it` callbacks are run in the order they are defined (i.e. found in the file).

``` js
run 'mocha spec.js'
|
spawn child process
|
|--------------> inside child process
  process and apply options
  |
  run spec file/s
  |
  |--------------> per spec file
    suite callbacks (e.g., 'describe')
    |
    'before' root-level pre-hook
    |
    'before' pre-hook
    |
    |--------------> per test
      'beforeEach' root-level pre-hook
      |
      'beforeEach' pre-hook
      |
      test callbacks (e.g., 'it')
      |
      'afterEach' post-hook
      |
      'afterEach' root-level post-hook
    |<-------------- per test end
    |
    'after' post-hook
    |
    'after' root-level post-hooks
  |<-------------- per spec file end
|<-------------- inside child process end
```

## Detects Multiple Calls to `done()`

If you use callback-based async tests, Mocha will throw an error if `done()` is called multiple times. This is handy for catching accidental double callbacks.

```javascript
it('double done', function(done) {
  // Calling `done()` twice is an error
  setImmediate(done);
  setImmediate(done);
});
```

Running the above test will give you the below error message:

```sh
$ ./node_modules/.bin/mocha mocha.test.js


  ✓ double done
  1) double done

  1 passing (6ms)
  1 failing

  1) double done:
     Error: done() called multiple times
      at Object.<anonymous> (mocha.test.js:1:63)
      at require (internal/module.js:11:18)
      at Array.forEach (<anonymous>)
      at startup (bootstrap_node.js:187:16)
      at bootstrap_node.js:608:3
```

## Assertions

Mocha allows you to use any assertion library you wish.  In the above example, we're using Node.js' built-in [assert](https://nodejs.org/api/assert.html) module--but generally, if it throws an `Error`, it will work! This means you can use libraries such as:

- [should.js](https://github.com/shouldjs/should.js) - BDD style shown throughout these docs
- [expect.js](https://github.com/LearnBoost/expect.js) - `expect()` style assertions
- [chai](http://chaijs.com/) - `expect()`, `assert()` and `should`-style assertions
- [better-assert](https://github.com/visionmedia/better-assert) -  C-style self-documenting `assert()`
- [unexpected](http://unexpected.js.org) - "the extensible BDD assertion toolkit"

## Asynchronous Code

Testing asynchronous code with Mocha could not be simpler! Simply invoke the callback when your test is complete. By adding a callback (usually named `done`) to `it()`, Mocha will know that it should wait for this function to be called to complete the test. This callback accepts both an `Error` instance (or subclass thereof) *or* a falsy value; anything else will cause a failed test.

```js
describe('User', function() {
  describe('#save()', function() {
    it('should save without error', function(done) {
      var user = new User('Luna');
      user.save(function(err) {
        if (err) done(err);
        else done();
      });
    });
  });
});
```

To make things even easier, the `done()` callback also accepts an `Error` instance (i.e. `new Error()`), so we may use this directly:

```js
describe('User', function() {
  describe('#save()', function() {
    it('should save without error', function(done) {
      var user = new User('Luna');
      user.save(done);
    });
  });
});
```

### Working with Promises

Alternately, instead of using the `done()` callback, you may return a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise). This is useful if the APIs you are testing return promises instead of taking callbacks:

```js
beforeEach(function() {
  return db.clear()
    .then(function() {
      return db.save([tobi, loki, jane]);
    });
});

describe('#find()', function() {
  it('respond with matching records', function() {
    return db.find({ type: 'User' }).should.eventually.have.length(3);
  });
});
```

> The latter example uses [Chai as Promised](https://www.npmjs.com/package/chai-as-promised) for fluent promise assertions.

In Mocha v3.0.0 and newer, returning a `Promise` *and* calling `done()` will result in an exception, as this is generally a mistake:

```js
const assert = require('assert');

it('should complete this test', function (done) {
  return new Promise(function (resolve) {
    assert.ok(true);
    resolve();
  })
    .then(done);
});
```

The above test will fail with `Error: Resolution method is overspecified. Specify a callback *or* return a Promise; not both.`.  In versions older than v3.0.0, the call to `done()` is effectively ignored.

### Using async / await

If your JS environment supports [async / await](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/async_function) you can also write asynchronous tests like this:

```js
beforeEach(async function() {
  await db.clear();
  await db.save([tobi, loki, jane]);
});

describe('#find()', function() {
  it('responds with matching records', async function() {
    const users = await db.find({ type: 'User' });
    users.should.have.length(3);
  });
});
```

## Synchronous Code

When testing synchronous code, omit the callback and Mocha will automatically continue on to the next test.

```js
describe('Array', function() {
  describe('#indexOf()', function() {
    it('should return -1 when the value is not present', function() {
      [1,2,3].indexOf(5).should.equal(-1);
      [1,2,3].indexOf(0).should.equal(-1);
    });
  });
});
```

## Arrow Functions

Passing [arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) ("lambdas") to Mocha is discouraged.  Lambdas lexically bind `this` and cannot access the Mocha context.  For example, the following code will fail:

```js
describe('my suite', () => {
  it('my test', () => {
    // should set the timeout of this test to 1000 ms; instead will fail
    this.timeout(1000);
    assert.ok(true);
  });
});
```

*If you do not need to use* Mocha's context, lambdas should work. However, the result will be more difficult to refactor if the need eventually arises.

## Hooks

With its default "BDD"-style interface, Mocha provides the hooks `before()`, `after()`, `beforeEach()`, and `afterEach()`.  These should be used to set up preconditions and clean up after your tests.

```js
describe('hooks', function() {

  before(function() {
    // runs before all tests in this block
  });

  after(function() {
    // runs after all tests in this block
  });

  beforeEach(function() {
    // runs before each test in this block
  });

  afterEach(function() {
    // runs after each test in this block
  });

  // test cases
});
```

> Tests can appear before, after, or interspersed with your hooks.  Hooks will run in the order they are defined, as appropriate; all `before()` hooks run (once), then any `beforeEach()` hooks, tests, any `afterEach()` hooks, and finally `after()` hooks (once).

### Describing Hooks

Any hook can be invoked with an optional description, making it easier to pinpoint errors in your tests.  If a hook is given a named function, that name will be used if no description is supplied.

```js
beforeEach(function() {
  // beforeEach hook
});

beforeEach(function namedFun() {
  // beforeEach:namedFun
});

beforeEach('some description', function() {
  // beforeEach:some description
});
```

### Asynchronous Hooks

All hooks (`before()`, `after()`, `beforeEach()`, `afterEach()`) may be sync or async as well, behaving much like a regular test-case. For example, you may wish to populate database with dummy content before each test:

```js
describe('Connection', function() {
  var db = new Connection,
    tobi = new User('tobi'),
    loki = new User('loki'),
    jane = new User('jane');

  beforeEach(function(done) {
    db.clear(function(err) {
      if (err) return done(err);
      db.save([tobi, loki, jane], done);
    });
  });

  describe('#find()', function() {
    it('respond with matching records', function(done) {
      db.find({type: 'User'}, function(err, res) {
        if (err) return done(err);
        res.should.have.length(3);
        done();
      });
    });
  });
});
```

### Root-Level Hooks

You may also pick any file and add "root"-level hooks.  For example, add `beforeEach()` outside of all `describe()` blocks.  This will cause the callback to `beforeEach()` to run before any test case, regardless of the file it lives in (this is because Mocha has an *implied* `describe()` block, called the "root suite").

```js
beforeEach(function() {
  console.log('before every test in every file');
});
```

### Delayed Root Suite

If you need to perform asynchronous operations before any of your suites are run, you may delay the root suite.  Run `mocha` with the `--delay` flag.  This will attach a special callback function, `run()`, to the global context:

```js
setTimeout(function() {
  // do some setup

  describe('my suite', function() {
    // ...
  });

  run();
}, 5000);
```

## Pending Tests

"Pending"--as in "someone should write these test cases eventually"--test-cases are simply those *without* a callback:

```js
describe('Array', function() {
  describe('#indexOf()', function() {
    // pending test below
    it('should return -1 when the value is not present');
  });
});
```

Pending tests will be included in the test results, and marked as pending. A pending test is not considered a failed test.

## Exclusive Tests

The exclusivity feature allows you to run *only* the specified suite or test-case
 by appending `.only()` to the function.  Here's an example of executing only a particular suite:

```js
describe('Array', function() {
  describe.only('#indexOf()', function() {
    // ...
  });
});
```

*Note*: All nested suites will still be executed.

Here's an example of executing an individual test case:

```js
describe('Array', function() {
  describe('#indexOf()', function() {
    it.only('should return -1 unless present', function() {
      // ...
    });

    it('should return the index when present', function() {
      // ...
    });
  });
});
```

Previous to v3.0.0, `.only()` used string matching to decide which tests to execute.  As of v3.0.0, this is no longer the case.  In v3.0.0 or newer, `.only()` can be used multiple times to define a subset of tests to run:

```js
describe('Array', function() {
  describe('#indexOf()', function() {
    it.only('should return -1 unless present', function() {
      // this test will be run
    });

    it.only('should return the index when present', function() {
      // this test will also be run
    });

    it('should return -1 if called with a non-Array context', function() {
      // this test will not be run
    });
  });
});
```

You may also choose multiple suites:

```js
describe('Array', function() {
  describe.only('#indexOf()', function() {
    it('should return -1 unless present', function() {
      // this test will be run
    });

    it('should return the index when present', function() {
      // this test will also be run
    });
  });

  describe.only('#concat()', function () {
    it('should return a new Array', function () {
      // this test will also be run
    });
  });

  describe('#slice()', function () {
    it('should return a new Array', function () {
      // this test will not be run
    });
  });
});
```

But *tests will have precedence*:

```js
describe('Array', function() {
  describe.only('#indexOf()', function() {
    it.only('should return -1 unless present', function() {
      // this test will be run
    });

    it('should return the index when present', function() {
      // this test will not be run
    });
  });
});
```

*Note*: Hooks, if present, will still be executed.

> Be mindful not to commit usages of `.only()` to version control, unless you really mean it! To do so one can run mocha with the option `--forbid-only` in the continuous integration test command (or in a git precommit hook).

## Inclusive Tests

This feature is the inverse of `.only()`.  By appending `.skip()`, you may tell Mocha to simply ignore these suite(s) and test case(s).  Anything skipped will be marked as [pending](#pending-tests), and reported as such.  Here's an example of skipping an entire suite:

```js
describe('Array', function() {
  describe.skip('#indexOf()', function() {
    // ...
  });
});
```

Or a specific test-case:

```js
describe('Array', function() {
  describe('#indexOf()', function() {
    it.skip('should return -1 unless present', function() {
      // this test will not be run
    });

    it('should return the index when present', function() {
      // this test will be run
    });
  });
});
```

> *Best practice*: Use `.skip()` instead of commenting tests out.

You may also skip *at runtime* using `this.skip()`.  If a test needs an environment or configuration which cannot be detected beforehand, a runtime skip is appropriate. For example:

```js
it('should only test in the correct environment', function() {
  if (/* check test environment */) {
    // make assertions
  } else {
    this.skip();
  }
});
```

The above test will be reported as [pending](#pending-tests).  It's also important to note that calling `this.skip()` will effectively *abort* the test.

> *Best practice*: To avoid confusion, do not execute further instructions in a test or hook after calling `this.skip()`.

Contrast the above test with the following code:

```js
it('should only test in the correct environment', function() {
  if (/* check test environment */) {
    // make assertions
  } else {
    // do nothing
  }
});
```

Because this test *does nothing*, it will be reported as *passing*.

> *Best practice*: Don't do nothing!  A test should make an assertion or use `this.skip()`.

To skip *multiple* tests in this manner, use `this.skip()` in a "before" hook:

```js
before(function() {
  if (/* check test environment */) {
    // setup code
  } else {
    this.skip();
  }
});
```

This will skip all `it`, `beforeEach/afterEach`, and `describe` blocks within the suite. `before/after` hooks are skipped unless they are defined at the same level as the hook containing `this.skip()`.

```js
describe('outer', function () {
  before(function () {
    this.skip();
  });

  after(function () {
    // will be executed
  });

  describe('inner', function () {
    before(function () {
      // will be skipped
    });

    after(function () {
      // will be skipped
    });
  });
});
```

> Before Mocha v3.0.0, `this.skip()` was not supported in asynchronous tests and hooks.

## Retry Tests

You can choose to retry failed tests up to a certain number of times. This feature is designed to handle end-to-end tests (functional tests/Selenium...) where resources cannot be easily mocked/stubbed. **It's not recommended to use this feature for unit tests**.

This feature does re-run `beforeEach/afterEach` hooks but not `before/after` hooks.

**NOTE**: Example below was written using Selenium webdriver (which [overwrites global Mocha hooks](https://github.com/SeleniumHQ/selenium/blob/c10e8a955883f004452cdde18096d70738397788/javascript/node/selenium-webdriver/testing/index.js) for `Promise` chain).

```js
describe('retries', function() {
  // Retry all tests in this suite up to 4 times
  this.retries(4);

  beforeEach(function () {
    browser.get('http://www.yahoo.com');
  });

  it('should succeed on the 3rd try', function () {
    // Specify this test to only retry up to 2 times
    this.retries(2);
    expect($('.foo').isDisplayed()).to.eventually.be.true;
  });
});

```

## Dynamically Generating Tests

Given Mocha's use of `Function.prototype.call` and function expressions to define suites and test cases, it's straightforward to generate your tests dynamically.  No special syntax is required &mdash; plain ol' JavaScript can be used to achieve functionality similar to "parameterized" tests, which you may have seen in other frameworks.

Take the following example:

```js
var assert = require('chai').assert;

function add() {
  return Array.prototype.slice.call(arguments).reduce(function(prev, curr) {
    return prev + curr;
  }, 0);
}

describe('add()', function() {
  var tests = [
    {args: [1, 2],       expected: 3},
    {args: [1, 2, 3],    expected: 6},
    {args: [1, 2, 3, 4], expected: 10}
  ];

  tests.forEach(function(test) {
    it('correctly adds ' + test.args.length + ' args', function() {
      var res = add.apply(null, test.args);
      assert.equal(res, test.expected);
    });
  });
});
```

The above code will produce a suite with three specs:

```sh
$ mocha

  add()
    ✓ correctly adds 2 args
    ✓ correctly adds 3 args
    ✓ correctly adds 4 args
```

<h2 id="test-duration">Test duration</h2>

Many reporters will display test duration and flag tests that are slow (default: 75ms), as shown here with the "spec" reporter:

![test duration](images/reporter-spec-duration.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

There are three levels of test duration (depicted in the following image):

1. FAST: Tests that run within half of the "slow" threshold will show the duration in green (if at all).
2. NORMAL: Tests that run exceeding half of the threshold (but still within it) will show the duration in yellow.
3. SLOW: Tests that run exceeding the threshold will show the duration in red.

![test duration range](images/test-duration-range.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

To tweak what's considered "slow", you can use the `slow()` method:

```js
describe('something slow', function() {
  this.slow(10000);

  it('should take long enough for me to go make a sandwich', function() {
    // ...
  });
});
```

## Timeouts

### Suite-level

Suite-level timeouts may be applied to entire test "suites", or disabled via `this.timeout(0)`. This will be inherited by all nested suites and test-cases that do not override the value.

```js
describe('a suite of tests', function() {
  this.timeout(500);

  it('should take less than 500ms', function(done){
    setTimeout(done, 300);
  });

  it('should take less than 500ms as well', function(done){
    setTimeout(done, 250);
  });
})
```

### Test-level

Test-specific timeouts may also be applied, or the use of `this.timeout(0)` to disable timeouts all together:

```js
it('should take less than 500ms', function(done){
  this.timeout(500);
  setTimeout(done, 300);
});
```

### Hook-level

Hook-level timeouts may also be applied:

```js
describe('a suite of tests', function() {
  beforeEach(function(done) {
    this.timeout(3000); // A very long environment setup.
    setTimeout(done, 2500);
  });
});
```

Again, use `this.timeout(0)` to disable the timeout for a hook.

> In v3.0.0 or newer, a parameter passed to `this.timeout()` greater than the [maximum delay value](https://developer.mozilla.org/docs/Web/API/WindowTimers/setTimeout#Maximum_delay_value) will cause the timeout to be disabled.

## Diffs

Mocha supports the `err.expected` and `err.actual` properties of any thrown `AssertionError`s from an assertion library.  Mocha will attempt to display the difference between what was expected, and what the assertion actually saw.  Here's an example of a "string" diff:

![string diffs](images/reporter-string-diffs.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

## Command-Line Usage

<!-- AUTO-GENERATED-CONTENT:START (usage:executable=bin/mocha) -->

```plain

mocha [spec..]

Run tests with Mocha

Commands
  mocha debug [spec..]  Run tests with Mocha                           [default]
  mocha init <path>     create a client-side Mocha setup at <path>

Rules & Behavior
  --allow-uncaught           Allow uncaught errors to propagate        [boolean]
  --async-only, -A           Require all tests to use a callback (async) or
                             return a Promise                          [boolean]
  --bail, -b                 Abort ("bail") after first test failure   [boolean]
  --check-leaks              Check for global variable leaks           [boolean]
  --delay                    Delay initial execution of root suite
  --exit                     Force Mocha to quit after tests complete  [boolean]
  --forbid-only              Fail if exclusive test(s) encountered     [boolean]
  --forbid-pending           Fail if pending test(s) encountered       [boolean]
  --global, --globals        List of allowed global variables            [array]
  --retries                  Retry failed tests this many times         [number]
  --slow, -s                 Specify "slow" test threshold (in milliseconds)
                                                          [number] [default: 75]
  --timeout, -t, --timeouts  Specify test timeout threshold (in milliseconds)
                                                        [number] [default: 2000]
  --ui, -u                   Specify user interface    [string] [default: "bdd"]

Reporting & Output
  --color, -c, --colors                     Force-enable color output  [boolean]
  --diff                                    Show diff on failure
                                                       [boolean] [default: true]
  --full-trace                              Display full stack traces  [boolean]
  --growl, -G                               Enable Growl notifications [boolean]
  --inline-diffs                            Display actual/expected differences
                                            inline within each string  [boolean]
  --reporter, -R                            Specify reporter to use
                                                      [string] [default: "spec"]
  --reporter-option, --reporter-options,    Reporter-specific options
  -O                                        (<k=v,[k1=v1,..]>)           [array]

Configuration
  --config   Path to config file                    [default: (nearest rc file)]
  --opts     Path to `mocha.opts`        [string] [default: "./test/mocha.opts"]
  --package  Path to package.json for config                            [string]

File Handling
  --exclude                        Ignore file(s) or glob pattern(s)
                                                       [array] [default: (none)]
  --extension, --watch-extensions  File extension(s) to load and/or watch
                                                           [array] [default: js]
  --file                           Specify file(s) to be loaded prior to root
                                   suite execution     [array] [default: (none)]
  --recursive                      Look for tests in subdirectories    [boolean]
  --require, -r                    Require module      [array] [default: (none)]
  --sort, -S                       Sort test files
  --watch, -w                      Watch files in the current working directory
                                   for changes                         [boolean]

Test Filters
  --fgrep, -f   Only run tests containing this string                   [string]
  --grep, -g    Only run tests matching this string or regexp           [string]
  --invert, -i  Inverts --grep and --fgrep matches                     [boolean]

Positional Arguments
  spec  One or more files, directories, or globs to test
                                                    [array] [default: ["test/"]]

Other Options
  --help, -h     Show usage information & exit                         [boolean]
  --version, -V  Show version number & exit                            [boolean]
  --interfaces   List built-in user interfaces & exit
  --reporters    List built-in reporters & exit

Mocha Resources
    Chat: [35mhttps://gitter.im/mochajs/mocha[39m
  GitHub: [34mhttps://github.com/mochajs/mocha.git[39m
    Docs: [33mhttps://mochajs.org/[39m

```

<!-- AUTO-GENERATED-CONTENT:END -->

### `--allow-uncaught`

By default, Mocha will attempt to trap uncaught exceptions thrown from running tests and report these as test failures.  Use `--allow-uncaught` to disable this behavior and allow uncaught exceptions to propagate.  Will typically cause the process to crash.

This flag is useful when debugging particularly difficult-to-track exceptions.

### `--async-only, -A`

Enforce a rule that tests must be written in "async" style, meaning each test provides a `done` callback or returns a `Promise`.  Non-compliant tests will be marked as failures.

### `--bail, -b`

Causes Mocha to stop running tests after the first test failure it encounters.

`--bail` does *not* imply `--exit`.

### `--check-leaks`

Use this option to have Mocha check for global variables that are leaked while running tests. Specify globals that are acceptable via the `--global` option (for example: `--check-leaks --global jQuery --global MyLib`).

### `--compilers`

> *`--compilers` was removed in v6.0.0. See [further explanation and workarounds](https://github.com/mochajs/mocha/wiki/compilers-deprecation).*

### `--exit`

> *Updated in v4.0.0.*

TL;DR: If your tests hang after an upgrade to Mocha v4.0.0 or newer, use `--exit` for a quick (though not necessarily recommended) fix.

*Prior to* version v4.0.0, *by default*, Mocha would force its own process to exit once it was finished executing all tests.  This behavior enables a set of potential problems; it's indicative of tests (or fixtures, harnesses, code under test, etc.) which don't clean up after themselves properly.  Ultimately, "dirty" tests can (but not always) lead to *false positive* or *false negative* results.

"Hanging" most often manifests itself if a server is still listening on a port, or a socket is still open, etc.  It can also be something like a runaway `setInterval()`, or even an errant `Promise` that never fulfilled.

The *default behavior* in v4.0.0 is `--no-exit`, where previously it was `--exit`.

**The easiest way to "fix" the issue is to simply pass `--exit` to the Mocha process.**  It *can* be time-consuming to debug--because it's not always obvious where the problem is--but it *is* recommended to do so.

To ensure your tests aren't leaving messes around, here are some ideas to get started:

- See the [Node.js guide to debugging](https://nodejs.org/en/docs/inspector/)
- Use the new [`async_hooks`](https://github.com/nodejs/node/blob/master/doc/api/async_hooks.md) API ([example](https://git.io/vdlNM))
- Try something like [wtfnode](https://npm.im/wtfnode)
- Use [`.only`](#exclusive-tests) until you find the test that causes Mocha to hang

### `--forbid-only`

Enforce a rule that tests may not be exclusive (use of e.g., `describe.only()` or `it.only()` is disallowed).

`--forbid-only` causes Mocha to fail when an exclusive ("only'd") test or suite is encountered, and it will abort further test execution.

### `--forbid-pending`

Enforce a rule that tests may not be skipped (use of e.g., `describe.skip()`,  `it.skip()`, or `this.skip()` anywhere is disallowed).

`--forbid-pending` causes Mocha to fail when a skipped ("pending") test or suite is encountered, and it will abort further test execution.

### `--global <variable-name>`

> *Updated in v6.0.0; the option is `--global` and `--globals` is now an alias.*

Define a global variable name. For example, suppose your app deliberately exposes a global named `app` and `YUI`, you may want to add `--global app --global YUI`.

`--global` accepts wildcards. You could do `--global '*bar'` and it would match `foobar`, `barbar`, etc. You can also simply pass in `'*'` to ignore all globals.

`--global` can accept a comma-delimited list; `--global app,YUI` is equivalent to `--global app --global YUI`.

By using this option in conjunction with `--check-leaks`, you can specify a whitelist of known global variables that you *expect* to leak into global scope.

### `--retries <n>`

Retries failed tests `n` times.

Mocha does not retry test failures by default.

### `--slow <ms>, -s <ms>`

Specify the "slow" test threshold in milliseconds.  Mocha uses this to highlight test cases that are taking too long.  "Slow" tests are not considered failures.

Note: A test that executes for *half* of the "slow" time will be highlighted *in yellow* with the default `spec` reporter; a test that executes for entire "slow" time will be highlighted *in red*.

### `--timeout <ms>, -t <ms>`

> *Update in v6.0.0: `--no-timeout` is implied when invoking Mocha using debug flags. It is equivalent to `--timeout 0`.  `--timeout 99999999` is no longer needed.*

Specifies the test case timeout, defaulting to two (2) seconds (2000 milliseconds).  Tests taking longer than this amount of time will be marked as failed.

To override you may pass the timeout in milliseconds, or a value with the `s` suffix, e.g., `--timeout 2s` and `--timeout 2000` are equivalent.

To disable timeouts, use `--no-timeout`.

Note: synchronous (blocking) tests are also bound by the timeout, but they will not complete until the code stops blocking.  Infinite loops will still be infinite loops!

### `--ui <name>, -u <name>`

The `--ui` option lets you specify the interface to use, defaulting to `bdd`.

### `--color, -c, --colors`

> *Updated in v6.0.0.  `--colors` is now an alias for `--color`.*

"Force" color output to be enabled, or alternatively force it to be disabled via `--no-color`.  By default, Mocha uses [supports-color](https://npm.im/supports-color) to decide.

In some cases, color output will be explicitly suppressed by certain reporters outputting in a machine-readable format.

### `--diff`

When possible, show the difference between expected and actual values when an assertion failure is encountered.

This flag is unusual in that it **defaults to `true`**; use `--no-diff` to suppress Mocha's own diff output.

Some assertion libraries will supply their own diffs, in which case Mocha's will not be used, regardless of the default value.

Mocha's own diff output does not conform to any known standards, and is designed to be human-readable.

### `--full-trace`

Enable "full" stack traces.  By default, Mocha attempts to distill stack traces into less noisy (though still useful) output.

This flag is helpful when debugging a suspected issue within Mocha or Node.js itself.

### `--growl, -G`

Enable [Growl](http://growl.info) (or OS-level notifications where available).

Requires extra software to be installed; see the [growl module's docs](https://npm.im/growl) for more information.

### `--inline-diffs`

Enable "inline" diffs, an alternative output for diffing strings.

Useful when working with large strings.

Does nothing if an assertion library supplies its own diff output.

### `--reporter <name>, -R <name>`

Specify the reporter that will be used, defaulting to `spec`.

Allows use of third-party reporters. For example, [mocha-lcov-reporter](https://npm.im/mocha-lcov-reporter) may be used with `--reporter mocha-lcov-reporter` after it has been installed.

### `--reporter-option <option>, -O <option>, --reporter-options <option>`

> *Updated in v6.0.0.  Can be specified multiple times.  `--reporter-options` is now an alias for `--reporter-option`.*

Provide options specific to a reporter in `<key>=<value>` format, e.g., `--reporter tap --reporter-option tapVersion=13`.

Not all reporters accept options.

Can be specified as a comma-delimited list.

### `--config <path>`

> *New in v6.0.0.*

Specify an explicit path to a [configuration file](#configuring-mocha-node-js).

By default, Mocha will search for a config file if `--config` is not specified; use `--no-config` to suppress this behavior.

### `--opts <path>`

> *Updated in v6.0.0; added `--no-opts`.*

Specify a path to [`mocha.opts`](#mocha-opts).

By default, Mocha looks for a `mocha.opts` in `test/mocha.opts`; use `--no-opts` to suppress this behavior.

### `--package <path>`

> *New in v6.0.0.*

Specify an explicit path to a [`package.json` file](#configuring-mocha-node-js)  (ostensibly containing configuration in a `mocha` property).

By default, Mocha looks for a `package.json` in the current working directory or nearest ancestor, and will use the first file found (regardless of whether it contains a `mocha` property); to suppress `package.json` lookup, use `--no-package`.

### `--exclude <file/directory/glob>`

Explicitly exclude one or more files, directories or "globs" that would otherwise be loaded.

Files specified using `--file` *are not affected* by this option.

Can be specified multiple times.

### `--extension <ext>, --watch-extensions <ext>`

> *Updated in v6.0.0.  Previously `--watch-extensions`, but now expanded to affect general test file loading behavior. `--watch-extensions` is now an alias*

Files having this extension will be considered test files.  Defaults to `js`.

Affects `--watch` behavior.

Specifying `--extension` will *remove* `.js` as a test file extension; use `--extension js` to re-add it.  For example, to load `.mjs` and `.js` test files, you must supply `--extension mjs --extension js`.

### `--file <file/directory/glob>`

Explicitly *include* a test file to be loaded before other test files files.  Multiple uses of `--file` are allowed, and will be loaded in order given.

Useful if you want to declare, for example, hooks to be run before every test across all other test files.

Files specified this way are not affected by `--sort` or `--recursive`.

Files specified in this way should contain one or more suites, tests or hooks.  If this is not the case, consider `--require` instead.

### `--recursive`

When looking for test files, recurse into subdirectories.

See `--extension` for defining which files are considered test files.

### `--require <module>, -r <module>`

Require a module before loading the user interface or test files.  This is useful for:

- Test harnesses
- Assertion libraries that augment built-ins or global scope (such as [should.js](https://npm.im/should.js))
- Instant ECMAScript modules via [esm](https://npm.im/esm)
- Compilers such as Babel via [@babel/register](https://npm.im/@babel/register) or TypeScript via [ts-node](https://npm.im/ts-node) (using `--require ts-node/register`)

Modules required in this manner are expected to do work synchronously; Mocha won't wait for async tasks in a required module to finish.

Note you cannot use `--require` to set a global `beforeEach()` hook, for example--use `--file` instead, which allows you to specify an explicit order in which test files are loaded.

### `--sort, -S`

Sort test files (by absolute path) using [Array.prototype.sort](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort).

### `--watch, -w`

Executes tests on changes to JavaScript in the current working directory (and once initially).

By default, only files with extension `.js` are watched.  Use `--extension` to change this behavior.

### `--fgrep <string>, -f <string>`

> *BREAKING CHANGE in v6.0.0; now mutually exclusive with `--grep`.*

Cause Mocha to only run tests having titles containing the given `string`.

Mutually exclusive with `--grep`.

### `--grep <regexp>, -g <regexp>`

> *BREAKING CHANGE in v6.0.0; now mutually exclusive with `--fgrep`.*

Cause Mocha to only run tests matching the given `regexp`, which is internally compiled to a [RegExp](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Regexp).

Suppose, for example, you have "api" related tests, as well as "app" related tests, as shown in the following snippet; One could use `--grep api` or `--grep app` to run one or the other. The same goes for any other part of a suite or test-case title, `--grep users` would be valid as well, or even `--grep GET`.

```js
describe('api', function() {
  describe('GET /api/users', function() {
    it('respond with an array of users', function() {
      // ...
    });
  });
});

describe('app', function() {
  describe('GET /users', function() {
    it('respond with an array of users', function() {
      // ...
    });
  });
});
```

Mutually exclusive with `--fgrep`.

### `--invert`

Use the *inverse* of the match specified by `--grep` or `fgrep`.

Requires either `--grep` or `--fgrep` (but not both).

### `--debug, --inspect, --debug-brk, --inspect-brk, debug, inspect`

> *BREAKING CHANGE in v6.0.0; `-d` is no longer an alias for `--debug`.*
> *Other updates in v6.0.0:*
> *In versions of Node.js implementing `--inspect` and `--inspect-brk`, `--debug` and `--debug-brk` are respectively aliases for these two options.*
> *Likewise, `debug` (not `--debug`) is an alias for `inspect` (not `--inspect`) in Node.js versions where `debug` is deprecated.*

Enables Node.js' debugger or inspector.

Use `--inspect` / `--inspect-brk` / `--debug` / `--debug-brk` to launch the V8 inspector for use with Chrome Dev Tools.

Use `inspect` / `debug` to launch Node.js' internal debugger.

All of these options are mutually exclusive.

Implies `--no-timeout`.

### About Option Types

> *Updated in v6.0.0.*

Each flag annotated of type `[boolean]` in Mocha's `--help` output can be *negated* by prepending `--no-` to the flag name.  For example, `--no-color` will disable Mocha's color output, which is enabled by default.

Unless otherwise noted, *all* boolean flags default to `false`.

### About `node` Flags

The `mocha` executable supports all applicable flags which the `node` executable supports.

These flags vary depending on your version of Node.js.

`node` flags can be defined in Mocha's [configuration](#configuring-mocha-nodejs).

### About V8 Flags

Prepend `--v8-` to any flag listed in the output of `node --v8-options` (excluding `--v8-options` itself) to use it.

V8 flags can be defined in Mocha's [configuration](#configuring-mocha-nodejs).

## Interfaces

Mocha's "interface" system allows developers to choose their style of DSL.  Mocha has **BDD**, **TDD**, **Exports**, **QUnit** and **Require**-style interfaces.

### BDD

The **BDD** interface provides `describe()`, `context()`, `it()`, `specify()`, `before()`, `after()`, `beforeEach()`, and `afterEach()`.

`context()` is just an alias for `describe()`, and behaves the same way; it just provides a way to keep tests easier to read and organized. Similarly, `specify()` is an alias for `it()`.

> All of the previous examples were written using the **BDD** interface.

```js
  describe('Array', function() {
    before(function() {
      // ...
    });

    describe('#indexOf()', function() {
      context('when not present', function() {
        it('should not throw an error', function() {
          (function() {
            [1,2,3].indexOf(4);
          }).should.not.throw();
        });
        it('should return -1', function() {
          [1,2,3].indexOf(4).should.equal(-1);
        });
      });
      context('when present', function() {
        it('should return the index where the element first appears in the array', function() {
          [1,2,3].indexOf(3).should.equal(2);
        });
      });
    });
  });
```

### TDD

The **TDD** interface provides `suite()`, `test()`, `suiteSetup()`, `suiteTeardown()`, `setup()`, and `teardown()`:

```js
suite('Array', function() {
  setup(function() {
    // ...
  });

  suite('#indexOf()', function() {
    test('should return -1 when not present', function() {
      assert.equal(-1, [1,2,3].indexOf(4));
    });
  });
});
```

### Exports

The **Exports** interface is much like Mocha's predecessor [expresso](https://github.com/tj/expresso). The keys `before`, `after`, `beforeEach`, and `afterEach` are special-cased, object values are suites, and function values are test-cases:

```js
module.exports = {
  before: function() {
    // ...
  },

  'Array': {
    '#indexOf()': {
      'should return -1 when not present': function() {
        [1,2,3].indexOf(4).should.equal(-1);
      }
    }
  }
};
```

### QUnit

The [QUnit](https://qunitjs.com)-inspired interface matches the "flat" look of QUnit, where the test suite title is simply defined before the test-cases. Like TDD, it uses `suite()` and `test()`, but resembling BDD, it also contains `before()`, `after()`, `beforeEach()`, and `afterEach()`.

```js
function ok(expr, msg) {
  if (!expr) throw new Error(msg);
}

suite('Array');

test('#length', function() {
  var arr = [1,2,3];
  ok(arr.length == 3);
});

test('#indexOf()', function() {
  var arr = [1,2,3];
  ok(arr.indexOf(1) == 0);
  ok(arr.indexOf(2) == 1);
  ok(arr.indexOf(3) == 2);
});

suite('String');

test('#length', function() {
  ok('foo'.length == 3);
});
```

### Require

The `require` interface allows you to require the `describe` and friend words directly using `require` and call them whatever you want. This interface is also useful if you want to avoid global variables in your tests.

*Note*: The `require` interface cannot be run via the `node` executable, and must be run via `mocha`.

```js
var testCase = require('mocha').describe;
var pre = require('mocha').before;
var assertions = require('mocha').it;
var assert = require('chai').assert;

testCase('Array', function() {
  pre(function() {
    // ...
  });

  testCase('#indexOf()', function() {
    assertions('should return -1 when not present', function() {
      assert.equal([1,2,3].indexOf(4), -1);
    });
  });
});
```

## Reporters

Mocha reporters adjust to the terminal window, and always disable ANSI-escape coloring when the stdio streams are not associated with a TTY.

### Spec

This is the default reporter.  The "spec" reporter outputs a hierarchical view nested just as the test cases are.

![spec reporter](images/reporter-spec.png?withoutEnlargement&resize=920,9999){:class="screenshot"}
![spec reporter with failure](images/reporter-spec-fail.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

### Dot Matrix

The dot matrix (or "dot") reporter is simply a series of characters which represent test cases.  Failures highlight in red exclamation marks (`!`), pending tests with a blue comma (`,`), and slow tests as yellow.  Good if you prefer minimal output.

![dot matrix reporter](images/reporter-dot.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

### Nyan

The "nyan" reporter is exactly what you might expect:

![js nyan cat reporter](images/reporter-nyan.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

### TAP

The TAP reporter emits lines for a [Test-Anything-Protocol](https://en.wikipedia.org/wiki/Test_Anything_Protocol) consumer.

![test anything protocol](images/reporter-tap.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

### Landing Strip

The Landing Strip (`landing`) reporter is a gimmicky test reporter simulating a plane landing :) unicode ftw

![landing strip plane reporter](images/reporter-landing.png?withoutEnlargement&resize=920,9999){:class="screenshot"}
![landing strip with failure](images/reporter-landing-fail.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

### List

The "list" reporter outputs a simple specifications list as test cases pass or fail, outputting the failure details at the bottom of the output.

![list reporter](images/reporter-list.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

### Progress

The "progress" reporter implements a simple progress-bar:

![progress bar](images/reporter-progress.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

### JSON

The "JSON" reporter outputs a single large JSON object when the tests have completed (failures or not).

![json reporter](images/reporter-json.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

### JSON Stream

The "JSON stream" reporter outputs newline-delimited JSON "events" as they occur, beginning with a "start" event, followed by test passes or failures, and then the final "end" event.

![json stream reporter](images/reporter-json-stream.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

### Min

The "min" reporter displays the summary only, while still outputting errors on failure. This reporter works great with `--watch` as it clears the terminal in order to keep your test summary at the top.

![min reporter](images/reporter-min.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

### Doc

The "doc" reporter outputs a hierarchical HTML body representation of your tests.  Wrap it with a header, footer, and some styling, then you have some fantastic documentation!

![doc reporter](images/reporter-doc.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

For example, suppose you have the following JavaScript:

```js
describe('Array', function() {
  describe('#indexOf()', function() {
    it('should return -1 when the value is not present', function() {
      [1,2,3].indexOf(5).should.equal(-1);
      [1,2,3].indexOf(0).should.equal(-1);
    });
  });
});
```

The command `mocha --reporter doc array` would yield:

```html
<section class="suite">
  <h1>Array</h1>
  <dl>
    <section class="suite">
      <h1>#indexOf()</h1>
      <dl>
      <dt>should return -1 when the value is not present</dt>
      <dd><pre><code>[1,2,3].indexOf(5).should.equal(-1);
[1,2,3].indexOf(0).should.equal(-1);</code></pre></dd>
      </dl>
    </section>
  </dl>
</section>
```

The SuperAgent request library [test documentation](https://visionmedia.github.io/superagent/docs/test.html) was generated with Mocha's doc reporter using this Bash command:

```bash
$ mocha --reporter=doc | cat docs/head.html - docs/tail.html > docs/test.html
```

View SuperAgent's [Makefile](https://github.com/visionmedia/superagent/blob/master/Makefile) for reference.

### Markdown

The "markdown" reporter generates a markdown TOC and body for your test suite. This is great if you want to use the tests as documentation within a Github wiki page, or a markdown file in the repository that Github can render. For example here is the Connect [test output](https://github.com/senchalabs/connect/blob/90a725343c2945aaee637e799b1cd11e065b2bff/tests.md).

### XUnit

The `xunit` reporter is also available.  It outputs an XUnit-compatible XML document, often applicable in CI servers.

By default, it will output to the console.  To write directly to a file, use `--reporter-options output=filename.xml`.

### Third-Party Reporters

Mocha allows you to define custom reporters. For more information see the [wiki](https://github.com/mochajs/mocha/wiki/Third-party-reporters). An example is the [TeamCity reporter](https://github.com/travisjeffery/mocha-teamcity-reporter).

### HTML Reporter

**The HTML reporter is not intended for use on the command-line.**

## Running Mocha in the Browser

Mocha runs in the browser. Every release of Mocha will have new builds of `./mocha.js` and `./mocha.css` for use in the browser.

### Browser-specific methods

  The following method(s) *only* function in a browser context:

  `mocha.allowUncaught()` : If called, uncaught errors will not be absorbed by the error handler.

A typical setup might look something like the following, where we call `mocha.setup('bdd')` to use the **BDD** interface before loading the test scripts, running them `onload` with `mocha.run()`.

```html
<html>
<head>
  <meta charset="utf-8">
  <title>Mocha Tests</title>
  <link href="https://unpkg.com/mocha@5.2.0/mocha.css" rel="stylesheet" />
</head>
<body>
  <div id="mocha"></div>

  <script src="https://unpkg.com/chai/chai.js"></script>
  <script src="https://unpkg.com/mocha@5.2.0/mocha.js"></script>

  <script>mocha.setup('bdd')</script>
  <script src="test.array.js"></script>
  <script src="test.object.js"></script>
  <script src="test.xhr.js"></script>
  <script>
    mocha.checkLeaks();
    mocha.run();
  </script>
</body>
</html>
```

### Grep

The browser may use the `--grep` as functionality.  Append a query-string to your URL: `?grep=api`.

### Browser Configuration

Mocha options can be set via `mocha.setup()`.  Examples:

```js
// Use "tdd" interface.  This is a shortcut to setting the interface;
// any other options must be passed via an object.
mocha.setup('tdd');

// This is equivalent to the above.
mocha.setup({
  ui: 'tdd'
});

// Use "tdd" interface, ignore leaks, and force all tests to be asynchronous
mocha.setup({
  ui: 'tdd',
  ignoreLeaks: true,
  asyncOnly: true
});
```

### Browser-specific Option(s)

The following option(s) *only* function in a browser context:

`noHighlighting`: If set to `true`, do not attempt to use syntax highlighting on output test code.

### Reporting

The "HTML" reporter is what you see when running Mocha in the browser.  It looks like this:

![HTML test reporter](images/reporter-html.png?withoutEnlargement&resize=920,9999){:class="screenshot"}

[Mochawesome](https://www.npmjs.com/package/mochawesome) is a great alternative to the default HTML reporter.

## Configuring Mocha (Node.js)

> *New in v6.0.0*

In addition to supporting the legacy [`mocha.opts`](#mochaopts) run-control format, Mocha now supports configuration files, typical of modern command-line tools, in several formats:

- **JavaScript**: Create a `.mocharc.js` in your project's root directory, and export an object (`module.exports = {/* ... */}`) containing your configuration.
- **YAML**: Create a `.mocharc.yaml` (or `.mocharc.yml`) in your project's root directory.
- **JSON**: Create a `.mocharc.json` in your project's root directory.  Comments--while not valid JSON--are allowed in this file, and will be ignored by Mocha.
- **`package.json`**: Create a `mocha` property in your project's `package.json`.

Mocha suggests using one of the above strategies for configuration instead of the legacy `mocha.opts` format.

### Custom Locations

You can specify a custom location for your configuration file with the `--config <path>` option.  Mocha will use the file's extension to determine how to parse the file, and will assume JSON if unknown.

You can specify a custom `package.json` location as well, using the `--package <path>` option.

### Ignoring Config Files

To skip looking for config files, use `--no-config`. Likewise, use `--no-package` to stop Mocha from looking for configuration in a `package.json`.

### Priorities

If no custom path was given, and if there are multiple configuration files in the same directory, Mocha will search for--and use--only one.  The priority is:

1. `.mocharc.js`
1. `.mocharc.yaml`
1. `.mocharc.yml`
1. `.mocharc.json`

### Merging

Mocha will also *merge* any options found in `package.json` *and* `mocha.opts` into its run-time configuration.  In case of conflict, the priority is:

1. Arguments specified on command-line
1. Configuration file (`.mocharc.js`, `.mocharc.yml`, etc.)
1. `mocha` property of `package.json`
1. `mocha.opts`

Options which can safely be repeated (e.g., `--require`) will be *concatenated*, with higher-priorty configuration sources appearing earlier in the list.  For example, a `.mocharc.json` containing `"require": "bar"`, coupled with execution of `mocha --require foo`, would cause Mocha to require `foo`, then `bar`, in that order.

### Extending Configuration

Configurations can inherit from other modules using the `extends` keyword.  See [here](http://yargs.js.org/docs/#api-configobject-extends-keyword) for more information.

### Configuration Format

- Any "boolean" flag (which doesn't require a parameter, such as `--bail`), can be specified using a boolean value, e.g.: `"bail": true`.
- Any "array"-type option (see `mocha --help` for a list) can be a single string value.
- For options containing a dash (`-`), the option name can be specified using camelCase.
- Aliases are valid names, e.g., `R` instead of `reporter`.
- Test files can be specified using `spec`, e.g., `"spec": "test/**/*.spec.js"`.
- Flags to `node` are *also* supported in configuration files, like in `mocha.opts`.  Use caution, as these can vary between versions of Node.js!

**For more configuration examples, see the [`example/config` directory on GitHub.](https://github.com/mochajs/mocha/tree/master/example/config)**

## `mocha.opts`

> *Updated in v6.0.0; `mocha.opts` is now considered "legacy"--though not yet deprecated--and we recommend using a configuration file instead.*

Mocha will attempt to load `"./test/mocha.opts"` as a run-control file of sorts.

Beginning-of-line comment support is available; any line _starting_ with a
hash (`#`) symbol will be considered a comment. Blank lines may also be used.
Any other line will be treated as a command-line argument (along with any
associated option value) to be used as a default setting. Settings should be
specified one per line.

The lines in this file are prepended to any actual command-line arguments.
As such, actual command-line arguments will take precedence over the defaults.

For example, suppose you have the following `mocha.opts` file:

```sh
# mocha.opts
  --require should
  --reporter dot
  --ui bdd
```

The settings above will default the reporter to `dot`, require the `should`
library, and use `bdd` as the interface. With this, you may then invoke `mocha`
with additional arguments, here enabling [Growl](http://growl.info/) support,
and changing the reporter to `list`:

```sh
$ mocha --reporter list --growl
```

To ignore your `mocha.opts`, use the `--no-opts` option.

## The `test/` Directory

By default, `mocha` looks for the glob `./test/*.js`, so you may want to put your tests in `test/` folder. If you want to include sub directories, pass the `--recursive` option.

To configure where `mocha` looks for tests, you may pass your own glob:

```sh
$ mocha --recursive "./spec/*.js"
```

Some shells support recursive matching by using the `**` wildcard in a glob. Bash >= 4.3 supports this with the [`globstar` option](https://www.gnu.org/software/bash/manual/html_node/The-Shopt-Builtin.html) which [must be enabled](https://github.com/mochajs/mocha/pull/3348#issuecomment-383937247) to get the same results as passing the `--recursive` option ([ZSH](http://zsh.sourceforge.net/Doc/Release/Expansion.html#Recursive-Globbing) and [Fish](https://fishshell.com/docs/current/#expand-wildcard) support this by default). With recursive matching enabled, the following is the same as passing `--recursive`:

```sh
$ mocha "./spec/**/*.js"
```

*Note*: Double quotes around the glob are recommended for portability.

## Editor Plugins

The following editor-related packages are available:

### TextMate

The [Mocha TextMate bundle](https://github.com/mochajs/mocha.tmbundle) includes snippets to make writing tests quicker and more enjoyable.

### JetBrains

[JetBrains](https://www.jetbrains.com/) provides a [NodeJS plugin](https://www.jetbrains.com/idea/features/nodejs.html) for its suite of IDEs (IntelliJ IDEA, WebStorm, etc.), which contains a Mocha test runner, among other things.

![JetBrains Mocha Runner Plugin in Action](images/jetbrains-plugin.png?withoutEnlargement&resize=920,9999&pngquant){:class="screenshot"}

The plugin is titled **NodeJS**, and can be installed via **Preferences** > **Plugins**, assuming your license allows it.

### Wallaby.js

[Wallaby.js](https://wallabyjs.com/) is a continuous testing tool that enables real-time code coverage for Mocha with any assertion library in VS Code, Atom, JetBrains IDEs (IntelliJ IDEA, WebStorm, etc.), Sublime Text and Visual Studio for both browser and node.js projects.

![Wallaby.js in Action](images/wallaby.png?withoutEnlargement&resize=920,9999&pngquant){:class="screenshot"}

### Emacs

[Emacs](https://www.gnu.org/software/emacs/) support for running Mocha tests is available via a 3rd party package [mocha.el](https://github.com/scottaj/mocha.el). The package is available on MELPA, and can be installed via `M-x package-install mocha`.

![Emacs Mocha Runner in Action](images/emacs.png?withoutEnlargement&resize=920,9999&pngquant){:class="screenshot"}

### Mocha Sidebar (VS Code)

[Mocha sidebar](https://marketplace.visualstudio.com/items?itemName=maty.vscode-mocha-sidebar) is the most complete mocha extension for vs code.

#### Features

- see all tests in VS Code sidebar menu
- run & debug tests for each level hierarchy from all tests to a single test (and each describe of course)
- auto run tests on file save
- see tests results directly in the code editor

![mocha side bar in Action](images/mocha_side_bar.png?withoutEnlargement&resize=920,9999&pngquant){:class="screenshot"}

## Examples

Real live example code:

- [Express](https://github.com/visionmedia/express/tree/master/test)
- [Connect](https://github.com/senchalabs/connect/tree/master/test)
- [SuperAgent](https://github.com/visionmedia/superagent/tree/master/test/node)
- [WebSocket.io](https://github.com/LearnBoost/websocket.io/tree/master/test)
- [Mocha](https://github.com/mochajs/mocha/tree/master/test)

## Testing Mocha

To run Mocha's tests, you will need GNU Make or compatible; Cygwin should work.

```sh
$ cd /path/to/mocha
$ npm install
$ npm test
```

To use a different reporter:

```sh
$ REPORTER=nyan npm test
```

## More Information

In addition to chatting with us on [Gitter], for additional information such as using spies, mocking, and shared behaviours be sure to check out the [Mocha Wiki](https://github.com/mochajs/mocha/wiki) on GitHub. For discussions join the [Google Group](https://groups.google.com/group/mochajs). For a running example of Mocha, view [example/tests.html](example/tests.html). For the JavaScript API, view the [API documentation](api/) or the [source](https://github.com/mochajs/mocha/blob/master/lib/mocha.js#L51).

<!-- AUTO-GENERATED-CONTENT:START (manifest:template=[Gitter]: ${gitter}) -->
[Gitter]: https://gitter.im/mochajs/mocha
<!-- AUTO-GENERATED-CONTENT:END -->
