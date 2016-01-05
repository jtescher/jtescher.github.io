---
layout: post
title:  "Using conventions to keep your code simple"
date:   1970-01-01 00:00:00
categories: startups advice code conventions
---

One piece of advice I wish I had been given when I started my first company was how to structure conventions in my
codebases. It may not seem like a big deal when you're hacking out your first prototypes in your garage, but a small
amount of discipline can make the difference between having malleable code that allows you to keep up with the changing
demands of your industry and writing yourself into a place where you have to throw everything out and re-write every
time requirements change as you search for product / market fit.

In this post I'll show you an example of a basic conventional
[RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer)
[CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) API using
[Play Framework](https://www.playframework.com/) but I try to keep the conventions abstract so they could be used with
any framework and language. All code can be found at [jtescher/play-api](https://github.com/jtescher/play-api).

## The CRUD API server

The CRUD API is the bread and butter of every startup's products and if your problem can fit into the confines of this
simple architecture it will remain easy to make large changes quickly and with confidence. The availability of open
source JSON REST clients in basically all languages makes it extremely quick to develop and change applications. Many of
these conventions come from [Rails](http://rubyonrails.org) and [Ember](http://emberjs.com).

### Routing Conventions

Standardized endpoints allow client applications to use simple open source REST clients like
[RestKit](https://github.com/RestKit/RestKit) or [Ember
Data](http://emberjs.com/api/data/classes/DS.RESTSerializer.html) without having to implement custom adapters. Here are
a few conventions to keep things simple and clean.

+ Use URL based API versioning. E.g. `/v1/posts`.
+ Use 5 standard endpoints with appropriate HTTP verb:
+ Use shallow nesting `/v1/posts/1/comments` and `/v1/comments/2` instead of `/v1/posts/1/comments/2`.

```
GET       /v1/posts        # => Index
POST      /v1/posts        # => Create
GET       /v1/posts/:id    # => Show
PUT       /v1/posts/:id    # => Update
DELETE    /v1/posts/:id    # => Destroy
```

### Controller Contentions

Your controllers should be as simple as possible and free of business logic. They should primarily be for exposing your
application over HTTP and handling concerns like mapping application responses and errors to status codes. If you find a
lot of code building up in your controller layer, you might consider pulling it out into a library or finding an open
source solution that accomplishes the same task. If done correctly, all of your controllers should look very uniform and
very simple.

+ Use pluralized resource name with `Controller` suffix. E.g. `PostsController`.
+ Use the
[ControllerConventions](https://github.com/jtescher/play-api/blob/master/app/com/example/api/controllers/utils/ControllerConventions.scala)
 trait to abstract common controller code like model parsing.
+ Move repetitive error handling logic into
[ErrorHandler](https://github.com/jtescher/play-api/blob/master/app/com/example/api/ErrorHandler.scala).
+ Import your model's JSON writes function from your corresponding serializer.
+ Include your model's JSON reads function in your controller for versioning, custom validations, and API clarity:

```scala
implicit val postJsonReads = (
  (__ \ "id").read[UUID] and
  (__ \ "title").read[String] and
  ...
)(Post.apply _)
```

## Serializers

Serializers should be versioned and explicit as to which keys and values will be present. Try to avoid macros or other
ways of having your serializers written for you as it can make your payloads more difficult to reason about.

+ Create one serializer per model.
+ Avoid having domain logic in your serializers wherever possible.
+ User the singularized resource name with `Serializer` suffix. E.g. `PostSerializer`.

```scala
implicit val postJsonWrites = new Writes[Post] {
  def writes(post: Post): JsObject = Json.obj(
    "id" -> post.id,
    "title" -> post.title,
    ...
  )
}
```

## Services

Your service layer is where all business logic should exist. Take extra care to keep domain concepts and concerns
separated out and refactor frequently as your understanding of your problem space evolves. Services should be kept to
one concern and one level of abstraction as much as possible. If you find your service is doing both high level
orchestration and low level work, or mixing concerns from multiple domains, you should consider breaking the logic
into more appropriate services.

+ User the singularized resource name with `Service` suffix. E.g. `PostService`.
+ Use dependency injection to keep services decoupled.
+ Keep services small and preferably < 100 lines of code.

## Data Access Objects

Data access concerns including database specific logic or remote API call specifics should be encapsulated here and
hidden from the service layer as much as possible. If your data access objects do not look uniform and simple consider
re-thinking your schema or API as this can be a warning sign that things could be overly complicated.

+ User the singularized resource name with `DAO` suffix. E.g. `PostDAO`.
+ Create one DAO per database table or remote resource.
+ Move repetitive create, read, update, delete functions into shared
[DAOConventions](app/com/example/api/daos/DAOConventions.scala).

## Database Migrations

+ Use environment variables for database connection configuration in [application.conf](conf/application.conf).
+ Use [Liquibase](http://www.liquibase.org) to perform migrations and [Play
  Liquibase](https://github.com/Ticketfly/play-liquibase)
to have migrations run in dev / test mode.
+ Use `YYYMMDDHHMMSS_database_migration_description.xml` as changelog names.
+ Tag your database after making each significant change for easy rollback.

## Plugins

+ Use [Scoverage](https://github.com/scoverage/sbt-scoverage) with `coverageMinimum := 100` and `coverageFailOnMinimum
  := true`
in [build.sbt](build.sbt).
+ Use [Scalastyle](http://www.scalastyle.org/sbt.html) with `level="error"` in
  [scalastyle-config.xml](scalastyle-config.xml).
+ Use [Scalariform](https://github.com/daniel-trinh/scalariform) for standard style enforcement.
+ Use a server monitoring tool like New Relic with [sbt-newrelic](https://github.com/gilt/sbt-newrelic).
+ Enforce UTC timezone in JVM with [sbt-utc](https://github.com/tim-group/sbt-utc).

## Additional Files

+ Include a [.java-version](.java-version) file with the expected Java version.
+ Include a [activator](activator) wrapper file that downloads all necessary dependencies
(serving the app and testing the app should be as simple as: `$ ./activator run` and `$ ./activator test`).
+ Include a [resetdb](resetdb) script that drops and re-creates the database for testing and new developers.

## Tests

+ Prefer [ScalaTest](http://www.scalatest.org) over [Specs2](https://etorreborre.github.io/specs2).
+ Include integration tests and unit tests in the same package as the source files.
+ Use `Spec` as the suffix for unit tests and `IntegrationSpec` as the suffix for integration tests.
+ Reset the DB before each integration test with [DatabaseCleaner](test/utils/DatabaseCleaner.scala) to avoid
  order-dependant tests.
+ Prefer a real database over an in-memory stand in for integration tests to find DB specific bugs.
+ Use factories like the [PostFactory](test/factories/PostFactory.scala) to keep test setup DRY and expressive.
+ Prefer high level integration tests for common paths and unit tests for edge cases and full coverage.

