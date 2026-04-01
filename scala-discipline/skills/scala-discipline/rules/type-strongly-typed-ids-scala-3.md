# type-strongly-typed-ids-scala-3

> All entity identifiers should be strongly typed (Scala 3, `neotype`).

## Why It Matters

Using plain primitive types (e.g., `String`) for identifiers allows accidentally passing a `UserId` where a `RoleId` is expected — a bug the compiler cannot catch. Strongly typed IDs make such mixups a compile-time error, eliminating an entire class of subtle runtime defects. They also serve as documentation, making it immediately clear from the type what entity an ID refers to.

## Setup

```scala
libraryDependencies += "io.github.kitlangton" %% "neotype" % "0.4.10"
```

```scala
import neotype.*
```

## Examples

### Bad

```scala
// Primitives — compiler cannot distinguish a user ID from a role ID
case class User(id: String, name: String, active: Boolean)
case class Role(id: String, userId: String, name: String)

def findUser(id: String): User = ???
def findRole(id: String): Role = ???

val userId = "u-123"
val roleId = "r-456"
findUser(roleId)  // compiles — silent bug
```

### Good

```scala
import java.util.UUID

type UserId = UserId.Type
object UserId extends Newtype[String]:
  def random: UserId = UserId.unsafeMake(UUID.randomUUID().toString)

type RoleId = RoleId.Type
object RoleId extends Newtype[String]

case class User(id: UserId, name: String, active: Boolean)
case class Role(id: RoleId, userId: UserId, name: String)

def findUser(id: UserId): User = ???
def findRole(id: RoleId): Role = ???

val userId = UserId("u-123")
val roleId = RoleId("r-456")
findUser(roleId)  // compile error — type mismatch
findUser(userId)  // OK
```

### Using Long or UUID as the underlying type

```scala
type OrderId = OrderId.Type
object OrderId extends Newtype[Long]

type TraceId = TraceId.Type
object TraceId extends Newtype[java.util.UUID]

val oid  = OrderId(42L)
val tid  = TraceId(java.util.UUID.randomUUID())
```

### Construction and access

neotype's `apply` is a macro — it validates literals **at compile time**. For runtime values, use `unsafeMake` (assume valid) or `make` (returns `Either`):

```scala
// Compile-time literal — validated at compile time
val id1: UserId = UserId("literal-id")

// Runtime value known to be valid (e.g. from a trusted source)
val id2: UserId = UserId.unsafeMake(someRuntimeString)

// Runtime value from untrusted source — validate explicitly
val result: Either[String, UserId] = UserId.make(someRuntimeString)
```

Unwrap the underlying value explicitly with `.unwrap`:

```scala
val raw: String = id1.unwrap
```

### Optional validation constraints

Add an `inline def validate` to enforce invariants at compile time (for literals) and at runtime via `make`:

```scala
type CustomerId = CustomerId.Type
object CustomerId extends Newtype[String]:
  override inline def validate(value: String) =
    if value.nonEmpty then true
    else "CustomerId must not be empty"

val c = CustomerId("")    // compile error for literals
CustomerId.make("")       // Left("CustomerId must not be empty")
```

### Typeclass instances

Add codecs and other instances directly inside the companion object:

```scala
import io.circe.{Encoder, Decoder}

object UserId extends Newtype[String]:
  def random: UserId = UserId.unsafeMake(UUID.randomUUID().toString)
  given Encoder[UserId] = Encoder[String].contramap(_.unwrap)
  given Decoder[UserId] = Decoder[String].map(UserId.unsafeMake)
```
