# api-refinement-scala-2

> When a field has known value constraints, encode them in the type with `refined` (Scala 2)

## Why It Matters

Plain `Int`, `String`, or `Double` fields do not communicate their valid ranges — readers must hunt for validation logic scattered across the codebase, and invalid values can slip through whenever that validation is bypassed. Refinement types make the constraints part of the type itself: the compiler rejects bad literals, and callers must explicitly handle invalid runtime input. This removes defensive runtime checks from business logic and turns invalid state into a compile error.

## Setup

```scala
libraryDependencies += "eu.timepit" %% "refined" % "0.11.2"
```

```scala
import eu.timepit.refined._
import eu.timepit.refined.api.Refined
import eu.timepit.refined.auto._
import eu.timepit.refined.numeric._
import eu.timepit.refined.string._
```

## Examples

### Bad

```scala
// No constraint in the type — caller must know `age` must be positive, `name` non-empty
case class User(name: String, age: Int)

def createUser(name: String, age: Int): User = {
  require(name.nonEmpty, "name must not be empty")
  require(age > 0, "age must be positive")
  User(name, age)
}
```

### Good

```scala
import eu.timepit.refined.collection.NonEmpty
import eu.timepit.refined.numeric.Positive

case class User(name: String Refined NonEmpty, age: Int Refined Positive)

// Compile-time verification for literals
val user: User = User("Alice", 30)

// Runtime validation returns Either
def createUser(name: String, age: Int): Either[String, User] =
  for {
    n <- refineV[NonEmpty](name)
    a <- refineV[Positive](age)
  } yield User(n, a)
```

### Bad

```scala
// Throws at runtime or forces caller to handle Option
def divide(dividend: Double, divisor: Double): Double = {
  if (divisor == 0.0) throw new IllegalArgumentException(s"${nameOf(divisor)} cannot be 0!")
  dividend / divisor
}

def divide(dividend: Double, divisor: Double): Option[Double] =
  Option.when(divisor != 0.0)(dividend / divisor)
```

### Good

```scala
import eu.timepit.refined.generic.Equal

// Constraint is part of the signature; division by zero is impossible
def divide(dividend: Double, divisor: Double Refined Not[Equal[0.0]]): Double =
  dividend / divisor
```

### Type inference between refined types

```scala
// A value refined with Greater[10] can be widened to Positive automatically
val precise: Int Refined Greater[10] = 42
val general: Int Refined Positive    = precise  // compile-time widening — no cast needed
```

## Available Predicates

### boolean

- `True`: always true
- `False`: always false
- `Not[P]`: negation of `P`
- `And[A, B]`: conjunction of `A` and `B`
- `Or[A, B]`: disjunction of `A` and `B`
- `Xor[A, B]`: exclusive disjunction of `A` and `B`
- `Nand[A, B]`: negated conjunction of `A` and `B`
- `Nor[A, B]`: negated disjunction of `A` and `B`
- `AllOf[PS]`: conjunction of all predicates in `PS`
- `AnyOf[PS]`: disjunction of all predicates in `PS`
- `OneOf[PS]`: exclusive disjunction of all predicates in `PS`

### char

- `Digit`: checks if a `Char` is a digit
- `Letter`: checks if a `Char` is a letter
- `LetterOrDigit`: checks if a `Char` is a letter or digit
- `LowerCase`: checks if a `Char` is a lower case character
- `UpperCase`: checks if a `Char` is an upper case character
- `Whitespace`: checks if a `Char` is white space

### collection

- `Contains[U]`: checks if an `Iterable` contains a value equal to `U`
- `Count[PA, PC]`: counts elements satisfying `PA` and passes the result to `PC`
- `Empty`: checks if an `Iterable` is empty
- `NonEmpty`: checks if an `Iterable` is not empty
- `Forall[P]`: checks if `P` holds for all elements
- `Exists[P]`: checks if `P` holds for some elements
- `Head[P]`: checks if `P` holds for the first element
- `Index[N, P]`: checks if `P` holds for the element at index `N`
- `Init[P]`: checks if `P` holds for all but the last element
- `Last[P]`: checks if `P` holds for the last element
- `Tail[P]`: checks if `P` holds for all but the first element
- `Size[P]`: checks if the size satisfies `P`
- `MinSize[N]`: checks if the size is greater than or equal to `N`
- `MaxSize[N]`: checks if the size is less than or equal to `N`

### generic

- `Equal[U]`: checks if a value is equal to `U`

### numeric

- `Less[N]`: checks if a numeric value is less than `N`
- `LessEqual[N]`: checks if a numeric value is less than or equal to `N`
- `Greater[N]`: checks if a numeric value is greater than `N`
- `GreaterEqual[N]`: checks if a numeric value is greater than or equal to `N`
- `Positive`: checks if a numeric value is greater than zero
- `NonPositive`: checks if a numeric value is zero or negative
- `Negative`: checks if a numeric value is less than zero
- `NonNegative`: checks if a numeric value is zero or positive
- `Interval.Open[L, H]`: checks if a numeric value is in the interval (`L`, `H`)
- `Interval.OpenClosed[L, H]`: checks if a numeric value is in the interval (`L`, `H`]
- `Interval.ClosedOpen[L, H]`: checks if a numeric value is in the interval [`L`, `H`)
- `Interval.Closed[L, H]`: checks if a numeric value is in the interval [`L`, `H`]
- `Modulo[N, O]`: checks if an integral value modulo `N` is `O`
- `Divisible[N]`: checks if an integral value is evenly divisible by `N`
- `NonDivisible[N]`: checks if an integral value is not evenly divisible by `N`
- `Even`: checks if an integral value is evenly divisible by 2
- `Odd`: checks if an integral value is not evenly divisible by 2
- `NonNaN`: checks if a floating-point number is not NaN

### string

- `EndsWith[S]`: checks if a `String` ends with the suffix `S`
- `IPv4`: checks if a `String` is a valid IPv4
- `IPv6`: checks if a `String` is a valid IPv6
- `MatchesRegex[S]`: checks if a `String` matches the regular expression `S`
- `Regex`: checks if a `String` is a valid regular expression
- `StartsWith[S]`: checks if a `String` starts with the prefix `S`
- `Uri`: checks if a `String` is a valid URI
- `Url`: checks if a `String` is a valid URL
- `Uuid`: checks if a `String` is a valid UUID
- `ValidByte`: checks if a `String` is a parsable `Byte`
- `ValidShort`: checks if a `String` is a parsable `Short`
- `ValidInt`: checks if a `String` is a parsable `Int`
- `ValidLong`: checks if a `String` is a parsable `Long`
- `ValidFloat`: checks if a `String` is a parsable `Float`
- `ValidDouble`: checks if a `String` is a parsable `Double`
- `ValidBigInt`: checks if a `String` is a parsable `BigInt`
- `ValidBigDecimal`: checks if a `String` is a parsable `BigDecimal`
- `Xml`: checks if a `String` is well-formed XML
- `XPath`: checks if a `String` is a valid XPath expression
- `Trimmed`: checks if a `String` has no leading or trailing whitespace
- `HexStringSpec`: checks if a `String` represents a hexadecimal number
