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
my convention recommendations are inspired by the [Rails](http://rubyonrails.org) and [Ember](http://emberjs.com)
conventions.

### Routing Conventions

Standardized endpoints allow client applications to use simple open source REST clients like
[RestKit](https://github.com/RestKit/RestKit) or [Ember
Data](http://emberjs.com/api/data/classes/DS.RESTSerializer.html) without having to implement custom adapters. Here are
a few conventions to keep things simple and clean.

+ Use URL based API versioning. E.g. `/v1/posts`.
+ Use 5 standard endpoints with appropriate HTTP verbs:
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
re-thinking your schema or API as this can be a warning sign that things might be overly complicated.

+ User the singularized resource name with `DAO` suffix. E.g. `PostDAO`.
+ Create one DAO per database table or remote resource.
+ Move repetitive create, read, update, delete functions into shared
[DAOConventions](https://github.com/jtescher/play-api/blob/master/app/com/example/api/daos/DAOConventions.scala).

## Database Migrations

In modern development and production environments database schema changes must be versioned and performed in an
automated way. Raw SQL hand written SQL statements should be avoided at all costs as well as anyone making database
changes by SSH'ing into the server and running `ALTER TABLE` statements manually. If the framework you are using is like
Play and does not include great tools for automating database migrations out of the box, I would recommend using
[Liquibase](http://www.liquibase.org) to perform migrations and a library like
[Play Liquibase](https://github.com/Ticketfly/play-liquibase) to automate running migrations in dev and test
environments.

+ Use environment variables for database connection configuration in
[application.conf](https://github.com/jtescher/play-api/blob/master/conf/application.conf).
+ Use `YYYMMDDHHMMSS_database_migration_description.xml` as changelog names.
+ Tag your database after making each significant change for easy rollback.

## Tests

Testing is an essential part of application development as it provides crucial feedback on your application architecture
and design during the dev process as well as providing confidence while refactoring. For Scala applications prefer
[ScalaTest](http://www.scalatest.org) over [Specs2](https://etorreborre.github.io/specs2) and include integration tests
and unit tests in the same package as the source files.

+ Use `Spec` as the suffix for unit tests and `IntegrationSpec` as the suffix for integration tests.
+ Reset the DB before each integration test with 
[DatabaseCleaner](https://github.com/jtescher/play-api/blob/master/test/utils/DatabaseCleaner.scala) to avoid
order-dependant tests.
+ Prefer a real database over an in-memory stand in for integration tests to find DB specific bugs.
+ Use a Factory library or create your own simple factories like
[PostFactory](https://github.com/jtescher/play-api/blob/master/test/factories/PostFactory.scala) to keep test setup DRY
and expressive.
+ Prefer high level integration tests for common paths and unit tests for edge cases and full coverage.

## Plugins

Certain concerns like test coverage and code conventions are very infrequently packaged with frameworks and must be
included to support modern development flows. For Play I recommend the following plugins:

+ Use [Scoverage](https://github.com/scoverage/sbt-scoverage) with `coverageMinimum := 100` and `coverageFailOnMinimum
  := true`
in [build.sbt](https://github.com/jtescher/play-api/blob/master/build.sbt).
+ Use [Scalastyle](http://www.scalastyle.org/sbt.html) with `level="error"` in
  [scalastyle-config.xml](https://github.com/jtescher/play-api/blob/master/scalastyle-config.xml).
+ Use [Scalariform](https://github.com/daniel-trinh/scalariform) for standard style enforcement.
+ Use a server monitoring tool like New Relic with [sbt-newrelic](https://github.com/gilt/sbt-newrelic).
+ Enforce UTC timezone in JVM with [sbt-utc](https://github.com/tim-group/sbt-utc).

## Additional Files

Applications on the JVM often have difficult initial setup procedures. In order to have the smoothest dev process you
should include a few extra files for convenience.

+ Include a [.java-version](https://github.com/jtescher/play-api/blob/master/.java-version) file with the expected Java
version.
+ Include a [activator](https://github.com/jtescher/play-api/blob/master/activator) wrapper file that downloads all
necessary dependencies. (serving the app and testing the app should be as simple as: `$ ./activator run` and
`$ ./activator test`).
+ Include a [resetdb](https://github.com/jtescher/play-api/blob/master/resetdb) script that drops and re-creates the
database for testing and new developers.

## TLDR

Sticking to the conventions of your framework can make development processes streamlined and avoid
[bikeshedding](https://en.wikipedia.org/wiki/Parkinson%27s_law_of_triviality). If your framework does not have
conventions then you should adopt conventions similar to the ones I laid out in this post to gain the same benefits.
These conventions apply to creating CRUD API's, but you should always find conventions and best practices for the
particular architectural style that your app, framework, and language uses.

