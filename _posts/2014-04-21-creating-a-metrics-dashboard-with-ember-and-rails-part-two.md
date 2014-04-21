---
layout: post
title:  "Creating A Metrics Dashboard With Ember.js, Bootstrap, and Rails - Part 2"
date:   2014-04-21 09:00:00
categories: rails ember.js metrics
---

*This is part two of a series on building a metrics dashboard with Ember, Bootstrap, and Rails. Over the next few weeks
I will be building out more functionality and writing posts to cover that. If you haven't read
[part one](/creating-a-metrics-dashboard-with-ember-and-rails-part-one) then that's a good place to start.* 

*[View code from this post in Github](github.com/jtescher/example-ember-rails-dashboard)*

In [part one](/creating-a-metrics-dashboard-with-ember-and-rails-part-one) we ended up with a Rails app that generated
the Ember app that rendered our metrics page. If you followed along your page should now look like this:

![Part One Final](https://jtescher.github.io/assets/creating-a-metrics-dashboard-with-ember-and-rails-part-one/ember-home-screen.png)

Today we're going to add some tables and see how data flows through an Ember app. If you are not familiar with Ember's
conventions and idioms you should first read their
[excellent documentation and guides](http://emberjs.com/guides/concepts/core-concepts/) before continuing here.

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


Creating Tables with Ember and Handlebars
-----------------------------------------

Often the simplest way to display metrics data is to just put it all in a table. I find that it is often helpful
when creating HTML with JavaScript to first start out with the final result and slowly add dynamic content in pieces.
Let's create an example of this to display some financial data about our sales process.

Step one is to create the Ember route to get to this URL. Let's edit `app/assets/javascripts/router.js.coffee` and add it:

``` coffeescript
# app/assets/javascripts/router.js.coffee
Dashboard.Router.map ()->
  @resource('orders')

```

And we can then create an idea of what we want the page to look like by mocking up some static HTML in `app/assets/javascripts/templates/orders.hbs`

``` html
<h1>Orders</h1>
<table class='table table-striped'>
  <tr>
    <th>#</th>
    <th>First Name</th>
    <th>Last Name</th>
    <th>Quantity</th>
    <th>Revenue</th>
  </tr>
  <tr>
    <td>1</td>
    <td>James</td>
    <td>Deen</td>
    <td>1</td>
    <td>$10.00</td>
  </tr>
  <tr>
    <td>2</td>
    <td>Alex</td>
    <td>Baldwin</td>
    <td>2</td>
    <td>$20.00</td>
  </tr>
</table>

<p>Total Quantity: <b>3</b></p>
<p>Total Revenue: <b>$30</b></p>

```

Then we can add a link to this page in our `application.hbs` file (the double link-to is an annoying hack but
explained [here](https://github.com/emberjs/ember.js/issues/4387)):
``` handlebars
...
<div class='collapse navbar-collapse' id='main-nav-collapse'>
  <ul class='nav navbar-nav'>
    {{#link-to 'index' tagName='li' activeClass='active'}}
      {{#link-to 'index'}}Home{{/link-to}}
    {{/link-to}}
    {{#link-to 'orders' tagName='li' activeClass='active'}}
      {{#link-to 'orders'}}Orders{{/link-to}}
    {{/link-to}}
  </ul>
</div>
...

```

Then if you reload and follow the orders nav link you should see this:

![Static Orders](https://jtescher.github.io/assets/creating-a-metrics-dashboard-with-ember-and-rails-part-two/orders-screen.png)

Creating Dynamic Tables
-----------------------

In Ember, a template typically retrieves information to display from a controller, and the controller is set up by its
route. Let's make this table dynamic by assigning the data required to build it in the `OrdersRoute`.

``` coffeescript
# app/assets/javascripts/routes/orders_route.js.coffee
Dashboard.OrdersRoute = Ember.Route.extend({
  model: ->
    [
      {
        id: 1,
        firstName: 'James',
        lastName: 'Deen',
        quantity: 1,
        revenue: '10.00',
      },
      {
        id: 2,
        firstName: 'Alex',
        lastName: 'Baldwin',
        quantity: 2,
        revenue: '20.00',
      }
    ]
})

```

And then we set up the controller to process the totals in the `OrdersController`.

``` coffeescript
# app/assets/javascripts/controllers/orders_controller.js.coffee
Dashboard.OrdersController = Ember.ArrayController.extend({

  totalQuantity: (->
    @get('content').reduce(((previousValue, order) ->
      previousValue + order.quantity
    ), 0)
  ).property('@each')

  totalRevenue: (->
    @get('content').reduce(((previousValue, order) ->
      previousValue + parseFloat(order.revenue)
    ), 0)
  ).property('@each')

})

```

And this lets us use the data provided by the controller in the template:

``` handlebars
<h1>Orders</h1>
<table class='table table-striped'>
  <tr>
    <th>#</th>
    <th>First Name</th>
    <th>Last Name</th>
    <th>Quantity</th>
    <th>Revenue</th>
  </tr>
  {{#each}}
  <tr>
    <td>{{id}}</td>
    <td>{{firstName}}</td>
    <td>{{lastName}}</td>
    <td>{{quantity}}</td>
    <td>{{revenue}}</td>
  </tr>
  {{/each}}
</table>

<p>Total Quantity: <b>{{totalQuantity}}</b></p>
<p>Total Revenue: <b>{{totalRevenue}}</b></p>

```

If you reload the page it should now look like this:

![Dynamic Orders](https://jtescher.github.io/assets/creating-a-metrics-dashboard-with-ember-and-rails-part-two/dynamic-orders-screen.png)

Formatting Values With Helpers
------------------------------

The image above looks pretty good but we seem to have lost our currency formatting. There are a few good libraries for
formatting currencies in JavaScript, and Ember makes it simple to access these libraries in your handlebars templates.
Let's add the [accounting.js](http://josscrowcroft.github.io/accounting.js/) library to format our revenue numbers.

First install the library to `vendor/assets/javascripts`:

``` bash
$ curl https://raw.github.com/josscrowcroft/accounting.js/master/accounting.js -o vendor/assets/javascripts/accounting-0.3.2.js
```

Then include it in your `application.js` file:

``` javascript
//= require jquery
//= require accounting-0.3.2
//= require_tree .

```

Then create a `currency_helpers.js.coffee` file to wrap the library.

``` coffeescript
# app/assets/javascripts/helpers/currency_helpers.js.coffee
Ember.Handlebars.registerBoundHelper('numberToCurrency', (number) ->
  accounting.formatMoney(number)
)

```

And finally we can use this helper in our `orders.hbs` view:

``` handlebars
...
{{#each}}
  <tr>
    <td>{{id}}</td>
    <td>{{firstName}}</td>
    <td>{{lastName}}</td>
    <td>{{quantity}}</td>
    <td>{{numberToCurrency revenue}}</td>
  </tr>
  {{/each}}
</table>

<p>Total Quantity: <b>{{totalQuantity}}</b></p>
<p>Total Revenue: <b>{{numberToCurrency totalRevenue}}</b></p>
```

Now that our numbers are formatted properly you can reload the page and see our final result for part 2:

![Currency Helpers](https://jtescher.github.io/assets/creating-a-metrics-dashboard-with-ember-and-rails-part-two/currency-helpers.png)
