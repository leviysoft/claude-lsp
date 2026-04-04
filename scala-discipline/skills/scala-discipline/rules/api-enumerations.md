# api-enumerations

> avoid `scala.Enumeration`; use `enumeratum` (Scala 2) or native `enum` (Scala 3)

## Why It Matters

Scala's built-in `Enumeration` has poor type safety — pattern matching on it is not exhaustive, and its values are all typed as `Value` rather than distinct types, making codec integration cumbersome. Both `enumeratum` (Scala 2) and native `enum` (Scala 3) provide proper types per member, exhaustiveness checking, and built-in value enumeration.

## Migration note

Do not flag `enumeratum` usage in a Scala 3 project as a violation if it is already widely used in that codebase (e.g. after a Scala 2 → 3 migration). In that context, `enumeratum` is an acceptable legacy choice. Only prefer native `enum` for *new* enumerations or when `enumeratum` would be introduced for the first time.

## Examples

### Bad

```scala
// scala.Enumeration — values are all typed as Color.Value, no exhaustiveness checks
object Color extends Enumeration {
  val Red, Green, Blue = Value
}

// Pattern match is not exhaustive — the compiler cannot warn about missing cases
def describe(c: Color.Value): String = c match {
  case Color.Red  => "red"
  case Color.Blue => "blue"
  // Green silently unhandled — no compiler warning
}
```

### Good – Scala 2 (`enumeratum`)

```scala
import enumeratum._

sealed trait Color extends EnumEntry
object Color extends Enum[Color] with CirceEnum[Color] {
  val values = findValues

  case object Red   extends Color
  case object Green extends Color
  case object Blue  extends Color
}

// Name-based lookup is built in
Color.withNameOption("Red")    // Some(Red)
Color.withNameOption("Purple") // None

// Full list of values available without manual maintenance
Color.values // IndexedSeq(Red, Green, Blue)
```

### Good – Scala 3 (native `enum`)

```scala
enum Color derives CirceCodec:
  case Red, Green, Blue

// Pattern match is exhaustive — compiler warns on missing cases
def describe(c: Color): String = c match
  case Color.Red   => "red"
  case Color.Green => "green"
  case Color.Blue  => "blue"

// Name-based lookup
Color.valueOf("Red")  // Color.Red

// Full list of values
Color.values // Array(Red, Green, Blue)
```
