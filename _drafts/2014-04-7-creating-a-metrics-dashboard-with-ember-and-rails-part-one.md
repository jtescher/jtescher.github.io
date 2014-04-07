---
layout: post
title:  "Creating A Metrics Dashboard With Ember.js, Bootstrap, and Rails - Part 1"
date:   2014-04-07 09:00:00
categories: rails ember.js metrics
---

Surfacing simple metrics can be easy. If you have an existing Rails app, you can simply add an admin-only section
that renders some simple tables. This works for a little while, but as you start exposing more information and
your data set grows, this gets out of hand.

There are some services that will handle parts of this for you like [Geckoboard](http://www.geckoboard.com/), but once
you need to expost a partner dashboard, or use charts that geckoboard does not provide, a custom solution is required.

Today we're going to talk about a fully client side solution using Ember.js. We will expose the metrics we collect from
our various services via API's and collect them in a rails app specific API.

You can find all of the code for this post at
[github.com/jtescher/example-ember-rails-dashboard](https://github.com/jtescher/example-ember-rails-dashboard).

The Rails Server
----------------

We want to be able to expose all of our stats from various internal and third party services with a consistent API so
this server will primarily handle authentication and data transformation. This server could just as easily be written
in [Node.js](http://nodejs.org/) or any language/framework, but for now we'll stick with Rails.

Create the server (skipping ActiveRecord and TestUnit because we don't need a database for this app and we will be
testing with Rspec):

```bash
$ rails new dashboard --skip-test-unit --skip-active-record
```

Then we want to add the route that will render our dashboard. We will use the home controller and the index action:

```bash
$ rails generate controller Home index --no-helper --no-assets
```

We wil route everything to Ember, so replace your `config/routes.rb` file with the following:

```ruby
# config/routes.rb
Rails.application.routes.draw do
  root 'home#index'
end
```

If you start your server now with `$ rails server` and open up [localhost:3000](http://localhost:3000) you should see
the following:

![Home screen](http://localhost:4000/assets/creating-a-metrics-dashboard-with-ember-and-rails-part-one/home-screen.png)

The Theme
---------

We will be using stock [Twitter Bootstrap](http://getbootstrap.com/) for this project so let's add the `bootstrap-sass`
gem to our `Gemfile`.

```ruby
...
# Front-end framework for developing responsive, mobile first projects on the web.
gem 'bootstrap-sass', '~> 3.1.1'
```

And configure it by creating a `bootstrap_config.css.scss` file.

```sass
// app/assets/stylesheets/bootstrap_config.css.scss
@import "bootstrap";
```

Then we can change the `home/index.html.erb` view to the following:

```html
<div class='container'>
  <div class='jumbotron'>
    <h1>Hello from Twitter Bootstrap!</h1>
    <p>Lots of predefined style that makes things easy to prototype!</p>
  </div>
</div>
```

And when you restart the server you should see:

![Home screen](http://localhost:4000/assets/creating-a-metrics-dashboard-with-ember-and-rails-part-one/bootstrap-defaults.png)
