---
layout: post
title: Jasmine Testing with Sauce Labs
tags:
- jasmine
- selenium
- saucelabs
---


Over the past few months, members of our front-end team have been busy
revamping much of our web application. While I can't tell you about the
specifics of what is coming down the pike, I can tell you about some of the
technologies that we're using along the way, most notably: Jasmine.

For the unfamiliar, Jasmine is a JavaScript-based unit testing library which
can help make test-driving full applications in JavaScript feasible.


With some of the recent changes that I still can't tell you about (sorry), Jasmine
has become an important part of our application development stack. We now have a
need to run our Jasmine unit tests as part of our "standard" continuous
integration workflow and release process. This means we must run Jasmine tests
when we run our normal unit tests, Rails functional tests and Selenium tests.

Since we're already big users of the Selenium testing service provided by
[Sauce Labs](http://saucelabs.com), it made sense to build on top of their
infrastructure to run our Jasmine tests.

Using the [Sauce gem](https://github.com/saucelabs/sauce_ruby) and the [Jasmine
gem](https://github.com/pivotal/jasmine-gem) from Pivotal Labs, we can
effortlessly use either a local browser or a remote browser hosted by Sauce.
The "trick" is some of the Rake tasks provided by the two gems:

    -> % rake -T jasmine
    rake jasmine                 # Run specs via server
    rake jasmine:ci              # Run continuous integration tests
    rake jasmine:sauce           # Execute Jasmine tests in a Chrome browser on Sauce Labs
    rake jasmine:sauce:all       # Execute Jasmine tests in Chrome, Firefox and Internet Explorer on Sauce Labs
    rake jasmine:sauce:chrome    # Execute Jasmine tests in chrome
    rake jasmine:sauce:firefox   # Execute Jasmine tests in firefox
    rake jasmine:sauce:iexplore  # Execute Jasmine tests in iexplore


The first two tasks (`jasmine` and `jasmine:ci`) are provided by the Jasmine
gem, while the rest are provided by the Sauce gem (after requiring
`sauce/jasmine/rake` in our Rakefile).

The only pre-requisite to running the `jasmine:sauce` tasks is that you need a [Sauce
Connect](http://saucelabs.com/docs/sauce-connect) tunnel open, to do this you
only need to open a separate terminal to run: `sauce connect`.

Once the tunnel is up and running, invoking `rake jasmine:sauce` will
automatically spin up a Jasmine test server, a Sauce Labs VM with Chrome, and
run your tests!

    -> % rake jasmine:sauce
    Waiting for jasmine server on 3001...
    jasmine server started.
    Starting job named: Jasmine Test Run 1341870335
    Waiting for suite to finish in browser ...
    ..........................

    Finished in 19.95 seconds
    26 examples, 0 failures
    -> %


### Jasmine on Jenkins

Running tests locally is fine, but nothing compares to running those tests in
[Jenkins](https://jenkins-ci.org). At Lookout, we use both
[Gerrit](https://code.google.com/p/gerrit/) and Jenkins heavily throughout our
development process, so being able to kick-off Jasmine tests from Jenkins is
very important.

To accomplish this we have a dedicated slave for running nothing but Jasmine
tests. The reason for dedicating a slave is so that we can keep a Sauce Connect
tunnel open across multiple test runs in order to keep tests as fast as
possible.

We use the [sauce-connect puppet
module](http://forge.puppetlabs.com/rtyler/sauceconnect) to run the tunnel, and
then bind all our jobs to the "jasmine" label to make them only run on that
slave.  One thing we found is that the Sauce Connect tunnel can get a bit
"wonky" if it's not regularly restarted, so we had to make sure the tunnel is
torn down every night.

Our node (in Puppet) for the builder looks something like this:

    node /^jasmine-builder$/ inherits server {
      class {
        'sauceconnect' :
          username => 'REDACTED',
          apikey   => 'EVENMOREREDACTED';
      }

      cron {
        'restart-sauceconnect' :
          ensure  => present,
          command => 'service sauce-connect restart',
          user    => 'root',
          minute  => 0,
          hour    => 3,
          require => Class['sauceconnect'];
      }
    }



By splitting off our Jasmine tasks to run on this dedicated Jasmine builder, it
only takes about 2 minutes to run all the tests.


---

With a little bit of elbow grease we brought together a lot of the existing
tools we had in our developer toolchest and now we're able to provide first-class
support for building, testing and deploying JavaScript applications.

Not bad for a "mobile company" if you ask me!


\- [R. Tyler Croy](https://github.com/rtyler/)
