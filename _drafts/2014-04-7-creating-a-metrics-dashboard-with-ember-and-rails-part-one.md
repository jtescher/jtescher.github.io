---
layout: post
title:  "Creating A Metrics Dashboard With Ember.js and Rails - Part 1"
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
$ cd dashboard
$ git init && git add -A && git commit -m "Initial Rails scaffold"
```
