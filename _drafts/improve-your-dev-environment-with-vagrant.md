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

## Creating The Application

For this example we will create a [Rails](http://rubyonrails.org) app that has a database dependency. Remember this will
work with any application. Also note that for simplicity we are creating the Rails app before we start using Vagrant, so
you have to have PostgreSQL installed, but once we add Vagrant later in this example it won't be required anymore (if
you're not happy with this you can skip down and install Vagrant first). Let's first generate a new Rails project that
we'll call blog.

```bash
  $ rails new blog --database=postgresql
  $ cd blog
```

Let's now give our blog app a `Post` scaffold so we can see some posts with a title and a body.

```bash
  $ rails generate scaffold post title:string body:string
  $ rake db:create
  $ rake db:migrate
```

Now let's start the server and see what we have so far.

```bash
  $ rails server
```

Now if we open [localhost:3000/posts](http://localhost:3000/posts), we see our functional blog scaffold.

![Post scaffold](https://jtescher.github.io/assets/improve-your-dev-environment-with-vagrant/posts-scaffold.png)
