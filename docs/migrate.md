Phantom 2.0.0 series
====================

### What got completed in Phantom 2.0.0

With the rapidly evolving requirements, Cassandra releases, and competition, it was only natural we kept Phantom up to scratch. In line with a lot of user feedback, the priorities of 2.0.0 were:

- Go back to the flexible licensing model everyone knows and loves(especially your legal department). No one wants to go through corporate litigation and licensing compliance to a `build.sbt` dependency, and if you've ever worked in a bank we all know it's not happening.

- Phantom was a really fun time saving introduction years ago when it was first introduced, but since then Scala has evolved to a point where many features of more esoteric components, such as the macro API, have reached a degree of stability that we can now exploit to our great advantage: **boilerplate elimitation**.

- From type parameters to keys, table class cake patterns, having to define `fromRow`, and a whole lot of other boilerplatey items, we have eliminated them one by one, reducing the amount of code you need to type to make it all work. The future looks even brighter, as we plan on fully eliminating the mapping DSL very shortly in favour of even more lightweight techniques.

Feedback and contributions are welcome, and we are happy to prioritise any crucial features Phantom may currently be lacking.

#### Licensing and distribution

- [x] Revert all Outworkers projects and all their dependencies to the Apache V2 License. 
- [x] Publish `outworkers-util` and all sub modules to Maven Central.
- [x] Publish `outworkers-diesel` and all sub modules to Maven Central.
- [x] Remove all non standard resolvers from Phantom, all dependencies should build from JCenter and Maven Central by default with no custom resolvers required. 
- [x] Change all package names and resolvers to reflect our business name change from `Websudos` to `Outworkers`.
- [x] Create a `1.30.x` release that allows users to transition to a no custom resolver version of Phantom 1.0.x even before 2.0.0 is stable.

#### Macro API to replace runtime features

- [x] Replace the Scala reflection library with a macro that can figure out what the contents of a table are.
- [x] Generate the name of a table using macros.
- [x] Generate the primary key of a table using macros.
- [x] Enforce primary key restrictions on a table using a macro.
- [x] Generate the `fromRow` method of `CassandraTable` using a macro if the `case class` fields and `table` columns are matched.
- [x] Enforce a same ordering restriction for case class fields and table columns to avoid generating invalid methods with the macro.
- [ ] Generate the `fromRow` if the fields match, they are in arbitrary order, but there are no duplicate types.
- [x] Allow arbitrary inheritance and usage patterns for Cassandra tables, and resolve inheritance resolutions with macros to correctly identify desired table structures.

#### Tech debt

- [x] Correctly implement Cassandra pagination using iterators, currently setting a `fetchSize` on a query does not correctly propagate or consume the resulting iterator, which leads to API inconsistencies and `PagingState` not being set on any `ResultSet`.
- [ ] Add a build matrix that will test phantom against multiple versions of Cassandra in Travis for Scala 2.11, with support for all major releases of Cassandra.
- [ ] Bump code coverage up to 100%

#### Features

- [ ] Native support for multi-tenanted environments via cached sessions.
- [ ] Case sensitive CQL.
- [ ] Materialized views.
- [ ] SASI index support
- [ ] Support for `PER PARTITION LIMIT` in `SelectQuery`.
- [ ] Support for `GROUP BY` in `SelectQuery`.

#### Scala 2.12 support

- [x] Add support for Scala 2.12 in the `util` library, remove all dependencies that don't comply.
- [x] Add support for Scala 2.12 in the `diesel-engine`.
- [x] Add support for Scala 2.12 in `phantom-dsl`
- [x] Add support for Scala 2.12 in `phantom-connectors`
- [ ] Add support for Scala 2.12 in `phantom-example`
- [ ] Add support for Scala 2.12 in `phantom-streams`
- [ ] Add support for Scala 2.12 in `phantom-thrift`
- [ ] Add support for Scala 2.12 in `phantom-finagle`

#### Documentation

- [x] Migration guide for transitioning to Phantom 2.0.0. [Guide here](https://github.com/outworkers/phantom/tree/feature/2.0.0#200-migration-guide). 
- [x] Move documentation back to the docs folder.
- [ ] Add a documentation website on the main page.
- [ ] Create a navigator that allows viewing the documentation at a particular point in time.


2.0.0 Migration guide
=====================

As a word of introduction, this guide is brand new and there may be certain elements we have currently left out. Phantom has an immense adopter base which includes many of you using the library in ways which we do not know of. 2.0.0 completely replaces fundemental aspects of the framework to provide superior performance and reliability, and we have tested back and forth to ensure the smoothest possible transition, but please feel free to report any issues via GitHub and we will fix them straight away.

- The OSS version of phantom has as of 2.0.0 returned to the Apache V2 license and the license is here to stay.
- All packages and dependencies are now available under the `com.outworkers` organisation instead of `com.websudos`. As
part of long term re-branding efforts, we have finally felt it's time to make sure the change is consistent throughout.
- There is a new and now completely optional Bintray resolver, `Resolver.bintrayRepo("outworkers", "oss-releases")`,
 that gives you free access to the latest cuts of our open source releases before they hit Maven Central. We assume
 no liability for your usage of latest cuts, but we welcome feedback and we do our best to have elaborate CI processes in place.
- Manually defining a `fromRow` inside a `CassandraTable` is no longer required if your column types match your case class types.
- `EnumColumn` is now relying entirely on `Primitive.macroImpl`, which means you will not need to pass in the enumeration
as an argument to `EnumColumn` anymore. This means `object enum extends EnumColumn(this, enum: MyEnum)` is now simply
`object enum extends EnumColumn[MyEnum#Value](this)`
- All dependencies are now being published to Maven Central. This includes outworkers util and outworkers diesel,
projects which have in their own right been completely open sourced under Apache V2 and made public on GitHub.
- All dependencies on `scala-reflect` have been completely removed.
- A new, macro based mechanism now performs the same auto-discovery task that reflection used to, thanks to `macro-compat`.
- Index modifiers no longer require a type parameter, `PartitionKey`, `PrimaryKey`, `ClusteringOrder` and `Index` don't require
the column type passed anymore.
- `KeySpaceDef` has been renamed to the more appropiate `
CassandraConnector`.
- `CassandraConnector` now natively supports specifying a keyspace creation query.
- `TimeWindowCompactionStrategy` is now natively supported in the CREATE/ALTER dsl.
- Collections can now be used as part of a primary or partition key.
- Tuples are now natively supported as valid types via `TupleColumn`.
- `phantom-reactivestreams` is now simply called `phantom-streams`.
- `Database.autocreate` and `Database.autotruncate` are now no longer accessible. Use `create`, `createAsync`, `truncate` and `truncateAsync` instead.
- `Database` now requires an f-bounded type argument: `class MyDb(override val connector: CassandraConnection) extends Database[MyDb](connector)`.
- Automated Cassandra pagination via paging states has been moved to a new method called `paginateRecord`. Using `fetchRecord` with a `PagingState` is no longer possible.
This is done to distinguish the underlying consumer mechanism of parsing and fetching records from Cassandra.
- `com.outworkers.phantom.dsl.context` should be used instead of `scala.concurrent.ExecutionContext.Implicits.global`.
