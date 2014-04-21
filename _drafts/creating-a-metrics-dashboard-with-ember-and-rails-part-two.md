---
layout: post
title:  "Creating A Metrics Dashboard With Ember.js, Bootstrap, and Rails - Part 2"
date:
categories: rails ember.js metrics
---

*This is part two of a series on building a metrics dashboard with Ember, Bootstrap, and Rails. Over the next few weeks
I will be building out more functionality and writing posts to cover that. If you haven't read
[part one](/creating-a-metrics-dashboard-with-ember-and-rails-part-one) then go read that first.*

In [part one](/creating-a-metrics-dashboard-with-ember-and-rails-part-one) we ended up with a Rails app that generated
the Ember app that rendered our metrics page. If you followed along your page should now look like this:

![Part One Final](https://jtescher.github.io/assets/creating-a-metrics-dashboard-with-ember-and-rails-part-one/ember-home-screen.png)

Today we're going to add some components and see how Ember keeps all your templates up to date as data changes.

Remember, if you get stuck you can find all of the code for this post at
[github.com/jtescher/example-ember-rails-dashboard](https://github.com/jtescher/example-ember-rails-dashboard).

Preparing the application
-------------------------

Before we add any more content let's update our Ember application template and add a basic structure with some
navigation in `app/assets/javascripts/templates/application.hbs`.

``` handlebars
<div class='container'>

  <header class='masthead'>
    <nav class='navbar navbar-default' role='navigation'>
      <div class='container-fluid'>
        <div class='navbar-header'>
          <button type='button' class='navbar-toggle'data-toggle='collapse' data-target='#main-nav-collapse'>
            <span class='sr-only'>Toggle navigation</span>
            <span class='icon-bar'></span>
            <span class='icon-bar'></span>
            <span class='icon-bar'></span>
          </button>
          {{#link-to 'application' class='navbar-brand'}}Dashboard{{/link-to}}
        </div>

        <div class='collapse navbar-collapse' id='main-nav-collapse'>
          <ul class='nav navbar-nav'>
            {{#link-to 'index' tagName='li' activeClass='active'}}
              {{#link-to 'index'}}Home{{/link-to}}
            {{/link-to}}
          </ul>
        </div>
      </div>
    </nav>
  </header>

  <section class='main-content'>
    {{outlet}}
  </section>

</div>

```

And let's add an index template so we know when we're on the home page in `app/assets/javascripts/templates/index.hbs`.

``` handlebars
<div class='jumbotron'>
  <h1>Hello from Ember.js!</h1>
  <p>Let's see some metrics.</p>
</div>

```

If you reload the page you should now see this:

![Ember application template](https://jtescher.github.io/assets/creating-a-metrics-dashboard-with-ember-and-rails-part-two/ember-home-screen.png)


Creating Dynamic Tables with Ember and Handlebars
-------------------------------------------------

Often the simplest way to display some metrics data is to just put it all in a table. I find that it is often helpful
when creating HTML with JavaScript to first start out with the final result and slowly add dynamic content in pieces.
Lets create an example of this to display some financial data about our sales process.

Step one is to create the Ember route to get to this URL. Let's edit `app/assets/javascripts/router.js.coffee` and add it:

``` coffeescript
# app/assets/javascripts/router.js.coffee
Dashboard.Router.map ()->
  @resource('orders')

```

And we can then create an idea of what we want the page to look like by adding `app/assets/javascripts/templates/orders.hbs`
