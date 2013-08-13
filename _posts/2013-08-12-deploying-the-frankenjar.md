---
layout: post
title: "The Frankenjar: The Easiest Ruby App Deployment Ever"
tags:
- sinatra
- ruby
- jruby
- rake
- java
- releng
---

I've had a love/hate relationship with Java and in turn the JVM. I love to hate
them both. Begrudgingly, I can acknowledge the performance and maturity of the
JVM, especially when compared with the simplistic Ruby VM (MRI). Historically,
Lookout has been a very Ruby-oriented company, I would estimate that 80%+ of
our server-side code is Ruby-based (including Chef). It's tremendously easy to
build Ruby projects at Lookout, but the deployment process, production
performance leaves something to be desired.

<img src="/images/jruby_large.png" title="JRuby!" width="200" align="right"/>

When my team started a new project, we looked at the status quo of the Ruby VM
and decided that [JRuby](http://www.jruby.org) was the best, most logical
environment to use in *production*. But for local development, the quick
boot-up time and adequate speed of MRI (1.9.3) made it the better choice. From
the very beginning our tests were running in Jenkins under MRI 1.9.3 and JRuby 1.7.

As the project matured, we needed to address the *deployment* question. The
application consisted of Ruby code, and a modern front-end application
developed using [Chaplin](https://github.com/chaplinjs/chaplin) (an application
framework built atop [Backbone.js](http://backbonejs.org/)). Our needs and
wants were:

 * We need to deploy continuously
 * We need easy rollbacks
 * We need to pre-compile the front-end application from
   [CoffeeScript](http://coffeescript.org/) and
   [Stylus](http://learnboost.github.io/stylus/) to minified JavaScript and CSS.
 * We want to reduce the complexity of the Operations work involved to run the
   application
 * We want to run the application with JRuby
 * We want as much control over the environment in production as possible
 * We need to be able to run an interactive console for debugging


After an afternoon of tinkering with
[Warbler](https://github.com/jruby/warbler#readme), a gem which makes `.jar`/`.war`
qiles from Ruby applications with JRuby, we were able to create the
"**frankenjar**." The frankenjar is a self-contained, self-running version of
the entire application.


## Creating the jar

The first of the two kinds of self-running files that Warbler will create is a
`.war` file that embeds a Java Servlet container. Using [Jetty](http://www.eclipse.org/jetty/) or
[Winstone](http://winstone.sourceforge.net/), combined with the
[jruby-rack](https://github.com/jruby/jruby-rack) gem to glue a standard
[Rack](https://github.com/rack/rack) application to these servlet containers.

The `.war` file is a perfectly acceptable way to run the web application, but
we wanted more than just a web server from the executable.

This led to creating an executable `.jar` file with the Rack application and
other tools packaged up together, such that we would end up with a single,
traceable archive at the end of the process.


The first step to building a frankenjar is defining a `config/warble.rb` file.
The file contains two important pieces of information: the directories/files to
bundle up, and the name of the resulting archive.

<script src="https://gist.github.com/rtyler/6208038.js?file=warble.rb" type="text/javascript">
</script>

Warbler does some helpful auto-detection for Rack applications which gets in
our way in this case. Since we *do* have a Rack application, but we *don't*
want to create a `.war` file, we need to "hide" `config.ru` from Warbler during
the packaging process. To help with this we created the following script to
properly pre-compile our front-end assets, and create the `.jar` file.

<script src="https://gist.github.com/rtyler/6208038.js?file=package.sh" type="text/javascript">
</script>


Executable jars created by Warbler will automatically invoke the first file
listed in the source tree's `bin/` directory when the `.jar` file is run. We
can use this to our advantage and provide a simple task invocation mechanism in
that file.

After an hour or two of cobbling something together "manually", I decided to
just use [Rake](http://rake.rubyforge.org/) as the task invocation mechanism.
By using Rake we are able to re-use the same exact task definitions that we're
using locally for performing tasks such as database migrations, starting up
test consoles, or even running the server itself.

Coaxing Rake to properly load the `Rakefile` embedded within the `.jar` file
was a bit of a challenge, and *does* require monkey-patching Rake:

<script src="https://gist.github.com/rtyler/6208038.js?file=franken.rb" type="text/javascript">
</script>


With all that put in place, invoking `./scripts/package.sh` will create a nice,
entirely self-contained, runnable `.jar` file which can be shuffled around as
needed.


## Running the jar

The system requirements for *running* the application are dead-simple: the Java
Runtime Environment. The Chef code required to set up machines for the
application was incredibly simple: install the JRE, make sure a Java process
runs forever, rotate logs as needed.

Within the `.jar` file, we're pretty much boot-strapping with Rake, the command
line interface should look familiar:

    -> % java -jar franken.jar -T
    (in /home/tyler/source/lookout/git/frankenjar)
    rake console:pry     # Run a simple pry-based console
    rake db:migrate      # Migrate the database
    rake db:rebuild      # Rebuild the entire database
    rake db:schema:dump  # Dump the new schema to the terminal
    rake server:puma     # Run the application with Puma
    rake server:webrick  # Run the application with WEBrick (development only)
    -> %

There are *many* other Rake tasks in the project, but most of them are related
to local development and testing. it should be noted that in the
`config/warble.rb` above only includes a select few `.rake` files to be bundled
inside the archive, no sense running unit tests in production.

Using Rake tasks in this way has the added benefit of ensuring the development
team can use the same tasks (and codepaths) locally when building the
application, as will be run in production.


## Deploying the jar


As you might imagine, shipping a *single* `.jar` file to production is rather
simple, as it should be. Where the additional complexity comes in is during the
updating of the running app instances. While traditional `.war` containers will
support hot restart, we have to manage this ourselves to support
zero-downtime deployments.

Unfortunately the JVM warm-up period, and loading
the entire runtime can take between 5-15s depending on load, so we walk through
app servers, restart them, and block until they're back online. To accomplish
this we use a little bit of [Capistrano](http://www.capistranorb.com/):

    desc 'Restart server'
    task :restart_apps, :roles => :app do
      find_servers_for_task(current_task).each do |server|
        puts "Running a server restart on #{server.host}"
        run "#{sudo} /srv/frankenjar/upstart-helper.sh", :hosts => [server.host]
      end
    end

Combined with a little bit of shell scripting to block the deployment until the
frankenjar is serving requests again:

<script src="https://gist.github.com/rtyler/6208038.js?file=upstart-helper.sh" type="text/javascript">
</script>


## It's Alive!

With the frankenjar being created as the last step of our internal build and
testing pipeline, we take `.jar` files from Jenkins and deploy them just about
*anywhere* with minimal overhead. The `.jar` is the ultimate reproducible
artifact, developers, QE, and operations can all access and use the **exact**
same code when reproducing any issues that may arise.

The approach allows the developer team to upgrade JRuby, any dependent gems,
switch out web servers, and reuse all the powerful profiling tooling available
in the Java community when doing it.

While the frankenjar is slightly off the beaten path in the JRuby community, but
definitely worth the extra little bit of work to get there.



*- [R. Tyler Croy](https://github.com/rtyler)*
