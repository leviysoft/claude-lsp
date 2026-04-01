# type-literally

> Use the `literally` library to define compile-time validated string interpolators for constrained types.

## Why It Matters

Many types are constructed from strings validated at runtime — valid values are known at compile time but errors are only caught when the code executes. With `literally` you wire the same validation logic into a macro, turning invalid literals into **compilation errors**.

## Bad

Constructing a value from a hardcoded string that is only validated at runtime:

```scala
// Throws IllegalArgumentException at runtime if the pattern is invalid
val fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ii")
```

## Good

Use the `dtf` interpolator — the pattern is validated at compile time:

```scala
import literals.*

val fmt = dtf"dd/MM/yyyy HH:mm:ss"   // compiles fine

val bad = dtf"dd/MM/yyyy HH:mm:ii"
// error: Unknown pattern letter: i
```

## Quick Start

```sbt
libraryDependencies += "org.typelevel" %% "literally" % "1.2.0"
```

## Implementing an interpolator

Defining a custom string interpolator for a type `A` requires:
- an `org.typelevel.literally.Literally[A]` instance containing the validation logic
- an extension method on `StringContext` that invokes it

### Scala 3

```scala
import org.typelevel.literally.Literally
import java.time.format.DateTimeFormatter
import scala.quoted.*

object literals:
  extension (inline ctx: StringContext)
    inline def dtf(inline args: Any*): DateTimeFormatter =
      ${DateTimeFormatLiteral('ctx, 'args)}

  object DateTimeFormatLiteral extends Literally[DateTimeFormatter]:
    def validate(s: String)(using Quotes): Either[String, Expr[DateTimeFormatter]] =
      try
        DateTimeFormatter.ofPattern(s)
        Right('{DateTimeFormatter.ofPattern(${Expr(s)})})
      catch
        case e: IllegalArgumentException => Left(e.getMessage)
```

### Scala 2

> **Note:** In Scala 2, macros must be defined in a separate compilation unit from their usage.

```scala
import org.typelevel.literally.Literally
import java.time.format.DateTimeFormatter
import scala.language.experimental.macros
import scala.reflect.macros.blackbox.Context

object literals {
  implicit class DateTimeFormatStringContext(val sc: StringContext) extends AnyVal {
    def dtf(args: Any*): DateTimeFormatter = macro DateTimeFormatLiteral.make
  }

  object DateTimeFormatLiteral extends Literally[DateTimeFormatter] {
    def validate(c: Context)(s: String): Either[String, c.Expr[DateTimeFormatter]] = {
      import c.universe._
      try {
        DateTimeFormatter.ofPattern(s)
        Right(c.Expr(q"java.time.format.DateTimeFormatter.ofPattern($s)"))
      } catch {
        case e: IllegalArgumentException => Left(e.getMessage)
      }
    }

    def make(c: Context)(args: c.Expr[Any]*): c.Expr[DateTimeFormatter] = apply(c)(args: _*)
  }
}
```
