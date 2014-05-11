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

To get the project started, first install Revel into your $GOPATH:

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
