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

`tightbound` provides `Vector[Size <: Int, A]` and `Matrix[Rows <: Int, Cols <: Int, A]`. Size constraints are encoded directly in type parameters — the same shared-parameter pattern as Scala 2, but without implicit evidence.

### Examples

#### Bad

```scala
def dot(xs: List[Int], ys: List[Int]): Int =
  xs.zip(ys).map(_ * _).sum
```

#### Good — dot product

```scala
import tightbound.*

def dot[N <: Int](xs: Vector[N, Int], ys: Vector[N, Int]): Int =
  Vector.map2(xs, ys, _ * _).sum
```

The shared type parameter `N` ensures both vectors have the same length. Passing mismatched lengths is a compile error.

#### Good — matrix multiplication

```scala
import tightbound.*

def mul[R <: Int, C <: Int, C2 <: Int](
  lhs: Matrix[R, C, Int],
  rhs: Matrix[C, C2, Int]
): Matrix[R, C2, Int] =
  rhs.transpose.mapRows(lhs.linearMap).transpose
```

The shared type parameter `C` encodes that the column count of `lhs` must equal the row count of `rhs`. Mismatched dimensions are a compile error.
