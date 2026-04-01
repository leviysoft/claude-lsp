# dom-prove-properties

> Prove domain properties — such as structural projection — at compile time using typeclasses

## Why It Matters

Domain invariants like "every field in this DTO exists in the domain model with the same type" are often left implicit — verified only when a conversion fails at runtime or caught by a diligent code reviewer. Encoding such constraints as typeclass evidence turns them into compiler-checked proofs: violations are compile errors, the invariant is self-documenting at every call site, and there is zero runtime cost.

The technique is general: define a typeclass that witnesses the property, derive instances via the generic representation of your types, and use the typeclass as an implicit/given constraint wherever the property must hold.

## Example: Structural Projection with `PropSubset`

`PropSubset[P, S]` witnesses that every field in `P` exists in `S` with the same name and type. Using it as a constraint on a function or class proves — at compile time — that `P` is a valid projection of `S`.

### Scala 2 (shapeless)

#### Setup

```scala
libraryDependencies += "com.chuusai" %% "shapeless" % "2.3.12"
```

#### Implementation

```scala
import shapeless._
import shapeless.labelled.FieldType
import shapeless.ops.record.Selector

import scala.annotation.implicitNotFound

/*
  Witnesses, that all fields of `Projection` exists in `Source` with the same types and names
*/
@implicitNotFound("${Projection} is not a valid projection of ${Source}")
trait PropSubset[Projection, Source]

object PropSubset {
  def apply[P, S](implicit propSubset: PropSubset[P, S]): PropSubset[P, S] = propSubset

  private val anySubset = new PropSubset[Any, Any] {}

  implicit def identitySubset[T]: PropSubset[T, T] =
    anySubset.asInstanceOf[PropSubset[T, T]]

  implicit def optionSubset[P, S](implicit notEq: P =:!= S, ps: PropSubset[P, S]): PropSubset[Option[P], Option[S]] =
    ps.asInstanceOf[PropSubset[Option[P], Option[S]]]

  implicit def hNilSubset[S <: HList]: PropSubset[HNil, S] =
    anySubset.asInstanceOf[PropSubset[HNil, S]]

  implicit def hListSubset[K <: Symbol, PH, PT <: HList, SF, S <: HList](implicit
    notEq: (FieldType[K, PH] :: PT) =:!= S,
    s: Selector.Aux[S, K, SF],
    ps0: Lazy[PropSubset[PH, SF]],
    psTail: PropSubset[PT, S]
  ): PropSubset[FieldType[K, PH] :: PT, S] =
    anySubset.asInstanceOf[PropSubset[FieldType[K, PH] :: PT, S]]

  implicit def genericSubset[P, S, PHL <: HList, SHL <: HList](implicit
    notEq: P =:!= S,
    pgen: LabelledGeneric.Aux[P, PHL],
    sgen: LabelledGeneric.Aux[S, SHL],
    hlSubset: Lazy[PropSubset[PHL, SHL]]
  ): PropSubset[P, S] =
    hlSubset.asInstanceOf[PropSubset[P, S]]
}
```

#### Usage

```scala
import java.time.LocalDate

case class Inner(fc: String)
case class Source(id: Int, text: String, date: LocalDate, in: Option[Inner])
case class InnerP(fc: String)
case class Projection(id: Int, in: Option[InnerP], text: String)
case class Projection2(id: Int, text: String, yoba: String)

// Use as a constraint — the compiler verifies the relationship
def narrowTo[P, S](s: S)(implicit ps: PropSubset[P, S]): Unit = ???

PropSubset[Projection, Source]          // OK
// PropSubset[Projection2, Source]      // Compile error: Projection2 is not a valid projection of Source
```

### Scala 3 (shapeless3 / Mirror)

#### Setup

```scala
libraryDependencies += "org.typelevel" %% "shapeless3-deriving" % "3.4.3"
```

#### Implementation

```scala
import scala.annotation.implicitNotFound
import shapeless3.deriving.*
import scala.deriving.Mirror

/*
  Witnesses, that all fields of `Projection` exists in `Source` with the same types and names
*/
@implicitNotFound("${Projection} is not a valid projection of ${Source}")
trait PropSubset[Projection, Source]

object PropSubset:
  private val anySubset = new PropSubset[Any, Any] {}

  def apply[P, S](using ps: PropSubset[P, S]): PropSubset[P, S] = ps

  given identityPs[T]: PropSubset[T, T] = anySubset.asInstanceOf[PropSubset[T, T]]

  given optionSubset[P, S](using notEq: P =:!= S, ps: PropSubset[P, S]): PropSubset[Option[P], Option[S]] =
    ps.asInstanceOf[PropSubset[Option[P], Option[S]]]

  given nilSubset[P, S](using
    kp: K0.ProductGeneric[P] { type MirroredElemTypes <: EmptyTuple },
    ks: K0.ProductGeneric[S]
  ): PropSubset[P, S] = anySubset.asInstanceOf[PropSubset[P, S]]

  given macroSubset[P, S](using notEq: P =:!= S, pm: Mirror.ProductOf[P], ps: Mirror.ProductOf[S]): PropSubset[P, S] =
    tupleSubset[pm.MirroredElemTypes, ps.MirroredElemTypes].asInstanceOf[PropSubset[P, S]]

  given tupleSubset[PT <: NonEmptyTuple, ST <: NonEmptyTuple](using Head[PT] =:= Head[ST], PropSubset[Tail[PT], Tail[ST]]): PropSubset[PT, ST] =
    anySubset.asInstanceOf[PropSubset[PT, ST]]

type Head[T] = T match
  case h *: t => h
type Tail[T] = T match
  case h *: t => t
```

#### Usage

```scala
case class Employee(id: Int, name: String, department: String)
case class EmpProj(id: Int, name: String)

// Use as a given constraint on a function
def okp[P, S](using PropSubset[P, S]) = ()

okp[EmpProj, Employee]   // OK
// okp[Employee, EmpProj] // Compile error: Employee is not a valid projection of EmpProj
```
