---
layout: post
title:  "Creating A Metrics Dashboard With Ember.js, Bootstrap, and Rails - Part 1"
date:   2014-04-07 09:00:00
categories: rails ember.js metrics
---

_This is part one of a series on building a metrics dashboard with ember bootstrap and rails. Over the next few weeks
I will be building out more functionality._

There are some services that will provide dashboards and visualizations for you like
[Geckoboard](http://www.geckoboard.com/), but once you need to provide custom analytics and graphs to clients or
co-workers you will need something a little more flexible.

Today we're going to assemble the first pieces of this app by creating a rails server, an Ember.js App and some basic
Bootstrap style.

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

![Home screen](https://jtescher.github.io/assets/creating-a-metrics-dashboard-with-ember-and-rails-part-one/home-screen.png)

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

![Bootstrap Defaults](https://jtescher.github.io/assets/creating-a-metrics-dashboard-with-ember-and-rails-part-one/bootstrap-defaults.png)

The Ember.js App
----------------


We can now add the `ember-rails` and `ember-source` gems for Ember assets and scaffolding to our `Gemfile`.

```ruby
...
# JS framework for creating ambitious web applications.
gem 'ember-rails',  '~> 0.14.1'
gem 'ember-source', '~> 1.5.0'
```

And install the Ember assets with:

```bash
$ rails generate ember:bootstrap
```

We can also remove turbolinks as ember will handle all the routing after the first page load. Your `application.js` file
should look like this:

```js
//= require jquery
//= require_tree .
```

Also I renamed the generated `app/assets/javascripts/application.js.coffee` file to be
`app/assets/javascripts/app.js.coffee` to not conflict with the first application.js file.

Next move the html from `home/index.html.erb` and replace it with the following:

```erb
<% # Rendered Entirely in EmberJS. See app/assets/javascripts %>
```

Then we can move the html generation to ember by placing it in the top level `application.hbs` handlebars file.

```html
<!-- app/assets/javascripts/templates/application.hbs -->
<div class='container'>
  <div class='jumbotron'>
    <h1>Hello from Ember.js!</h1>
    <p>The link to the home page is {{#link-to 'application'}}here{{/link-to}}</p>
  </div>
</div>
```

And the end of part one should be rendered by Ember looking like this:

![Final](https://jtescher.github.io/assets/creating-a-metrics-dashboard-with-ember-and-rails-part-one/ember-home-screen.png)
