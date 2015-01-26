---
layout: post
title:  "Create Flexible System Boundaries With Akka and Camel Part One"
date:   2015-01-26 09:00:00
categories: akka camel play scala rabbitmq message-bus
---

![Akka Logo](https://jtescher.github.io/assets/create-flexible-system-boundaries-with-akka-and-camel-part-one/akka-logo.png)

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

To demonstrate how this works I will set up a Play producer app and a simple akka consumer app (all source code is 
available for the [producer](https://github.com/jtescher/akka-camel-producer-example) and the 
[consumer](https://github.com/jtescher/akka-camel-consumer-example). These apps will then communicate via 
[RabbitMQ pub/sub](https://www.rabbitmq.com/tutorials/tutorial-three-python.html), but instead of using the RabbitMQ 
libraries directly we will use Akka Camel producer and consumer actors.


Akka Camel
==========

Apache Camel has many capabilities including a really powerful routing DSL, but in this post we will focus on its 
integration with Akka via the [Camel module](http://doc.akka.io/docs/akka/snapshot/scala/camel.html).

The full power of Camel is a little difficult to explain but here is how the Akka documentation introduces it:

> The akka-camel module allows Untyped Actors to receive and send messages over a great variety of protocols and APIs.
> In addition to the native Scala and Java actor API, actors can now exchange messages with other systems over large
> number of protocols and APIs such as HTTP, SOAP, TCP, FTP, SMTP or JMS, to mention a few. At the moment, approximately
> 80 protocols and APIs are supported.


Creating The Producer App
=========================

Integration with any Akka app is quite simple. To get started first install RabbitMQ via Homebrew `$ brew install 
rabbitmq` and then start the server with `$ rabbitmq-server` then to create the Play app, install activator via 
`$ typesafe-activator`. Now you can start creating your producer app:

```bash
$ activator new akka-camel-producer-example
Browse the list of templates: http://typesafe.com/activator/templates
Choose from these featured templates or enter a template name:
  1) minimal-akka-java-seed
  2) minimal-akka-scala-seed
  3) minimal-java
  4) minimal-scala
  5) play-java
  6) play-scala
(hit tab to see a list of all templates)
> 6
OK, application "akka-camel-producer-example" is being created using the "play-scala" template.
```

We will be creating a new user and sending that user to the akka app for further processing, so the first step is to 
create a basic integration test and then add a play route in `conf/routes` and a `Users` controller so we can test our 
Play scaffold:

Our user data for now will simply have a `firstName` and `lastName` so our first test will look something like this:

```scala
import org.junit.runner.RunWith
import org.specs2.mutable.Specification
import org.specs2.runner.JUnitRunner
import play.api.libs.json.Json
import play.api.test.Helpers._
import play.api.test.{ FakeApplication, FakeRequest, WithApplication }

@RunWith(classOf[JUnitRunner])
class UsersSpec extends Specification {
  "Users" should {
    "accept new users" in new WithApplication(FakeApplication()) {
      val user = Json.obj("firstName" -> "John", "lastName" -> "Doe")
      val request = FakeRequest(POST, "/users").withJsonBody(user)
      val create = call(controllers.Users.create, request)

      status(create) must equalTo(CREATED)
      contentType(create) must beSome.which(_ == "application/json")
      contentAsJson(create) must beEqualTo(user)
    }
  }
}
```

Now that we have a failing test we can add our controller and route:

```scala
POST    /users    controllers.Users.create
```

And then create the Users controller in `app/controllers/Users.scala`:

```scala
package controllers

import play.api.mvc.{ Action, BodyParsers, Controller }

object Users extends Controller {

  def create = Action(BodyParsers.parse.json) { request =>
    Created(request.body)
  }
}
```

`$ activator test` should now show all tests as being green.


Adding The Camel Producer Actor
==============================

To add the producer we need to add `akka-camel` and whichever transport mechanisms you want to support to your
`build.sbt` file. In our case we will use `camel-rabbitmq`.

```scala
libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-camel" % "2.3.8",
  "org.apache.camel" % "camel-rabbitmq" % "2.14.1",
  cache,
  ws
)
```

Then we should add our message bus configuration to `conf/application.conf`:

```scala
# Message Bus
# ~~~~~
# Message bus configuration
message-bus.protocol="rabbitmq"
message-bus.protocol=${?MESSAGE_BUS_PROTOCOL}
message-bus.username="guest"
message-bus.username=${?MESSAGE_BUS_USERNAME}
message-bus.password="guest"
message-bus.password=${?MESSAGE_BUS_PASSWORD}
message-bus.hostname="localhost"
message-bus.hostname=${?MESSAGE_BUS_HOSTNAME}
message-bus.port=5672
message-bus.port=${?MESSAGE_BUS_PORT}
message-bus.path="/camel-example?exchangeType=fanout&autoDelete=true"
message-bus.url=${message-bus.protocol}"://"${message-bus.username}":"${message-bus.password}"@"${message-bus.hostname}":"${message-bus.port}${message-bus.path}
```

Then we can add our producer actor in `app/actors/EventStreamProducerActor.scala`:

```scala
package actors

import akka.actor.{ Actor, Props }
import akka.camel.{ Oneway, Producer }
import play.api.Play

class EventStreamProducerActor extends Actor with Producer with Oneway {
  private val messageBusURL = Play.current.configuration.getString("message-bus.url").get
  def endpointUri = messageBusURL
}

object EventStreamProducerActor {
  def props: Props = Props[EventStreamProducerActor]
}
```

And when we send messages to this actor, they are automatically converted to Camel messages and sent to RabbitMQ! Let's 
see how that might look in the `Users` controller:

```scala
package controllers

import actors.EventStreamProducerActor
import akka.actor.ActorRef
import play.api.libs.concurrent.Akka
import play.api.Play.current
import play.api.mvc.{ Action, BodyParsers, Controller }

object Users extends Controller {

  val eventStreamProducer: ActorRef = Akka.system.actorOf(EventStreamProducerActor.props)

  def create = Action(BodyParsers.parse.json) { request =>
    eventStreamProducer ! request.body.toString.getBytes
    Created(request.body)
  }
}
```

And with that we have a functional camel producer app! 

To see this app in action you can use `$ activator run` and post a user to the server:

```bash
$ curl -X POST -d '{"firstName": "John", "lastName": "Doe"}' http://localhost:9000/users -H 'Content-Type: application/json'
#=> {"firstName":"John","lastName":"Doe"}
```

And if you open your RabbitMQ web UI (by default at [localhost:15672](http://localhost:15672)) then you should be able to see the 
messages being sent to the broker.


