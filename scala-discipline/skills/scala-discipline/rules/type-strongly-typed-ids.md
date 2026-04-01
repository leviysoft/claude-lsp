# type-stringly-typed-ids

> All entity identifiers should be strongly typed.

## Why It Matters

Using plain primitive types (e.g., `String`) for identifiers allows accidentally passing a `SID[User]` where a `SID[Role]` is expected — a bug the compiler cannot catch. Strongly typed IDs make such mixups a compile-time error, eliminating an entire class of subtle runtime defects. They also serve as documentation, making it immediately clear from the type what entity an ID refers to.

## Generic typed id implemetation

First of all, ensure that there is no implementation in the project already!

Required dependencies:

```
    "com.softwaremill.common" %% "tagging" % "2.3.5"
```

Put somewhere in utility package (or subproject) the following code:

```scala
    import com.softwaremill.tagging._

    trait IDCompanion[I] {
      type Aux[T] >: I @@ T <: I @@ T

      def apply[T](id: I): I @@ T = id.taggedWith[T]

      def unapply(id: I @@ ?): Some[I] = Some(id)
    }
```

## Using typed ids

For any primitive type, you need to create a strongly typed ID and add the following implementation:

```scala
    type SID[T] = SID.Aux[T]

    object SID extends IDCompanion[String] {
      def random[T]: SID[T] = SID(UUID.randomUUID().toString)
    }
```

Type alias is added for conciseness. Any typeclass instances (codecs, etc.) and utility functions can be added inside the object (as shown). For example:
```scala
    //inside object SID from previous example
    implicit def keyEncoderForSID[T]: KeyEncoder[SID[T]] = identity[SID[T]](_)
    implicit def keyDecoderForSID[T]: KeyDecoder[SID[T]] = (key: String) => Some(SID(key))
```

Now SID is ready to be used in domain entities:

```scala
    case class User(id: SID[User], name: String, active: Boolean)
    case class Role(id: SID[Role], userId: SID[User], name: String)
```