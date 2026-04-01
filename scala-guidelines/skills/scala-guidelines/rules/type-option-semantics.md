# type-option-semantics

> `Option[T]` type should be used in models ONLY if the field is optional according to requirements and DO NOT have a default value

## Why It Matters

The core principle is to make states that are invalid according to business rules **unrepresentable** in the model — as far as the Scala type system allows. When `Option[T]` is applied to a field that is always required or has a default, the type system permits constructing objects that violate business invariants, forcing unnecessary defensive handling (`getOrElse`, pattern matching) for a case that should never exist. Conversely, omitting `Option` for a genuinely absent-able field hides that fact and pushes the "missing" state into unsafe conventions like sentinel values.

## Details
- When there is a default value for a field (regardless of whether it is a newly introduced field or pre-existing one) it is preferable to use default parameter values
- If a field is required in one case but not in another, it is better to use sealed traits.

## Bad

```scala
case class Payment(
  amount: Double,
  currency: String,
  status: Option[String],
  transactionId: Option[String],
  failureReason: Option[String]
)
```

## Good
```scala
sealed trait Payment

case class SuccessfulPayment(amount: Double, currency: String, transactionId: String) extends Payment
case class FailedPayment(amount: Double, currency: String, failureReason: String) extends Payment
```