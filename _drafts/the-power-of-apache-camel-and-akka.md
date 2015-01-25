---
layout: post
title:  "The Power of Apache Camel And Akka"
date:   2015-01-26 09:00:00
categories: akka camel message-bus
---

![Akka Logo](https://jtescher.github.io/assets/the-power-of-apache-camel-and-akka/akka-logo.png)

[Akka](http://akka.io/) takes a lot of pain out of trying to create highly concurrent systems. Akka's use of the 
[actor model](https://en.wikipedia.org/wiki/Actor_model) provides you with lots of great building blocks with which to 
develop your applications. Also as of Akka 2.1 there is now support for 
[clustering](http://doc.akka.io/docs/akka/snapshot/scala/cluster-usage.html) your actor system across several machines 
and building distributed applications without a single point of failure.

Creating communication channels *between* systems can still be a challenge. When you have multiple independent actor 
systems that need to pass messages to each other (or to non-actor based applications) there is no official mechanism 
that you can simply drop in place. In this post I will show you one way of creating an abstraction over system 
boundaries that allows you to define your messages and behavior once, and swap out the underlying transport mechanism 
(JSON REST, [RabbitMQ](https://www.rabbitmq.com), [Kafka](https://kafka.apache.org), etc) whenever you want to with
[Apache Camel](https://camel.apache.org).

To demonstrate how this works I will set up two simple akka apps, one as a producer and the other as a consumer (all 
source code is available for the [producer](https://github.com/jtescher/akka-camel-producer-example) and the 
[consumer](https://github.com/jtescher/akka-camel-consumer-example).

