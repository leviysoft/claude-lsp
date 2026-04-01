# patterns-chimney

> Never share case classes across application layers; use chimney for boilerplate-free conversions between them.

## Why It Matters

Reusing the same model (e.g., a database row) across the API, domain, and persistence layers couples them together. A change to the DB schema leaks into the API contract; a new API field pollutes the domain model. Separate models per layer make each layer independently evolvable and testable. Chimney eliminates the resulting conversion boilerplate through macro-derived transformations that are checked at compile time — missing fields or type mismatches are errors, not runtime surprises.

## Bad

```scala
// Single "User" entity shared everywhere
case class User(id: Long, name: String, passwordHash: String, createdAt: Instant)

// DB layer returns it directly
def findUser(id: Long): Future[Option[User]] = ???

// API layer serialises it directly — passwordHash leaks to the client!
def getUser(id: Long): Future[Option[User]] = findUser(id)
```

## Good

```scala
// persistence layer
case class UserRow(id: Long, name: String, passwordHash: String, createdAt: Instant)

// domain model
case class User(id: Long, name: String, createdAt: Instant)

// API DTO
case class UserDto(id: Long, name: String)

import io.scalaland.chimney.dsl._

// DB → domain (drop passwordHash via explicit mapping)
def toDomain(row: UserRow): User =
  row.into[User].transform  // compile error if fields don't align

// domain → API DTO (automatic: field names match)
def toDto(user: User): UserDto =
  user.transformInto[UserDto]
```

## Setup

```scala
// build.sbt

// Scala 2 & Scala 3
libraryDependencies += "io.scalaland" %% "chimney" % "1.5.0"
```

## Usage

### Automatic transformation (field names and types match)

```scala
import io.scalaland.chimney.dsl._

val dto: UserDto = user.transformInto[UserDto]
```

### Customised transformation (rename, compute, or supply missing fields)

```scala
val domain: User = row
  .into[User]
  .withFieldComputed(_.displayName, r => s"${r.firstName} ${r.lastName}")
  .withFieldRenamed(_.ts, _.createdAt)
  .transform
```

### Reusable transformer (define once, use implicitly)

```scala
import io.scalaland.chimney.Transformer

implicit val rowToUser: Transformer[UserRow, User] =
  Transformer.define[UserRow, User].buildTransformer

// now transformInto resolves the implicit automatically
val user: User = row.transformInto[User]
```

### Partial transformer (for fallible conversions)

```scala
import io.scalaland.chimney.PartialTransformer

implicit val stringToStatus: PartialTransformer[String, Status] =
  PartialTransformer(s => partial.Result.fromOption(Status.withNameOption(s)))

val result: partial.Result[Status] = "active".transformIntoPartial[Status]
```
