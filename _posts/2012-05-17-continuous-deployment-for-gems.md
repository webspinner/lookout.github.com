---
layout: post
title: Continuous Deployment for Ruby Gems
tags:
- ruby
- jenkins
- rubygems
---

As a Ruby shop, [we](https://www.lookout.com/about/careers) have created and
maintain a large number of Ruby gems.

In the earlier days we set up an internal gem repository with clearly
documented SSH credentials and developers were responsible for building and
packaging their gems, then SCPing them over to the primary gem repository.

Over time the list of gems we create and maintain has grown dramatically and
the building, testing and release management of these gems became a bit of a
chore. The biggest issue with the "simple" workflow described above is that it
is *far* too easy for a developer to package up a gem, ship it, and then never
commit their changes into Git.  Projects "downstream" from that gem could find
themselves the unwilling beta testers for untested changes that came straight
from a developer's workstation.

(Not that this *ever* happened[^yarly], Lookout engineers _always_ commit their code
after running a plethora of tests. We're supremely disciplined!)


### Seamless segue to: the workflow


Usually when describing a workflow, it's easier to diagram it out, so I've
charted out the workflow that gems follow with this [amazing ASCII
diagram](http://www.asciiflow.com/#5034072917650760102):

    +----------------+
    | Local Git Repo |
    +--+-------------+
       |
       | git-push(1) for review
       |
       |    +----------+        Code review passes
       +--->|  Gerrit  +---------------+
            +----------+               v
                             +---------------------+
                             | "Gold" GitHub Repo  |
                             +----------+----------+
                                        |
                                        v
                                   +---------+
                                   | Jenkins |
                                   +----+----+
                                        |         +----------------+
                                        +-------->| Gem Repository |
                                Tests pass?       +----------------+



Stepping through this clearly amazing ASCII diagram we can follow a change "A1"
from being committed to deployed:

1. A developer runs tests locally (of course), and creates a commit `A1`
1. The developer pushes `A1` up to [Gerrit](http://code.google.com/p/gerrit/) and
   adds another developer as a reviewer.
1. (*Not diagrammed*) Jenkins integrates with Gerrit, pre-testing the commit.
   Afterward, the other developer will review (and hopefully) okay the change.
1. The originally developer will "submit" his change in Gerrit, which will
   replicate into the "Gold" GitHub repository (master branch).
1. Jenkins learns that a new commit, `A1`, is available for the building and
   testing. Jenkins fires off a job which _effectively_ just runs:

        % rake test
        % rake build

1. If the job successfully completes, we auto-promote the gem to a "release"
   using the [Promoted Builds plugin](https://wiki.jenkins-ci.org/display/JENKINS/Promoted+Builds+Plugin) to
   execute some post-build promotion processes such as:
    * Keep this build forever
    * Publish the archived `.gem` artifact over SCP to the Gem repo with the [SCP Plugin](https://wiki.jenkins-ci.org/display/JENKINS/SCP+plugin)
    * Fire off another job which updates the Gem repo's indexes

1. At the end of this pipeline we typically will update another project's
   `Gemfile.lock` and then go through a similar flow to get the updated gem
   used by downstream projects.

### Conventions we follow

To enable this fairly standard workflow across the *many* gems we maintain, we
have a couple of conventions:

* Inspired by [Semantic Versioning](http://semver.org/) [^semver] all gemspecs typically
  have the following set as their version:

        gem.version = "1.2.#{ENV['BUILD_NUMBER'] || 'dev'}"

  This uses the `BUILD_NUMBER` environment variable exposed by Jenkins to
  ensure that all gems built by Jenkins have a "marker" allowing them to be
  traced back to a particular build. This has had the bonus effect of reducing
  the number of superfluous commits of "Bump gem version" unless we make
  meaningful API changes to warrant a minor version bump.
* All gems are expected to enumerate *all* their dependencies in the `Gemfile`
  in the repo. This allows Jenkins jobs to self-bootstrap using a job-specific
  [RVM](https://rvm.io/) gemset for the gem build.
* All gems are required to define the `build` and `test` Rake tasks. Naturally
  this means they also must include a Rakefile. More often than not the
  Rakefile looks just like this:

        require 'bundler/gem_tasks' # Provides `build` task
        require 'rspec/core/rake_task'
        RSpec::Core::RakeTask.new(:test)
        task :default => :test

---


With the above workflow and these conventions in place, we've reaped a couple
of benefits, the most obvious one has been the severely reduced time it
takes for new gems to be created and incorporated into production systems.

At a higher level, we've gained more confidence in our gems with this emphasis
on testing, traceability and code review baked into the process from the start.

That confidence pays dividends when looking at moving larger, more complex
systems towards continuous deployment, which we'll cover in later postings.



\- [R. Tyler Croy](https://github.com/rtyler/)


[^yarly]: This has totally happened
[^semver]: Contrary to what semver.org describes, the "patch number" does not
get reset to zero when the minor or major versions get bumped. I am personally
okay with this given the added benefit of "deep traceability" that the build
number from Jenkins provides for gems referenced in production.
