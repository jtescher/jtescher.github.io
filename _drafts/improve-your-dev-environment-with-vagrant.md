---
layout: post
title:  "Improve Your Dev Environment With Vagrant"
date:   1970-01-01 00:00:00
categories: Vagrant DevOps Chef
---

![Vagrant Logo](https://jtescher.github.io/assets/improve-your-dev-environment-with-vagrant/vagrant-logo.png)

Installing development dependencies for all of your company's applications can be a pain. As a developer this is a
nuisance that wasts time and breaks your flow, and as a designer this can be so frustrating that it stops you from
running applications altogether.

One example could be if your company is a Ruby on Rails shop, you may have several versions of Ruby running in
production as you upgrade them individually to the newest version. This means that when people want to run an app
locally, they need to have the app's current version of Ruby installed via a tool like [RVM](https://rvm.io) or
[rbenv](https://github.com/sstephenson/rbenv). If you are using a database like [PostgreSQL](http://www.postgresql.org)
in production and want to mirror that configuration in your local development environment to find bugs earlier in the
process (a practice which I would encourage), then you also might need to have multiple versions of PostgreSQL
installed. All of these individually versioned development dependencies need to be kept up to date as things get
upgraded and even as a single developer working on a few applications this can become a mess.

An excellent solution to this problem is to use [Vagrant](https://docs.vagrantup.com) to isolate dependencies and their
configuration into a single disposable, consistent environment that can be created and destroyed with a single command.
In tis post I will show you how the required current versions of Ruby and PostgreSQL for example can be added and
configured easily to produce a single easily reproducible and isolated environment using Vagrant.

All code for this post can be found at [github.com/jtescher/vagrant-rails](https://github.com/jtescher/vagrant-rails).
