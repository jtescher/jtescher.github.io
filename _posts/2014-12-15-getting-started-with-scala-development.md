---
layout: post
title:  "Getting Started With Scala Development"
date:   2014-12-15 09:00:00
categories: Scala languages
---

![Scala Logo](https://jtescher.github.io/assets/getting-started-with-scala-development/scala-logo.png)

Learning new languages can be a great way to expand your skills and stay up to date with software development 
trends. The language that I'm currently learning is Scala! It's a great language for picking up functional programming 
concepts, and it being on the JVM allows you to leverage the Java ecosystem's libraries and frameworks. In this post I'll 
show you how to install Scala and give you a few resources to get you started.

Installing Scala
================

To install Scala you need to install a [Java runtime](https://www.java.com/en/) version 1.6 or later. Once you have 
that installed you can install Scala through [homebrew](http://brew.sh) if you are on a mac, or follow [these 
instructions](http://scala-lang.org/download) if you are not.

```bash
$ brew install scala
```

Using Scala
===========

You can get started playing with the language basics from the terminal with the `scala` command.

```bash
$ scala
Welcome to Scala version 2.11.4 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_25).
Type in expressions to have them evaluated.
Type :help for more information.

scala> 
```

Here you can evaluate simple expressions and quickly try a few things (enter :quit to exit the REPL).

```bash
scala> println("Hello world")
Hello world

scala> 
```

Learning Scala
==============

Scala has great free resources to get you up to speed quickly. I would recommend the following:

* The online course [Functional Programming Principles in Scala](https://www.coursera.org/course/progfun) available on 
Coursera. This class is taught by the creator of Scala, Martin Odersky, and is a good overview of scala functional 
programming styles.

* The [Principles of Reactive Programming](https://www.coursera.org/course/reactive) course, also taught by 
Martin and goes into a lot of ways to create composable software that is event-driven, scalable under load, resilient 
and responsive in the presence of failures

* The books [Programming in Scala](http://www.artima.com/shop/programming_in_scala_2ed) and [Scala in Action]
(http://www.manning.com/raychaudhuri/) which cover the language features in depth.

* Also [Kojo](http://www.kogics.net/sf:kojo) which is an interesting interactive learning environment.

