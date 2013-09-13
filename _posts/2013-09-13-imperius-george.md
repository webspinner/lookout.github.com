---
layout: post
title: End-to-End Testing in Android with ImperiusGeorge
author: Tim
tags:
- android
- adb
- testing
- ruby
- java
---

The Android ecosystem can be a diverse and difficult target while maintaining stability
across all devices. Various testing approaches can alleviates this problem– and the holy
grail is end-to-end testing on production phones with production builds.

##Ho, UI-Automator!
Enter [UI-Automator](http://developer.android.com/tools/help/uiautomator/index.html),
google's new tool included in 4.1+ phones. Unlike [Robotium](https://code.google.com/p/robotium/)
it runs on production builds of *all* apps on production android phones.

To use it you must extend a junit test class and form your tests into small un-ordered test
cases written in Java. It's possible to use the Apache network libraries and do network
communication and let you do end-to-end testing but there are a couple drawbacks. You can't
quickly and easily install and remove apks or use many of the developer tools available via
ADB. It's also difficult to coordinate multiple devices.

<div style="text-align:center"><img src="/images/post-images/imperius-george/george.jpg" alt="funny logo" /></div>
##Enter ImperiusGeorge
We wrote a tool to let us write Ruby on a coordinating computer that controls any UI-Automator
enabled device over USB. You write ruby on your desktop/server and it'll drive the java runtime
on Android.

It's called ImperiusGeorge after the [Imperius Curse](http://harrypotter.wikia.com/wiki/Imperius_Curse)
and the adorable [Curious George](http://pbskids.org/curiousgeorge/) character. It's remote mind control
like `imperio` and curiously amazing like our monkey friend. Forget having testing monkeys manually
verifying your project, run an unforgivable curse on Curious George instead!

Don't write:

<script src="https://gist.github.com/t413/769326b5c260fc6f1c1f.js" type="text/javascript"> </script>

Instead, in ruby write this:
<script src="https://gist.github.com/t413/91816cb4c8bf86376b3d.js" type="text/javascript"> </script>

##How it works
Most of the work is done in [LanguageAdapter.java](https://github.com/lookout/ImperiusGeorge/blob/master/src/imperiusgeorge/backend/LanguageAdapter.java).
Here's where we make a single method `run` introspectively run any available method on any
class or object with any arguments. `run` takes three parameters, 'on', 'method', and 'arguments'.

1. On first tries to find a class via the `findClass()` method (It'll work if it's a fully-qualified
    class name like 'com.android.uiautomator.core.UiDevice'). If that succeeds it'll run The method statically.
    * If the method is 'new' it'll run the constructor it can find with the given arguments.
    * If it can't find the class it'll try looking up the 'on' string in a `HashMap<String,Object>` that
        stores previously returned complex objects.
2. It then loops through each method available, finding ones with the same name and matching number of arguments.
    * adaptArgs() then tries to adapt, convert, or cast the provided arguments to the method's contract
    * If it throws an `IllegalArgumentException` it tries more matching methods (to support overloading).
3. Finally it calls the method and calls `adaptReturn()` on the result
    * If the returned value is a java.lang* object it'll json-serialize it, returning it back to ruby.
    * A planned improvement is to also return serializable arrays and maps.
    * If the object is not directly returnable it stores it in a `HashMap<String,Object>` and returns a
        unique hash key. That's how the third option for the 'on' parameter works.


##We, for one, welcome our new overlords
At Lookout we've put this functionality into our checkin verification process on jenkins.
If you write something on the client it'll run an unsigned production build on real devices first
before merging.

*- [Tim O'Brien](https://github.com/t413)*
