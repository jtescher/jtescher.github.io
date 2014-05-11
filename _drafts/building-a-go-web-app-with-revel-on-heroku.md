---
layout: post
title:  "Building A Go Web App with Revel on Heroku"
date:   2014-05-12 09:00:00
categories: golang revel heroku
---

I've been interested in Go for a long time now. The language has lots of aspects that make it well suited for building
modern web applications including its powerful [standard library](http://golang.org/pkg/), concurrency primitives, and
impressive [performance benchmarks](http://www.techempower.com/benchmarks). The community is also gaining lots of
ground in terms of tooling, web frameworks, and other resources for creating sites in go.

In this post I will show you how to create a basic app in go and how to host it on [Heroku](https://www.heroku.com/).

If you do not have go installed on your system, you can do so on a mac with `$ brew install go` or follow the
instructions [here](http://golang.org/doc/install). Also if you’re interested in learning the basics of go,
be sure to check out [A Tour of Go](http://tour.golang.org/).

If you get stuck you can find all of the code for this post at
[github.com/jtescher/example-revel-app](https://github.com/jtescher/example-revel-app).


Creating a Revel App
--------------------

![Revel Logo](https://jtescher.github.io/assets/building-a-go-web-app-with-revel-on-heroku/revel-logo.png)

