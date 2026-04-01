# api-enumerations-scala-2

> for enumerations in Scala 2 use `enumeratum` library

## Why It Matters

Scala's built-in `Enumeration` has poor type safety — pattern matching on it is not exhaustive, and its values are all typed as `Value` rather than distinct types, making codec integration cumbersome. `enumeratum` provides a `values` sequence and `withName`/`withNameOption` lookups while keeping each member a proper type. It also integrates with common JSON and persistence libraries out of the box.

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

### Good

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
Color.withNameOption("Red")   // Some(Red)
Color.withNameOption("Purple") // None

// Full list of values available without manual maintenance
Color.values // IndexedSeq(Red, Green, Blue)
```