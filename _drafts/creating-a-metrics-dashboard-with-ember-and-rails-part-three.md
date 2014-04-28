---
layout: post
title:  "Creating A Metrics Dashboard With Ember.js, Bootstrap, and Rails - Part 3"
date: 2014-08-25 09:00:00
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


Choosing The Right Library
--------------------------

[![Highcharts Demo](/assets/creating-a-metrics-dashboard-with-ember-and-rails-part-three/highcharts-demo.png)](http://www.highcharts.com/demo/combo/)

There are many good options when it comes to JavaScript graphing, charting, and visualizations. I find
[highcharts](http://www.highcharts.com/) to be a good place to get started and they are free for non-commercial uses!
If you find yourself needing more control or having a very specific requirement you can always look at projects like
[d3.js](http://d3js.org/).


Adding Highcharts
-----------------

Let's download the latest version of highcharts to our `vendor/assets/javascripts` directory.

```bash
$ curl http://code.highcharts.com/4.0.1/highcharts.js \
    -o vendor/assets/javascripts/highcharts-4.0.1.js
```

And then require the file in `app/assets/javascripts/application.js`

```js
...
//= require jquery
//= require accounting-0.3.2
//= require highcharts-4.0.1
//= require_tree .

```


Creating The Ember Component
----------------------------

Ember makes adding reusable components quite simple. We can add a component that represents a specific chart we want to
show on the screen and have ember re-render the chart whenever the data changes. You can read more about how the Ember
components work [here](http://emberjs.com/guides/components/).

As an example we can add a highcharts column chart to show revenue by product. First let's add the component in our
`app/assets/javascripts/templates/orders.hbs` file:

```html
<h1>Orders</h1>

{{column-chart chartId='revenue-by-product'}}

<table class='table table-striped'>
...

```

Then we can add the template for the component in `app/assets/javascripts/templates/components/column-chart.hbs`:

```html
<div {{bind-attr id='chartId'}} style='width: 100%;'></div>

```

And finally we can define the component in `app/assets/javascripts/components/column-chart.js.coffee`:

```coffeescript
Dashboard.ColumnChartComponent = Ember.Component.extend
  tagName: 'div'
  classNames: ['highcharts']

  contentChanged: (->
    @rerender()
  ).observes('series')

  didInsertElement: ->
    $("##{@chartId}").highcharts({
      chart: { type: 'column' },
      title: { text: 'Revenue by Product' },
      legend: { enabled: false },
      xAxis: {
        title: {
          text: 'Product Number'
        }
      },
      series: [{
        name: 'Quantity',
        data: [1, 2]
      }, {
        name: 'Revenue',
        data: [10.0, 20.0]
      }]
    })

  willDestroyElement: ->
    $("##{@chartId}").remove()

```

Then when you reload the page it should look like this:

![Orders Static Column Chart](/assets/creating-a-metrics-dashboard-with-ember-and-rails-part-three/orders-static-column-chart.png)
