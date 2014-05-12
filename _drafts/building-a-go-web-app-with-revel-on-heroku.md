---
layout: post
title:  "Building A Go Web App with Revel on Heroku"
date:   2014-05-12 09:00:00
categories: go golang revel heroku
---

I've been interested in Go for a long time now. The language has lots of aspects that make it well suited for building
modern web applications including its powerful [standard library](http://golang.org/pkg/), concurrency primitives, and
impressive [performance benchmarks](http://www.techempower.com/benchmarks). The community is also gaining lots of
ground in terms of tooling, web frameworks, and other resources for creating sites in Go.

In this post I will show you how to create a basic app in Go and how to host it on [Heroku](https://www.heroku.com/).

If you do not have Go installed on your system, you can do so on a mac with `$ brew install go` or follow the
instructions [here](http://golang.org/doc/install). Also if you’re interested in learning the basics of Go,
be sure to check out [A Tour of Go](http://tour.golang.org/).

If you get stuck you can find all of the code for this post at
[github.com/jtescher/example-revel-app](https://github.com/jtescher/example-revel-app).


Creating a Revel App
--------------------

[![Revel Logo](https://jtescher.github.io/assets/building-a-go-web-app-with-revel-on-heroku/revel-logo.png)](http://revel.github.io/)

Revel is a high-productivity web framework for the Go language. It has an impressive feature set and is much more in
line with the rails "convention over configuration" approach than most other go web frameworks.

To get the project started, first install Revel into your `$GOPATH`:

```bash
$ go get github.com/revel/cmd/revel
```

Then we can create a new app with the `revel new` command. Remember you should use your own github username below:

```bash
$ revel new github.com/jtescher/blog
```

And we can start up the app using the `revel run` command. Again use your github usename, not mine.

```bash
$ revel run github.com/jtescher/blog
~
~ revel! http://robfig.github.com/revel
~
INFO  2014/05/11 16:50:10 revel.go:292: Loaded module static
INFO  2014/05/11 16:50:10 revel.go:292: Loaded module testrunner
INFO  2014/05/11 16:50:10 run.go:57: Running blog (github.com/jtescher/blog) in dev mode
INFO  2014/05/11 16:50:10 harness.go:157: Listening on :9000
INFO  2014/05/11 16:50:23 revel.go:292: Loaded module testrunner
INFO  2014/05/11 16:50:23 revel.go:292: Loaded module static
INFO  2014/05/11 16:50:23 main.go:29: Running revel server
Go to /@tests to run the tests.
```

Now you can go to [localhost:9000](http://localhost:9000/) and you should see the welcome page.

![Revel Welcome Page](https://jtescher.github.io/assets/building-a-go-web-app-with-revel-on-heroku/revel-welcome.png)


Serving Content From Revel
--------------------------

Revel has an easily understood MVC structure and ships with bootstrap by default. You can read more about the flow of
control through a Revel app in their excellent [core concepts](http://revel.github.io/manual/concepts.html)
documentation, but for now let's just render some custom content by adjusting the `App/index.html` view. Revel uses Go's
built in templating for HTML rendering.

```html
<!-- app/views/App/index.html -->
{{set . "title" "Home"}}
{{template "header.html" .}}

<div class="container">
  <div class="hero-unit">
    <h1>Hello from Revel!</h1>
    <p>Creating HTML pages is very straightforward.</p>
    <p><a class="btn btn-primary btn-large" role="button">Awesome</a></p>
  </div>

  <div class="row">
    <div class="span6">
      {{template "flash.html" .}}
    </div>
  </div>
</div>

{{template "footer.html" .}}
```

And if you reload the page you should see your changes.

![Custom Revel Content](https://jtescher.github.io/assets/building-a-go-web-app-with-revel-on-heroku/custom-revel-content.png)


Deploying The App
-----------------

Deploying a Revel app to Heroku is quite simple. First we create a `.godir` file at the root of our project with the
path to the project in it.

```bash
$ echo "github.com/jtescher/blog" > .godir
```

Then we can create the app on Heroku using a buildpack.

```bash
$ heroku create -b https://github.com/robfig/heroku-buildpack-go-revel.git
```

And finally we can push our app to Heroku.

```bash
$ git push heroku
$ heroku open
```

And that's it! Your Revel app should now be live.
