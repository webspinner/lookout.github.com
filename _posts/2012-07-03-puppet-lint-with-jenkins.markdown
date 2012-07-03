---
layout: post
title: Using Puppet Lint with Jenkins
tags:
- puppet
- jenkins
- continuoussdeployment
- howto
---


As the Puppet and Chef developer communities have matured, there has been
an increased emphasis on style and sanity checking, also known as "linting."

In the Chef community the hammer of choice is
[foodcritic](http://acrmp.github.com/foodcritic/), while Puppet users have
[puppet-lint](https://github.com/rodjek/puppet-lint/) to rely on.


<img src="/images/post-images/puppet-lint-jenkins/puppet-lint-build-page.png"
alt="Puppet Lint Warnings in Jenkins" align="right"/>


Console-based warnings are great for local development, but since we're already
running all our Puppet code through Jenkins for validation, why not let Jenkins
track linting as well? This can be easily accomplished with the Jenkins [Warnings
plugin](https://wiki.jenkins-ci.org/display/JENKINS/Warnings+Plugin) and a
little bit of set up work.



### Steps

1. First make sure you install the Warnings plugin from the Jenkins "Manage
   Plugins" page
1. Add a special task to your `Rakefile` which will invoke puppet-lint with a
   specific log format, [as shown here](https://gist.github.com/3041462)
1. Invoke the new Rake task in the build (`rake lint:ci`)
1. Configure the Jenkins job to look for the puppet-lint warnings
    ![Warnings scan configuration](/images/post-images/puppet-lint-jenkins/warnings-plugin-scan.png)
1. Run some builds, and enjoy your new puppet-lint reports and trend graphs!


---

On the build pages you will be able drill into an overview report such as the one
pictured below, hyperlinked to allow you to dig deeper into the specific warnings

![Build-specific warnings report](/images/post-images/puppet-lint-jenkins/puppet-lint-warnings-build-page.png)

---

On the job page (http://jenkins/job/my-puppet-module) you will also have a
cross-build trend graph to give an indication of the trend of warnings as time
goes on.

![Overall Warnings Trend](/images/post-images/puppet-lint-jenkins/puppet-lint-trend.png)

---


That's all there is to it! There isn't any more setup required to get
puppet-lint and Jenkins working together nicely.  The hardest part of the whole
process seems to be resolving all of the warnings and the second hardest part seems to
be keeping the warnings at zero as time progresses.

The [Warnings
plugin](https://wiki.jenkins-ci.org/display/JENKINS/Warnings+Plugin) allows for
a lot more configuration than I've covered in this post, so be sure to explore
it's more "Advanced" options once you're up and running!


\- [R. Tyler Croy](https://github.com/rtyler/)

