---
layout: post
title: Cheffing
tags:
- chef
- chefspec
- minitest
- continuous integration
---

At Lookout we're deploying [Chef](http://www.opscode.com/chef/) to manage our infrastructure. We've made several decisions about how we'll use Chef:
* Chef server, not chef-solo
* Each cookbook in a separate repo
* Cookbook development in VMs
* Unit- and integration-tests for our cookbooks and chef installation
* Continuous integration

### Chef server, not chef-solo
You can run chef in two modes: [chef-solo](http://wiki.opscode.com/x/9QAS) or with a [chef server](http://wiki.opscode.com/x/5QAS). Chef-solo is simply your cookbooks plus the chef-client software; it configures the machine it runs on with no need to contact any other machine. The beauty of chef-solo is its simplicty: you can tar up your cookbooks, download them to a machine, install chef and run it, and you're good to go. The downside is that everything the cookbooks need must be in the tarball or available on the machine; there's no access to a central repository or directory service. Using chef server requires more up-front effort; you have to set up a chef server and upload your cookbooks to the chef server; to configure a client machine you install chef, configure it to point at the chef server, and the client downloads the cookbooks from the chef. The power of chef server, though, is that cookbooks can use [search](http://wiki.opscode.com/x/tgAS). Want to configure your Nagios server using chef server? You can generate your host and service definitions by using search to iterate all of your hosts. You can't do that with chef-solo. For us, using chef-server was a no-brainer.

### Each cookbook in a separate repo
Chef's original structure had a single repo that contained all of the cookbooks:

    chef-repo
      cookbooks
        apache2
        build-essential
        ...
        sudo
        yum

This made workflow simple: each developer could group their changes for some spiffy new feature that affected several cookbooks into one or a few commits. But, it was messy to use upstream cookbooks (you're not really going to write your own apache2 cookbook, you're going to grab the [community's cookbook](http://community.opscode.com/cookbooks/apache2)). Since we're using a mix of upstream cookbooks and our own, we decided to put each cookbook in its own repo. Our initial implementation used `git submodule`. That turned out to be untenable in practice: git submodules are best suited for modules that don't change very often, and we're revving our cookbooks constantly. So, we got rid of the submodules and wrote Rake tasks to do the grunt work of creating cookbooks, keeping your local copy up-to-date, etc. The jury's still out on whether our current approach will give us a (mostly) painless workflow, but I think we're close.

### Development in VMs
Since you need a chef server and a chef client to develop and test, we created [Vagrant](http://vagrantup.com)-based tooling to make our workflow simple. It basically makes it dead simple to make a change to a cookbook, upload it to your chef server VM, and test it on your chef client. We're building something similar for AWS and possibly Openstack using [fog](http://fog.io/index.html).

### Unit- and Integration-testing
A dirty little secret of the Ops world is that we've been slow to adopt test-driven development. At Lookout, we wanted to start our chef work with TDD. So we insist that every cookbook have tests. One challenge of testing provisioning tools like Chef is that provisioning nodes can be time-consuming. So we started with [chefspec](https://www.relishapp.com/acrmp/chefspec/docs), which enables us to write RSpec tests for our chef cookbooks that don't require provisioning (or, in Chef parlance, converging) a node. Chefspec can't test everything (it's really more for unit-testing) but it's great for confirming that a gem gets installed under the proper circumstances or that a config file is generated properly. (You can see some simple examples of how to use chefspec [here](https://github.com/jimhopp/chefspec_exploration)).

We're just starting to use [Minitest Chef Handler](https://github.com/calavera/minitest-chef-handler) for integration testing. Minitest-chef-handler enables you to run Minitest tests at the end of a chef-client run to confirm that the node is correctly configured. Right now we only run the Minitest tests in test environments, but we're thinking about running them in production to ensure each node is correctly configured.

### Continuous Integration
We're also big fans of [CI](http://en.wikipedia.org/wiki/Continuous_integration). We've structured our chef workflow so that every cookbook commit submitted for review to our [Gerrit](http://code.google.com/p/gerrit/) instance triggers a build in [Jenkins](http://jenkins-ci.org/) that runs the chefspec tests for that cookbook. Changes to the chef-repo (adding a cookbook, updating a role) trigger a build of the chef-server and chef-client; we also build the full stack nightly. We have separate jobs for integration tests on specific system types (e.g., our Mongo instances). (I'm presenting a [session](http://chefconf2012.sched.org/event/b2b1a41277c11c865d55b733b4814c1a) at [#ChefConf](http://chefconf.opscode.com/) on our approach to testing.)

Chef is a great tool for infrastructure management, and incorporating techniques like CI have made it easy to fit in into our development and deployment workflow.

\- [Jim Hopp](https://github.com/jimhopp/)
