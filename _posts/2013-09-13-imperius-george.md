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

TL;DR; check out **[ImperiusGeorge](https://github.com/lookout/ImperiusGeorge/)**
and *[ImperiusGem](https://github.com/lookout/ImperiusGem/)* on github and drive
Android UI-Automator remotely in ruby!

Android is awesome. But its diverse ecosystem can be insanely tough to target. There are dozens of manufactures, hundreds of carriers, thousands of models, several OS iterations and an inordinate amount of OEM 'improvements' (touchwiz, sense, etc). There are many tools to apply automated testing to make development easier and more stable like Robotium, Robolectric and now UI-Automator. We've made a new way to run UI-Automator to achieve the holy grail of testing: end-to-end tests on production phones with *production builds*.

##Ho, UI-Automator!
[UI-Automator](http://developer.android.com/tools/help/uiautomator/index.html) is Google's new UI testing tool that's included in 4.1+ phones. It's powerful, well-integrated, and unlike [Robotium](https://code.google.com/p/robotium/)
it runs on production builds of *all* apps on production android phones!

Its problem is flexibility. Tests must run junit-style and don't have filesystem access (there is a workaround) or package-manager or application-manager access. It *is* possible to use the Apache network libraries and talk to servers– potentially doing end-to-end testing but there are a couple drawbacks. You can't quickly and easily install and remove apks or use many of the developer tools available via ADB. It's also difficult to coordinate multiple devices.

We wanted a way to keep our entire test suite in Ruby, running on a coordinating computer with adb access and multiple connected devices.

<div style="text-align:center"><img src="/images/post-images/imperius-george/george.jpg" alt="funny logo" /></div>
##Enter ImperiusGeorge
We wrote a tool to let us write Ruby on that runs on a coordinating computer that controls any UI-Automator enabled device over USB. You write ruby on your desktop/server and it'll drive the java runtime
on Android.

It's called [ImperiusGeorge](https://github.com/lookout/ImperiusGeorge/) after the [Imperius Curse](http://harrypotter.wikia.com/wiki/Imperius_Curse) and the adorable [Curious George](http://pbskids.org/curiousgeorge/) character. It's remote mind control like `imperio` and curiously amazing like our monkey friend. Forget having testing monkeys manually verifying your project, run an unforgivable curse on Curious George instead!

Instead of writing this in Java, compiling, pushing to your phone, then running via ADB:

<script src="https://gist.github.com/t413/769326b5c260fc6f1c1f.js" type="text/javascript"> </script>

Instead write this in ruby alongside your server interaction code:
<script src="https://gist.github.com/t413/91816cb4c8bf86376b3d.js" type="text/javascript"> </script>

##How it works
We needed a way to drive the Java runtime via text HTTP Get requests. Most of the work is done in [LanguageAdapter.java](https://github.com/lookout/ImperiusGeorge/blob/master/src/imperiusgeorge/backend/LanguageAdapter.java). Here's where we make a single method `run` introspectively run any available method on any class *or object* with any arguments. `run` takes three `String` parameters, 'on', 'method', and 'arguments'.

1. `on` first tries to find a class via the `findClass()` method (It'll work if it's a fully-qualified
 class name like 'com.android.uiautomator.core.UiDevice'). If that succeeds it'll run The method statically.
    * If the method is 'new' it'll run the constructor it can find with the given arguments.
    * If it can't find the class it'll try looking up the instance (see step 3)
2. It then loops through each method available, finding ones with the same name and matching number of arguments.
    * adaptArgs() then tries to adapt, convert, or cast the provided arguments to the method's contract
    * If it throws an `IllegalArgumentException` it tries to match more methods (to support overloading).
3. Finally it calls the method and calls `adaptReturn()` on the result
    * If the returned value is a java.lang.* object it'll json-serialize it, returning it back to ruby.
    * A planned improvement is to also return serializable arrays and maps.
    * If the object is not directly returnable it stores it in a `HashMap<String,Object>` and returns a unique hash key. That's how the third option for the 'on' parameter works.

*A note about bundled code*: UI-Automator is actually a system tool, invoked via `adb shell`. It loads compiled jars' classes and introspectively runs the parameter passed class (it's junit extended). Because of this architecture you can't include separate libs (oh, we tried). We wanted to use gson but chose  [simple-json](https://code.google.com/p/json-simple/) for simplicity. We also used [NanoHTTPD](https://github.com/NanoHttpd/nanohttpd) for the web server functionality. These libraries simplified development a lot! We bundled the source to help simplify yours too, it'll compile and run as-is.

##We, for one, welcome our new overlords
At Lookout we've put this functionality into our checkin verification process on jenkins.
If you write something on the client it'll run an unsigned production build on real devices first
before merging.

*- [Tim O'Brien](https://github.com/t413)*
