---
layout: post
title: Integration testing with Foreman
tags:
- ruby
- opensource
- foreman
---

At Lookout we find ourselves building more and more APIs and backend services
these days. Naturally we would like to be certain that everything will work
fine and dandy once it has been deployed. The reality of building out a
service-oriented architecture is that you not only have to expect failure to
happen, you have to plan and test for it.

As of late we've been using a tool called
[Foreman](https://github.com/ddollar/foreman) for some projects to manage their
own "development stacks." A single service might be composed of a
`redis-server` instance, a MySQL database and a Rails or
[Sinatra](http://sinatrarb.com) application.

Managing this with Foreman is easy enough, we would create a `Procfile` with
the contents:

    web: ruby app.rb
    redis: redis-server -c config/redis.conf
    mysql: ./script/run-mysql-ramdisk


When we run `foreman start`, Foreman will manage bringing all of these services
online at once, then when you Ctrl-C `foreman` it will bring down all of the
servers appropriately.

That's great for simple local development and testing, but what about with
integration testing the service?


## Meet Test Engineer

The [Test Engineer](https://github.com/lookout/testengineer) gem builds on top
of Foreman and adds some basic testing functionality. Currently it's only been
used with [Cucumber](http://cukes.info) but it could easily be incorporated
into other acceptance testing set ups.

With Test Engineer you can use your existing `Procfile` to start and stop the
entire stack with `TestEngineer#start_stack` and `TestEngineer#stop_stack`.


### With Cucumber

If you're already using Cucumber, this becomes very easy to incorporate into
existing Features with the `@testengineer` tag:


`features/support/env.rb`:

    require 'testengineer/cucumber'

`features/user_login.feature`:

    @testengineer
    Feature: Log in to mylookout.com
      In order to find or scream my phone
      As a registered Lookout user
      I should be able to log into the user area

      Scenario: With a valid email and password
        Given I am a registered user
        When I log in to Lookout
        Then I should see my devices listed
        And I should see my news feed


Test Engineer will bring up the entier stack defined in your `Procfile` for
each and every scenario listed, providing a good isolated test environment for
your integration tests.


### Choas Engineer

Test Engineer also allows you to arbitrarily turn off services during the
scenario, which allows for some interesting fault tolerance testing. You can
define a simple step which invokes `TestEngineer#stop_process(name)`, e.g.:

    Given /^the cache server is offline$/ do
      TestEngineer.stop_process('memcache')
    end

Then in my Cucumber `.feature` file I can turn off the memcached service
mid-way through the test to verify a fault tolerance condition:


    @testengineer
    Feature: Survive cache service degradation

      Scenario: Locate my device
        Given I am a registered user
        And I have an Android device
        And the cache server is offline
        When I locate my device
        Then my device should attempt to locate


That's about all there is to integration and fault tolerance testing with
Foreman and Test Engineer!

----

*Errata:* Test Engineer currently relies on some goofy hacking with some
Foreman internals, which is part of the reason it cannot arbitrarily *start* a
process after it has been stopped. I am currently working on making Foreman
more easily embeddable with the author [David
Dollar](https://github.com/ddollar).




\- [R. Tyler Croy](https://github.com/rtyler/)
