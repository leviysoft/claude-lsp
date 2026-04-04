# api-refinement-scala-3

> When a field has known value constraints, encode them in the type with `iron` (Scala 3)

## Why It Matters

Unrefined types do not communicate their valid ranges or invariants — readers must hunt for validation logic scattered across the codebase, and invalid values can slip through whenever that validation is bypassed. Refinement types make the constraints part of the type itself: the compiler rejects bad literals, and callers must explicitly handle invalid runtime input. This removes defensive runtime checks from business logic and turns invalid state into a compile error.

> **Encode only what the specification requires.** Only add constraints that are explicitly stated in the user story, domain model, or program specification. Do not invent constraints based on current implementation convenience or speculative assumptions — over-constraining makes APIs unnecessarily restrictive and couples types to assumptions that may not hold universally.

## Migration note

If the codebase already uses `refined` (e.g. migrated from Scala 2), **do not flag existing `refined` usage as a violation** and **do not introduce `iron`**. Continue using `refined` for any new code in that codebase to keep the dependency set consistent. Only use `iron` when no refinement library is established yet, or when the user explicitly asks to migrate away from `refined`.

## Setup

```scala
libraryDependencies += "io.github.iltotore" %% "iron" % "2.6.0"
```

```scala
import io.github.iltotore.iron.*
import io.github.iltotore.iron.constraint.numeric.*
import io.github.iltotore.iron.constraint.string.*
import io.github.iltotore.iron.constraint.collection.*
```

## Examples

### Bad

```scala
// No constraint in the type — caller must know `age` must be positive, `name` non-empty
case class User(name: String, age: Double)

def log(x: Double): Double = {
  require(x > 0, "x must be positive")
  Math.log(x)
}
```

### Good

```scala
import io.github.iltotore.iron.constraint.numeric.Positive
import io.github.iltotore.iron.constraint.collection.MinLength

case class User(name: String :| MinLength[1], age: Double :| Positive)

def log(x: Double :| Positive): Double =
  Math.log(x) // used like a normal Double

// Compile-time verification for literals
log(1.0)   // OK
log(-1.0)  // Compile-time error: Should be strictly positive

// Runtime validation
val runtimeValue: Double = ???
runtimeValue.refineEither[Positive].map(log)   // monadic style
```

### Bad

```scala
// Throws at runtime or forces caller to handle Option
def divide(dividend: Double, divisor: Double): Double = {
  if (divisor == 0.0) throw new IllegalArgumentException(s"divisor cannot be 0!")
  dividend / divisor
}

def divide(dividend: Double, divisor: Double): Option[Double] =
  Option.when(divisor != 0.0)(dividend / divisor)
```

### Good

```scala
import io.github.iltotore.iron.constraint.any.{Not, StrictEqual}

// Constraint is part of the signature; division by zero is impossible
def divide(dividend: Double, divisor: Double :| Not[StrictEqual[0.0]]): Double =
  dividend / divisor
```

## Available Constraints

### Global (`io.github.iltotore.iron.constraint.any`)

- `StrictEqual`: checks if a value is equal to a given one
- `Not`: negates another constraint (alias: `!`)
- `DescribedAs`: attaches a custom description to a constraint
- `True`: always-true constraint
- `False`: always-false constraint
- `Xor`: boolean XOR between two constraints
- `In`: checks if a value is in a given value tuple

### Char (`io.github.iltotore.iron.constraint.char`)

- `Digit`: checks if a character is a digit
- `Letter`: checks if a character is a letter
- `LowerCase`: checks if a character is a lower case character
- `UpperCase`: checks if a character is an upper case character
- `Whitespace`: checks if a character is a whitespace character
- `Special`: checks if a character is a special character (not a digit nor a letter)

### Numeric (`io.github.iltotore.iron.constraint.numeric`)

- `Less`: checks if a value is less than a given one
- `Greater`: checks if a value is greater than a given one
- `LessEqual`: checks if a value is less than or equal to a given one
- `GreaterEqual`: checks if a value is greater than or equal to a given one
- `Positive`: checks if a value is strictly positive
- `Negative`: checks if a value is strictly negative
- `Positive0`: checks if a value is positive or zero
- `Negative0`: checks if a value is negative or zero
- `Interval.Closed`: checks if a value is in a closed interval
- `Interval.Open`: checks if a value is in an open interval
- `Interval.OpenClosed`: checks if a value is in an open-closed interval
- `Interval.ClosedOpen`: checks if a value is in a closed-open interval
- `Infinity`: checks if a value is infinite (positive or negative)
- `NaN`: checks if a value is not a representable number
- `Multiple`: checks if a value is a multiple of another one
- `Divide`: checks if a value is a divisor of another one
- `Odd`: checks if a value is odd
- `Even`: checks if a value is even

### Collection (`io.github.iltotore.iron.constraint.collection`)

- `ForAll`: checks if a constraint passes for all elements
- `Exists`: checks if a constraint passes for at least one element
- `Length`: checks if the collection length satisfies a given constraint
- `Empty`: checks if a collection is empty
- `FixedLength`: checks if a collection has a fixed length
- `MinLength`: checks if a collection has a minimum length
- `MaxLength`: checks if a collection has a maximum length
- `Contains`: checks if a collection contains a given element
- `Head`: checks if a collection's head satisfies a given constraint
- `Last`: checks if a collection's last element satisfies a given constraint
- `Tail`: checks if a collection's tail satisfies a given constraint
- `Init`: checks if a collection's init satisfies a given constraint

### String (`io.github.iltotore.iron.constraint.string`)

Note: as `String` is an `Iterable[Char]`, collection constraints also apply.

- `Blank`: checks if a string is blank (empty or only whitespace)
- `StartWith`: checks if a string starts with a given prefix
- `EndWith`: checks if a string ends with a given suffix
- `Match`: checks if a string matches a given regular expression
- `Alphanumeric`: checks if a string contains only alphanumeric characters
- `LettersLowerCase`: checks if all letters are lower-cased
- `LettersUpperCase`: checks if all letters are upper-cased
- `Trimmed`: checks if a string has no leading or trailing whitespace
- `ValidUUID`: checks if a string is a valid UUID
- `ValidURL`: checks if a string is a valid URL
- `SemanticVersion`: checks if a string is a valid semantic version
