---
layout: post
title: Android instrumentation & "FAILED BINDER TRANSACTION"
tags:
- android
- adb
- testing
- continuous integration
---

### Enumerating the list of all tests to run

Recently we've been focusing on reducing the amount of time it takes to run our
Android test suite. As part of this effort we've written code to support
running our Android tests in parallel across multiple builders in Jenkins.

While this system currently works, something else doesn't: *our tests*. Running
the tests like this has changed the order of our tests, and uncovered some very
unfortunate inter-test dependencies.

In trying to track down exactly which tests make assumptions about the state of
the system, I've written up a little Jenkins job that uses Android's `am
instrument` command to enumerate all the tests that would be run, then runs each
of them individually. Theoretically, I could also reset the emulator state
(using snapshots) between each one, to ensure a **completely** clean slate, but
so far I'm happy with just having the application in-memory state wiped.

The command I originally used to enumerate the tests is this one:

    adb shell am instrument -w -e log true -e package com.lookout \
      -e notAnnotation com.lookout.annotations.ExcludeFromDefault \
      com.lookout.tests/android.test.InstrumentationTestRunner

You can read more about the various options to `am instrument` at the [Android
Developer reference for InstrumentationTestRunner][instrumentation-test-runner]

The important part of this command is `-e log true`, which makes
InstrumentationTestRunner only print what it would do, and not actually do it.

### Hanging instrumentation (FAILED BINDER TRANSACTION)

The problem that I encountered was that `am instrument` would never return (on
an emulator running 2.3.3), and I'd get a bunch of errors in `adb logcat` like
the following:

    I/TestRunner(  531): started: testSignalFlareDoesNotRunBecauseDisabled(com.lookout.utils.SignalFlareTests)
    E/JavaBinder(  531): !!! FAILED BINDER TRANSACTION !!!
    I/TestRunner(  531): finished: testSignalFlareDoesNotRunBecauseDisabled(com.lookout.utils.SignalFlareTests)

Doing some quick Googling revealed nothing useful - people mostly talk about
hitting this error when resizing bitmaps, which I'm not doing. As far as I know,
none of the code we've written is actually executed in this scenario.

It turns out that the something inside the Android test runner is using Binder
to pass data for each test, and the sheer volume of these requests is causing
JavaBinder to run out of buffer.

So, what's the solution? Hackishly, but trivially, all it needs to do is limit
the frequency of these requests. Luckily, the default InstrumentationTestRunner
already implements a feature for doing just that. By passing `-e delay_msec X`
to `am instrument`, it'll sleep X milliseconds between each test.
Experimentation showed that 5ms seems to be enough, which makes the final
command this:

    adb shell am instrument -w -e delay_msec 5 -e log true -e package com.lookout \
      -e notAnnotation com.lookout.annotations.ExcludeFromDefault \
      com.lookout.tests/android.test.InstrumentationTestRunner


With this change, `am instrument` logs all the tests it would run (to `adb
logcat`) and returns successfully!


---


Hopefully this will help anyone else who comes across this issue, and give you
some inspiration to make a really awesome distributed Android test runner! :-)


\- [Jørgen P. Tjernø](https://github.com/jorgenpt/)

[instrumentation-test-runner]: http://developer.android.com/reference/android/test/InstrumentationTestRunner.html
