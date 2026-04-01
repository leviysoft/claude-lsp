# type-strongly-typed-ids-scala-3

> All entity identifiers should be strongly typed (Scala 3).

## Why It Matters

Using plain primitive types (e.g., `String`) for identifiers allows accidentally passing a `SID[User]` where a `SID[Role]` is expected — a bug the compiler cannot catch. Strongly typed IDs make such mixups a compile-time error, eliminating an entire class of subtle runtime defects. They also serve as documentation, making it immediately clear from the type what entity an ID refers to.

## Generic typed ID implementation

No library is required — Scala 3's `opaque type` provides zero-cost generic IDs natively.

Put the following in a utility package (or subproject):

```scala
import java.util.UUID

object ids:

  opaque type SID[T] = String
  object SID:
    def apply[T](value: String): SID[T]         = value
    def random[T]: SID[T]                        = apply(UUID.randomUUID().toString)
    def unapply[T](id: SID[T]): Some[String]     = Some(id)
    extension [T](id: SID[T]) def unwrap: String = id

  opaque type LID[T] = Long
  object LID:
    def apply[T](value: Long): LID[T]          = value
    def unapply[T](id: LID[T]): Some[Long]     = Some(id)
    extension [T](id: LID[T]) def unwrap: Long = id
```

## Using typed IDs

Import the `ids` object to bring `SID`, `LID`, and their extensions into scope:

```scala
import myapp.ids.*

type SID[T] = ids.SID[T]  // optional re-export alias for convenience
```

Add utility methods and typeclass instances directly inside the companion. For example:

```scala
// inside object SID
given [T]: Encoder[SID[T]] = Encoder[String].contramap(_.unwrap)
given [T]: Decoder[SID[T]] = Decoder[String].map(SID(_))
```

Now `SID` and `LID` are ready to be used in domain entities:

```scala
case class User(id: SID[User], name: String, active: Boolean)
case class Role(id: SID[Role], userId: SID[User], name: String)
case class Order(id: LID[Order], userId: SID[User])
```

### Bad

```scala
// Plain primitives — compiler cannot distinguish user IDs from role IDs
case class User(id: String, name: String, active: Boolean)
case class Role(id: String, userId: String, name: String)

def findUser(id: String): User = ???

val roleId = "r-456"
findUser(roleId)  // compiles — silent bug
```

### Good

```scala
import myapp.ids.*

case class User(id: SID[User], name: String, active: Boolean)
case class Role(id: SID[Role], userId: SID[User], name: String)

def findUser(id: SID[User]): User = ???

val uid = SID[User]("u-123")
val rid = SID[Role]("r-456")

findUser(rid)  // compile error — type mismatch: SID[Role] vs SID[User]
findUser(uid)  // OK
```

## Accessing the underlying value

Use `.unwrap` defined as an extension in the companion:

```scala
val raw: String = uid.unwrap
```

## Pattern matching

```scala
val SID(raw) = uid   // raw: String
```
