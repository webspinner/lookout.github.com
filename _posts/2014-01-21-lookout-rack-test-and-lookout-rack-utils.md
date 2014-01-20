---
layout: post
title: "Lookout::Rack::Test and Lookout::Rack::Utils"
tags:
- sinatra
- ruby
- rails
- rack
- testing
- gems
- cucumber
- rspec
---

At Lookout, we build a lot of Rack applications - some on Rails, some on
Sinatra.  One of the nice things about Sinatra is that it is much simpler and
lighter-weight than Rails. There's more you have to build/include for yourself,
yes, but
it cuts down on the magic happening just out of sight and gives you more
control.  This of course has its downsides, sometimes you do want all of the
things Rails gives you.

We recently released two gems containing some of the helper code we've written
for our Sinatra apps:
[`lookout-rack-utils`](https://github.com/lookout/lookout-rack-utils) and
[`lookout-rack-test`](https://github.com/lookout/lookout-rack-test).

`lookout-rack-utils` consists of various utilities having to do with logging (to
a file, or to [logstash](http://logstash.net) via our
[rack-requestash](https://github.com/lookout/rack-requestash) gem),
[Graphite](http://graphite.wikidot.com/) tracking, i18n, and request handling.

`lookout-rack-test` contains code useful for testing Rack apps with RSpec and
Cucumber, examples of which can be found in the repo's
[`spec/`](https://github.com/lookout/lookout-rack-test/tree/master/spec) and
[`features/`](https://github.com/lookout/lookout-rack-test/tree/master/features)
directories.

Both gems are under the MIT License; forks and pull
requests are welcome.

Share and enjoy!

*- [Ian Smith](https://github.com/ismith)*
