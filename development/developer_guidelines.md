---
layout: page
#title: Developer Guidelines
permalink: /development/guidelines.html
---

## Naming Conventions

* All binaries should start with either **handler**, **check**, **metrics**, **extension**, or **mutator** depending on their primary function.  This is done to ensure that a user can tell from the command what the primary action of the script is.  It also makes things easier for infrastructure tools.

* Please format the names of scripts using dashes to separate words and with an extension (`.rb`, `.sh`, etc), and make sure they are `chmod +x`'d. Extensions are unfortunately necessary for Sensu to be able to directly exec plugins and handlers on Windows.  There is a rake task that is run by travis that will automatically make all files in */bin* executable.

* Any repos created need to follow te format of *sensu-plugins-app*, where *app* is the group name such as windows, disk-checks, or influxdb.  The exception to the rule are repos used for the site or tooling such as GIR or sensu-plugins.github.io.  This is done so that the rake tasks and other automation tools can easily parse Github and effectively work with the ~150+ repos.

## Coding Style

* When developing your plugins please use the [sensu plugin class][1].  This will ensure that all plugins have an identical run structure.

* When using options please use the following structure.  At the very least your option needs to include a description to assist the user with configuration and deployment.

{% highlight ruby %}
option :port,
short: '-p PORT',
long: '--port PORT',
description: 'Port',
default: '1234'
{% endhighlight %}

* Each script should use the following standard header:

{% highlight ruby %}
#! /usr/bin/env ruby
#
#   <script name>
#
# DESCRIPTION:
#
# OUTPUT:
#   plain text, metric data, etc
#
# PLATFORMS:
#   Linux, Windows, BSD, Solaris, etc
#
# DEPENDENCIES:
#   gem: sensu-plugin
#   gem: <?>
#
# USAGE:
#
# NOTES:
#
# LICENSE:
#   <your name>  <your email>
#   Released under the same terms as Sensu (the MIT license); see LICENSE
#   for details.
#
{% endhighlight %}

## Documentation

All documentation will be handled by [Yard][2] and using the default markup at this time. A brief introduction to Yard markup can be found [here][3]. All scripts should have as much documentation coverage as possible, ideally 100%.  You can test your coverage by installing Yard locally and running

{% highlight bash %}
rake yard
{% endhighlight %}

Documentation can always be made better, if you would like to contribute to it, have at it and submit a PR.

## Dependency Management

Dependencies (ruby gems, packages, etc) and other requirements should be declared in the header of the plugin/handler file.  Try to use the standard library or the same dependencies as other plugins to keep the stack as small as possible.  If you have questions about using a specific gem feel free to ask.

## Vagrant Boxes

There is a [Vagrantfile][4] in each repo with shell provisioning that is capable of setting up  the major versions of Ruby using RVM and a sensu gemset for each if you wish to use it.  To get started install [Vagrant][5] then type `vagrant up` in the root directory of the repo or using [GIR][11] type `rake vagrant:up plugin=<app>` from the PROJECT_DIRECTORY.  Once it is up type `vagrant ssh` or `rake vagrant:up plugin=<app>` to remote into the box and then `cd /vagrant && bundle install` to set all necessary dependencies.

The box currently defaults to Ruby 2.1.4 but 1.9.3 and 2.0.0 are available for installation as well, just uncomment them from the install script.  See the file comments for further details.

## Testing

### Linting

**Only pull requests passing Rubocop will be merged.**

Rubocop is used to lint the ruby plugins. This is done to standardize the style used within these plugins and ensure high quality code.  Most [current rules][6] are in effect.  No linting is done on Ruby code prior to version 2x.  See the [.travis.yml][7] and [Rakefile][8] templates in GIR as they are autogenerated upon initial repo creation and may be updated at any given time.

Ruby 1.9.2 and 1.8.7 support has been dropped, the plugins may still function with these versions but no tests will be run against them nor will code, such as hashes, be specifically written or enforced to ensure backwards compatibility.

You can test rubocop compliance for yourself by installing the gem and running `rake test:rubocop plugin=<app>` from the PROJECT_DIRECTORY.  Running `rake test:rubocop_fix plugin=<app>` will attempt to autocorrect any issues, saving yourself considerable time in large files.

If it truly makes sense for your code to violate a rule you can disable that rule with your code by either using

{% highlight ruby %}
# rubocop:disable <rule>, <rule>
{% endhighlight  %}

at the end of the line in violation or

{% highlight ruby %}
rubocop:disable <rule>, <rule>
<code block>
rubocop:enable <rule>, <rule>
{% endhighlight  %}

If you use either of these methods please mention in the PR as this should be kept to an absolute minimum, at times this can be necessary, especially concerning method length and complexity.

### Rspec

Currently we have RSpec3 as a [test framework][9]. Please add coverage for your check.  Checks will not be considered production grade and stable until they have complete coverage.

You can use the included Vagrantfile for easy testing.  All necessary versions of Ruby can be installed with their own dedicated gem sets using RVM.  Just boot up the machine and drop into /vagrant and execute

{% highlight ruby %}
rake default
{% endhighlight  %}

to run all specs and rubocop tests.  RSpec tests are currently run against 2.0, and 2.1.  There are currently no plans to support 1.8.x or test against 1.9.2 and 1.9.3.

This is little bit hard almost impossible for non-ruby checks. Let someone from [team][10] know and maybe can can help.

## Issue and Pull Request Submissions

If you see something wrong or come across a bug please open up an issue.  Try to include as much data in the issue as possible.  If you feel the issue is critical than tag a team member and we will respond as soon as is feasible.

When submitting a pull request please follow the guidelines below for the quickest possible merge.  These not only make our lives easier, but also keep the repo and commit history as clean as possible.

* When at all possible do a  `git pull --rebase` or use [GIR][11] `rake git:pull plugin=<app>` which defaults to rebase, both before you start working on the repo and then before you commit.  This will help ensure you have the most up to date codebase, Rubocop rules, and documentation.  It will also go along way towards cutting down or eliminating(hopefully) annoying merge commits.

If you wish to track the status of your PR or issue, check out our [waffle.io][12].  This single location will allow contributors to stay on top of interwinding issues more effectively.  As the number of repositories grow and issues cross those bounds this will be the main organizational tool for tracking.

Please do not not abandon your pull request, only you can help us merge it. We will wait for feedback from you on your pull request for up to 60 days. A lack of feedback in after this may require you to re-open your pull request.

## Technical Debt

For those who don't deal with or understand technical debt, it is debt incurred when designing or developing software.  All the #FIXME, #HACK, etc littered through a script add up over time, this is your technical debt.

When working on the code if you see an issue and can't fix it right away then tag it either #YELLOW, #ORANGE, or #RED based upon the below guidelines.  This will allow yourself or other to come back later when they have some available cycles.

### Technical Debt Levels

**YELLOW**

* simple issues that require basic Ruby and no more than 4 hours to fix

**ORANGE**

* these may require 4 - 8 hours but still only a basic or intermediate Ruby skillset

**RED**

* may require 8+ hours or some domain specific Ruby skills such as Amazon, or Elastic Search

[1]: https://github.com/sensu/sensu-plugin
[2]: http://yardoc.org/
[3]: http://www.rubydoc.info/gems/yard/file/docs/GettingStarted.md
[4]: https://github.com/sensu-plugins/GIR/blob/master/files/templates/gem/Vagrantfile.erb
[5]: https://www.vagrantup.com/
[6]: https://github.com/sensu-plugins/GIR/blob/master/files/templates/gem/rubocop.yml.erb
[7]: https://github.com/sensu-plugins/GIR/blob/master/files/templates/gem/travis.yml.erb
[8]: https://github.com/sensu-plugins/GIR/blob/master/files/templates/gem/Rakefile.erb
[9]: https://github.com/sensu/sensu-plugin-spec
[10]: https://github.com/orgs/sensu-plugins/people
[11]: http://sensu-plugins.github.io/development/gir
[12]: https://waffle.io/sensu-plugins/sensu-plugins.github.io
