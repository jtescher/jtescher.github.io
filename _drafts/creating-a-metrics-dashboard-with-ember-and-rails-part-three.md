---
layout: post
title:  "Creating A Metrics Dashboard With Ember.js, Bootstrap, and Rails - Part 3"
date:
categories: rails ember.js metrics
---

*This is part three of a series on building a metrics dashboard with Ember, Bootstrap, and Rails. Over the next few weeks
I will be building out more functionality and writing posts to cover that. If you haven't read
[part one](/creating-a-metrics-dashboard-with-ember-and-rails-part-one) and
[part two](/creating-a-metrics-dashboard-with-ember-and-rails-part-two) then that's a good place to start.*

In [part two](/creating-a-metrics-dashboard-with-ember-and-rails-part-two) we ended up with an Ember app that rendered
dynamic tables. If you followed along your page should now look like this:

![Part Two Final](https://jtescher.github.io/assets/creating-a-metrics-dashboard-with-ember-and-rails-part-two/currency-helpers.png)

Today we're going to add some graphs and see how Ember components work.

Remember, if you get stuck you can find all of the code for this post at
[github.com/jtescher/example-ember-rails-dashboard](https://github.com/jtescher/example-ember-rails-dashboard).

Adding charts
-------------
