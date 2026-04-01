# dom-sized-collections

> Use sized (length-indexed) collections to enforce compile-time constraints between the sizes of two or more collections

## Why It Matters

When an operation requires that two or more collections have sizes related by a specific constraint — equal lengths, compatible matrix dimensions, paired coordinates — encoding that constraint in plain `List` or `Vector` leaves it implicit. Violations surface only at runtime (an exception, a wrong result, or a crash), and there is no machine-checked documentation of the invariant.

Sized collections lift these constraints into the type system. Passing a vector of the wrong length, or multiplying incompatible matrices, becomes a compile error. The invariant is self-documenting at every call site and carries zero runtime cost.

**Only apply this technique when the size relationship is explicitly stated in the specification or task, or is mathematically inherent to the operation** (e.g. dot product requires equal-length vectors; matrix multiplication requires that the column count of the left operand equals the row count of the right operand). Do not speculatively add sized types for collections whose size relationship is incidental, undocumented, or may change.

### When NOT to use sized collections

| Goal | Better tool |
|------|-------------|
| Ensure a collection is non-empty | `NonEmptyList` / `refined` `NonEmpty` / `iron` `Not[Empty]` (see `api-refinement-scala-2`, `api-refinement-scala-3`) |
| Enforce a fixed size on a single collection | A `refined` / `iron` size predicate |
| Relate sizes of two or more collections | Sized collections — see below |

---

## Scala 2 — typequux

### Setup

```scala
libraryDependencies += "com.simianquant" %% "typequux" % "0.9.0"
```

### Examples

#### Bad

```scala
// No compile-time guarantee that xs and ys have equal length
def dot(xs: List[Int], ys: List[Int]): Int =
  xs.zip(ys).map { case (x, y) => x * y }.sum

// No compile-time guarantee on matrix dimensions
type Matrix = List[List[Int]]
def mul(lhs: Matrix, rhs: Matrix): Matrix = ???
```

#### Good — dot product

```scala
import typequux._
import typequux.Typequux._

type Vec[N <: Dense] = SizedVector[N, Int]

def dot[N <: Dense](xs: Vec[N], ys: Vec[N]): Int =
  xs.zip(ys).map { case (x, y) => x * y }.backing.sum
```

The shared type parameter `N` ensures both vectors have the same length. Passing mismatched lengths is a compile error.

#### Good — matrix multiplication

```scala
import typequux._
import typequux.Typequux._

type Matrix[R <: Dense, C <: Dense] = SizedVector[R, SizedVector[C, Int]]

def mul[R1 <: Dense, C1 <: Dense, R2 <: Dense, C2 <: Dense](
  lhs: Matrix[R1, C1],
  rhs: Matrix[R2, C2]
)(implicit ev: C1 =:= R2): Matrix[R1, C2] = {
  val trhs = rhs.transpose
  lhs.map { row =>
    trhs.map { col =>
      row.zip(col).map { case (x, y) => x * y }.backing.sum
    }
  }
}
```

The `implicit ev: C1 =:= R2` constraint encodes that the column count of `lhs` must equal the row count of `rhs`. Without it the call site will not compile.

---

## Scala 3 — tightbound

### Setup

```scala
libraryDependencies += "io.github.leviysoft" %% "tightbound" % "0.1.0"
```

`tightbound` provides `Vector[Size <: Int, A]` and `Matrix[Rows <: Int, Cols <: Int, A]` with built-in size-safe operations. No manual proof is needed — the library's API encodes the constraints directly in method signatures.

### Examples

#### Bad

```scala
def dot(xs: List[Int], ys: List[Int]): Int =
  xs.zip(ys).map(_ * _).sum
```

#### Good — dot product (built-in)

```scala
import tightbound.*

val v1: Vector[3, Int] = Vector.of(1, 2, 3)
val v2: Vector[3, Int] = Vector.of(4, 5, 6)

v1 dot v2   // 32 — size equality enforced by the shared type parameter

// v1 dot Vector.of(1, 2)
// compile error: Found Vector[(2 : Int), Int], Required Vector[(3 : Int), Int]
```

#### Good — matrix multiplication (built-in)

```scala
import tightbound.*

val m1: Matrix[2, 3, Int] = Matrix {
  Vector.of(
    Vector.of(1, 2, 3),
    Vector.of(4, 5, 6),
  )
}
val m2: Matrix[3, 1, Int] = Matrix {
  Vector.of(
    Vector.of(7),
    Vector.of(8),
    Vector.of(9),
  )
}

m1 x m2   // Matrix[2, 1, Int] — column/row compatibility verified at compile time

// m2 x m1
// compile error: column count of m2 (1) does not match row count of m1 (2)
```
