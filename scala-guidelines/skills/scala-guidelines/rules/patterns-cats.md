# patterns-cats

> Prefer Cats type class methods over hand-rolled equivalents; consult the glossary below when unsure what method to reach for.

## Why It Matters

Cats provides a rich, lawful vocabulary for effectful and functional programming in Scala. Manually re-implementing traversals, error handling, or effect sequencing obscures intent, introduces subtle bugs, and misses free optimisations. Using the canonical Cats methods communicates intent precisely — `traverse` over a manual fold-into-applicative, `handleErrorWith` over a nested `flatMap`/recover, `*>` over `map(_ => ())` — and keeps code consistent across the codebase. The glossary below is a quick-reference cheat sheet organised by type class.

## Imports

Always use `import cats.syntax.all._`. Never use `import cats.implicits._` — it is a legacy alias and its use should be avoided in all new and existing code.

## Examples

### Bad

```scala
// Manually sequencing a list of effects
list.foldRight(IO.pure(List.empty[B])) { (a, acc) =>
  f(a).flatMap(b => acc.map(b :: _))
}

// Discarding left result
fa.flatMap(_ => fb)

// Conditional effect
if (condition) fa else IO.unit

// Chaining Option-producing effects without OptionT
def findUser(id: UserId): IO[Option[User]] = ???
def findRole(user: User): IO[Option[Role]] = ???

findUser(id).flatMap {
  case None       => IO.pure(None)
  case Some(user) => findRole(user)
}

// Nested map instead of monad transformer
fa.map(_.map(f))          // F[Option[A]] — use OptionT
fa.map(_.map(f))          // F[Either[E, A]] — use EitherT
fa.flatMap(opt => IO(opt.map(f)))
```

### Good

```scala
list.traverse(f)        // Traverse[List].traverse

fa *> fb                // Apply#productR

fa.whenA(condition)     // Applicative#whenA

// Use OptionT to eliminate manual Option-unwrapping
(for {
  user <- OptionT(findUser(id))
  role <- OptionT(findRole(user))
} yield role).value

// Use monad transformers instead of nested map/flatMap
OptionT(fa).map(f).value            // instead of fa.map(_.map(f))
EitherT(fa).map(f).value            // instead of fa.map(_.map(f))
OptionT(fa).flatMap(a => OptionT(g(a))).value  // instead of fa.flatMap { case None => ...; case Some(a) => g(a) }
```

## Reference: Cats Method Glossary

### Type-Classes over an `F[_]`

#### Functor

| Type                           | Method Name    | Notes |
|--------------------------------|----------------|-------|
| `F[A] => F[Unit]`              | `void`         |
| `F[A] => B => F[B]`            | `as`           |
| `F[A] => (A => B) => F[B]`     | `map`          |
| `F[A] => (A => A1) => F[A1])`  | `mapOrKeep`    | A1 >: A, the (A => A1) is a PartialFunction
| `F[A] => (A => B) => F[(A,B)]` | `fproduct`     |
| `F[A] => (A => B) => F[(B,A)]` | `fproductLeft` |
| `F[A] => B => F[(B, A)]`       | `tupleLeft`    |
| `F[A] => B => F[(A, B)]`       | `tupleRight`   |
| `(A => B) => (F[A] => F[B])`   | `lift`         |

#### Apply

| Type          | Method Name | Symbol   |
| ------------- |--------------|------------|
| `F[A] => F[B] => F[A]` | `productL`  | `<*`
| `F[A] => F[B] => F[B]` | `productR`  | `*>`
| `F[A] => F[B] => F[(A,B)]` | `product`  |
| `F[A => B] => F[A] => F[B]` | `ap`  |  `<*>`
| `F[A => B => C] => F[A] => F[B] => F[C]` | `ap2`  |
| `F[A] => F[B] => (A => B => C) => F[C]` | `map2` |

#### Applicative

| Type          | Method Name | Notes   |
| ------------- |--------------|------------|
| `A => F[A]`   | `pure` |
| `=> F[Unit]`  | `unit` |
| `Boolean => F[Unit] => F[Unit]` | `whenA`   | Performs effect iff condition is true
|                                 | `unlessA` | Adds effect iff condition is false

#### FlatMap

| Type          | Method Name |
| ------------- |---------------|
| `F[F[A]] => F[A]` | `flatten`  |
| `F[A] => (A => F[B]) => F[B]` | `flatMap`
| `F[A] => (A => F[B]) => F[(A,B)]` | `mproduct`
| `F[Boolean] => F[A] => F[A] => F[A]` | `ifM`
| `F[A] => (A => F[B]) => F[A]` | `flatTap`

#### FunctorFilter

| Type        | Method Name   | Notes  |
|-------------|---------------|--------|
| `F[A] => (A => Boolean) => F[A]` |   `filter` |
| `F[A] => (A => Option[B]) => F[B]` | `mapFilter` |
| `F[A] => (A => B) => F[B]` | `collect` | The `A => B` is a PartialFunction
| `F[Option[A]] => F[A]` | `flattenOption` |


#### ApplicativeError

The source code of `Cats` uses the `E` type variable for the error type.

| Type         | Method Name  | Notes |
|--------------|--------------|-------|
| `E => F[A]`   | `raiseError`   |
| `F[A] => F[Either[E,A]]`    | `attempt`     |
| `F[A] => (E => A) => F[A]`  | `handleError`   |
| `F[A] => (E => F[A]) => F[A]` | `handleErrorWith`  |
| `F[A] => (E => A) => F[A]` | `recover`  |  The `E => A` is a PartialFunction.
| `F[A] => (E => F[A]) => F[A]` | `recoverWith`  |  The `E => F[A]` is a PartialFunction.
| `F[A] => (E => F[Unit]) => F[A]` | `onError`  | The `E => F[Unit]` is a PartialFunction.
| `Either[E,A] => F[A]` | `fromEither` |
| `Option[A] => E => F[A]` | `liftFromOption` |

#### MonadError

Like the previous section, we use the `E` for the error parameter type.

| Type          | Method Name  | Notes  |
| ------------- |--------------|--------|
| `F[A] => E => (A => Boolean) => F[A]` | `ensure`
| `F[A] => (A => E) => (A => Boolean) => F[A]` | `ensureOr`
| `F[A] => (E => E) => F[A]`  | `adaptError` | The `E => E` is a PartialFunction.
| `F[Either[E,A]] => F[A]`     | `rethrow`


#### UnorderedFoldable

| Type          | Method Name  | Constraints
| ------------- |--------------|----------------
| `F[A] => Boolean` | `isEmpty` |
| `F[A] => Boolean` | `nonEmpty` |
| `F[A] => Long` | `size` |
| `F[A] => (A => Boolean) => Boolean`| `forall` |
| `F[A] => (A => Boolean) => Boolean`| `exists` |
| `F[A] => A`  | `unorderedFold` | `A: CommutativeMonoid`
| `F[A] => (A => B) => B`| `unorderedFoldMap` | `B: CommutativeMonoid`

#### Foldable

| Type          | Method Name  | Constraints
| ------------- |--------------|-----------
| `F[A] => A` | `fold` | `A: Monoid`
| `F[A] => B => ((B,A) => B) => B` | `foldLeft`
| `F[A] => (A => B) => B` | `foldMap` | `B: Monoid`
| `F[A] => (A => G[B]) => G[B]` | `foldMapM` | `G: Monad` and `B: Monoid`
| `F[A] => (A => B) => Option[B]` | `collectFirst` | The `A => B` is a `PartialFunction`
| `F[A] => (A => Option[B]) => Option[B]` | `collectFirstSome` |
| `F[A] => (A => G[B]) => G[Unit]` | `traverse_` | `G: Applicative`
| `F[G[A]] => G[Unit]` | `sequence_` | `G: Applicative`
| `F[A] => (A => Either[B, C]) => (F[B], F[C])` | `partitionEither` | `G: Applicative`

#### Reducible

| Type          | Method Name  | Constraints
| ------------- |--------------|-----------
| `F[A] => ((A,A) => A) => A` | `reduceLeft` |
| `F[A] => A` | `reduce`  | `A: Semigroup`   |

#### Traverse

| Type         | Method Name  | Constraints |
|------------|--------------|-----------|
| `F[G[A]] => G[F[A]]`  | `sequence` | `G: Applicative` |
| `F[A] => (A => G[B]) => G[F[B]]` | `traverse` | `G: Applicative` |
| `F[A] => (A => G[F[B]]) => G[F[B]]` | `flatTraverse` | `F: FlatMap` and `G: Applicative`
| `F[G[F[A]]] => G[F[A]]` | `flatSequence` | `G: Applicative` and `F: FlatMap`
| `F[A] => F[(A,Int)]` | `zipWithIndex` |
| `F[A] => ((A,Int) => B) => F[B]` | `mapWithIndex` |
| `F[A] => ((A,Int) => G[B]) => G[F[B]]` | `traverseWithIndexM` | `F: Monad`

#### SemigroupK
| Type         | Method Name  | Constraints |
|------------|--------------|-----------|
| `F[A] => F[A] => F[A]`| `combineK` |
| `F[A] => Int => F[A]` | `combineNK`
| `F[A] => F[B] => F[Either[A, B]]` | `sum` | `F: Functor`
| `IterableOnce[F[A]] => Option[F[A]]` | `combineAllOptionK`

#### MonoidK
| Type         | Method Name  | Constraints |
|------------|--------------|-----------|
| `F[A]` | `empty`
| `F[A] => Boolean` | `isEmpty`
| `IterableOnce[F[A]] => F[A]` | `combineAllK`

#### Alternative
| Type         | Method Name  | Constraints |
|------------|--------------|-----------|
| `F[G[A]] => F[A]`  | `unite` | `F: FlatMap` and `G: Foldable`
| `F[G[A, B]] => (F[A], F[B])`  | `separate` | `F: FlatMap` and `G: Bifoldable`
| `F[G[A, B]] => (F[A], F[B])`  | `separateFoldable` | `F: Foldable` and `G: Bifoldable`
| `Boolean => F[Unit]` | `guard`
| `IterableOnce[A] => F[A]` | `fromIterableOnce`
| `G[A] => F[A]` | `fromFoldable` | `G: Foldable`

#### NonEmptyAlternative
| Type         | Method Name  | Constraints |
|------------|--------------|-----------|
| `A => F[A] => F[A]` | `prependK`
| `F[A] => A => F[A]` | `appendK`

### Transformers

#### Constructors and wrappers

| Data Type  | is an alias or wrapper of |
|------------|--------------|
| `OptionT[F[_], A]`    | `F[Option[A]]`
| `EitherT[F[_], A, B]` | `F[Either[A,B]`
| `Kleisli[F[_], A, B]` | `A => F[B]`
| `Reader[A, B]` | `A => B`
| `ReaderT[F[_], A, B]` | `Kleisli[F, A, B]`
| `Writer[A, B]` | `(A,B)`
| `WriterT[F[_], A, B]` | `F[(A,B)]`
| `Tuple2K[F[_], G[_], A]` | `(F[A], G[A])`
| `EitherK[F[_], G[_], A]` | `Either[F[A], G[A]]`
| `FunctionK[F[_], G[_]]`   | `F[X] => G[X]` for every `X`
| `F ~> G`   | Alias of `FunctionK[F, G]`

#### OptionT

For convenience, in these types we use the symbol `OT` to abbreviate `OptionT`.

| Type     | Method Name  | Constraints |
|----------|--------------|-------------|
| `=> OT[F, A]` | `none` | `F: Applicative` |
| `A => OT[F, A]` | `some` or `pure` | `F: Applicative`
| `F[A] => OT[F, A]` | `liftF`  | `F: Functor`
| `Boolean => F[A] => OT[F, A]` | `whenF` | `F: Applicative`
| `F[Boolean] => F[A] => OT[F, A]` | `whenM` | `F: Monad`
| `OT[F, A] => F[Option[A]]` | `value`
| `OT[F, A] => A => Boolean => OT[F, A]` | `filter` | `F: Functor`
| `OT[F, A] => A => F[Boolean] => OT[F, A]` | `filterF` | `F: Monad`
| `OT[F, A] => (A => B) => OT[F, B]` | `map`  | `F: Functor`
| `OT[F, A] => (F ~> G) => OT[G, B]` | `mapK`
| `OT[F, A] => (A => Option[B]) => OT[F, B]` | `mapFilter` | `F: Functor`
| `OT[F, A] => B => (A => B) => F[B]` | `fold` or `cata`
| `OT[F, A] => (A => OT[F, B]) => OT[F,B]` | `flatMap`
| `OT[F, A] => (A => F[Option[B]]) => OT[F,B]` | `flatMapF`  | `F: Monad` |
| `OT[F, A] => A => F[A]` | `getOrElse` | `F: Functor`  |
| `OT[F, A] => F[A] => F[A]` | `getOrElseF` | `F: Monad`  |
| `OT[F, A] => OT[F, A] => OT[F, A]` | `orElse` | `F: Monad`

#### EitherT

Here, we use `ET` to abbreviate `EitherT`; and we use `A` and `B` as type variables for the left and right sides of the `Either`.

| Type     | Method Name  | Constraints |
|----------|--------------|-------------|
| `A => ET[F, A, B]` | `leftT` | `F: Applicative` |
| `B => ET[F, A, B]` | `rightT` | `F: Applicative` |
|                    | `pure` | `F: Applicative` |
| `F[A] => ET[F, A, B]` | `left`  | `F: Applicative` |
| `F[B] => ET[F, A, B]` | `right` | `F: Applicative` |
|                       | `liftF` | `F: Applicative` |
| `Either[A, B] => ET[F, A, B]` | `fromEither` | `F: Applicative` |
| `Option[B] => A => ET[F, A, B]` | `fromOption` | `F: Applicative` |
| `F[Option[B]] => A => ET[F, A, B]` | `fromOptionF` | `F: Functor` |
| `F[Option[B]] => F[A] => ET[F, A, B]` | `fromOptionM` | `F: Monad` |
| `Boolean => B => A => ET[F, A, B]` | `cond`   | `F: Applicative` |
| `ET[F, A, B] => (A => C) => (B => C) => F[C]` | `fold` | `F: Functor` |
| `ET[F, A, B] => ET[F, B, A]` | `swap` | `F: Functor` |
| `ET[F, A, A] => F[A]`        | `merge` |

#### Kleisli (or ReaderT)

Here, we use `Ki` as a short-hand for `Kleisli`.

| Type     | Method Name  | Constraints |
|----------|--------------|-------------|
| `Ki[F, A, B] => (A => F[B])` | `run`  |
| `Ki[F, A, B] => A => F[B]` | `apply`  |
| `A => Ki[F, A, A]` | `ask` | `F: Applicative`
| `B => Ki[F, A, B]` | `pure` | `F: Applicative`
| `F[B] => Ki[F, A, B]` | `liftF` |
| `Ki[F, A, B] => (C => A) => Ki[F, C, B]` | `local` |
| `Ki[F, A, B] => Ki[F, A, A]` | `tap` |
| `Ki[F, A, B] => (B => C) => Ki[F, A, C]` | `map` |
| `Ki[F, A, B] => (F ~> G) => Ki[G, A, B]` | `mapK` |
| `Ki[F, A, B] => (F[B] => G[C]) => Ki[F, A, C]` | `mapF` |
| `Ki[F, A, B] => Ki[F, A, F[B]]` | `lower` |


### Type Classes for types `F[_, _]`

#### Bifunctor

| Type          | Method Name  |
| ------------- |--------------|
| `F[A,B] => (A => C) => F[C,B]` | `leftMap` |
| `F[A,B] => (B => D) => F[A,D]` | `.rightFunctor` and `.map` |
| `F[A,B] => (A => C) => (B => D) => F[C,D]` | `bimap` |

##### Profunctor

| Type           | Method Name  |
--------|-------------
| `F[A, B] => (B => C) => F[A, C]` | `rmap`  |
| `F[A, B] => (C => A) => F[C, B]` | `lmap`  |
| `F[A, B] => (C => A) => (B => D) => F[C,D]` | `dimap`  |

##### Strong Profunctor

| Type  | Method Name |
--------|-------------|
| `F[A, B] => F[(A,C), (B,C)]` | `first`  |
| `F[A, B] => F[(C,A), (C,B)]` | `second` |

##### Compose, Category, Choice

| Type  | Method Name | Symbol |
--------|-------------|--------------|
| `F[A, B] => F[C, A] => F[C, B]` | `compose` | `<<<` |
| `F[A, B] => F[B, C] => F[A, C]` | `andThen` | `>>>` |
| `=> F[A,A]` | `id`  |
| `F[A, B] => F[C, B] => F[Either[A, C], B]` | `choice` | `|||` |
| `=> F[ Either[A, A], A]` | `codiagonal` |

##### Arrow

| Type           | Method Name  | Symbol |
|----------------|--------------|--------------|
| `(A => B) => F[A, B]`  | `lift`    |
| `F[A,B] => F[C,D] => F[(A,C), (B,D)]` | `split` | `***` |
| `F[A,B] => F[A,C] => F[A, (B,C)]` | `merge` | `&&&` |

##### ArrowChoice

| Type  | Method Name | Symbol |
--------|-------------|--------------|
| `F[A,B] => F[C,D] => F[Either[A, C], Either[B, D]]` | `choose` | `+++`
| `F[A,B] => F[Either[A, C], Either[B, C]]` | `left`  |
| `F[A,B] => F[Either[C, A], Either[C, B]]` | `right` |