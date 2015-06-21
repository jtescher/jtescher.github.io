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


## Adding Vagrant

Up until now this has been a pretty typical development process for anyone interacting with a Rails app. You will notice
that anyone who wants to work on your blog has to do a few things now just to get it up and running. Even though this
app couldn't be simpler, they have to have the right version of Rails installed and all of the other gems in your
`Gemfile` as well as the right version of PostgreSQL. If you make any changes to either of those things, all other
developers will have to manually update their dependencies. Ugh.

A great solution to this problem is to use [Vagrant](https://docs.vagrantup.com) to manage your dependencies for you.
This will create an isolated development environment for you in a [virtual
machine](https://en.wikipedia.org/wiki/Virtual_machine), and at any point if things aren't working properly or if major
changes get made, you can simply destroy and re-create the whole thing from scratch.

Getting Vagrant installed on your machine is simple with [Homebrew](http://brew.sh).

```bash
  $ brew install caskroom/cask/brew-cask
  $ brew cask install virtualbox
  $ brew cask install vagrant
```

Now that we have [VirtualBox](https://www.virtualbox.org) installed, we can use [Chef](https://www.chef.io) through
Vagrant to provision the VM's. Let's install Chef, the [ChefDK](https://downloads.chef.io/chef-dk), and the cookbook
manager plugin [vagrant-berkshelf](https://github.com/berkshelf/vagrant-berkshelf) as the final part of our setup.

```bash
  $ brew cask install chefdk
  $ vagrant plugin install vagrant-berkshelf
```

## Configuring Vagrant

Vagrant can be configured simply through a `Vagrantfile` at the root of your project. Let's add one now for this
project.

```ruby
Vagrant.configure(2) do |config|

  # Use ubuntu base
  config.vm.box = "ubuntu/trusty64"

  # Forward Rails port
  config.vm.network "forwarded_port", guest: 3000, host: 3000

  # Configure chef recipes
  config.vm.provision :chef_zero do |chef|
    chef.json = {
      'postgresql' => {
        'password'  => {
          'postgres' => 'iloverandompasswordsbutthiswilldo'
        }
      }
    }
    chef.run_list = [
      # Install Ruby
      "recipe[ruby-ng::dev]",

      # Install Node.js for rails assets
      "recipe[nodejs::default]",

      # Install PostgreSQL DB
      "recipe[postgresql::server]"
    ]
  end

end
```

To download and version the cookbooks used by chef we can add a `Berksfile` at the root of your project.

```ruby
source 'https://supermarket.getchef.com'

cookbook 'ruby-ng', '~> 0.3.0'
cookbook 'postgresql', '~> 3.4.20'
cookbook 'nodejs', '~> 2.4.0'
```

And the final step is to install the cookbooks with:

```bash
  $ berks install
```

## Running Vagrant

Now for the magic part. To start your new virtual development environment run 

```bash
$ vagrant up
```

Now that your environment has been created, let's ssh into it and get our rails app running.

```bash
$ vagrant ssh
$ cd /vagrant/
$ bundle install
$ sudo -u postgres createuser -s vagrant
$ rake db:create
$ rake db:migrate
```

Your environment is now ready!

``
$ rails server -b 0.0.0.0
```

You can now if we open [localhost:3000/posts](http://localhost:3000/posts), and see your posts scaffold served from the
Vagrant box!

![Post scaffold](https://jtescher.github.io/assets/improve-your-dev-environment-with-vagrant/posts-scaffold.png)
