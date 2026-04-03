---
name: scala-discipline
description: >
  This skill should be used when the user asks to "write Scala code", "review Scala code",
  "refactor Scala code", "implement a Scala API", "design a Scala domain model", or mentions
  Scala-specific topics such as refinement types, enumerations, strongly typed IDs, Option semantics,
  Cats, Chimney, or compile-time validation. Apply whenever generating, reviewing, or refactoring Scala 2
  or Scala 3 code.
---

# Scala Best Practices

Comprehensive guide for writing high-quality Scala code. Prioritized by impact to guide LLMs in code generation and refactoring.

## When to Apply

Reference these guidelines when:
- Writing new Scala functions, classes, or modules
- Implementing error handling or async code
- Designing public APIs for libraries
- Reviewing code
- Refactoring existing Scala code

## Quick Reference

### API Design (HIGH)
- [`api-refinement-scala-2`](rules/api-refinement-scala-2.md) - when a field has known value constraints, encode them in the type with `refined` (Scala 2)
- [`api-refinement-scala-3`](rules/api-refinement-scala-3.md) - when a field has known value constraints, encode them in the type with `iron` (Scala 3)
- [`api-enumerations`](rules/api-enumerations.md) - avoid `scala.Enumeration`; use `enumeratum` (Scala 2) or native `enum` (Scala 3)

### Type Safety (HIGH)
- [`type-option-semantics`](rules/type-option-semantics.md) - use `Option[T]` only for genuinely optional fields without defaults; prefer default parameter values or sealed traits otherwise
- [`type-literally`](rules/type-literally.md) - use the `literally` library to turn hardcoded string values (date patterns, regexes, URIs, etc.) into compile-time validated literals
- [`type-nameof`](rules/type-nameof.md) - use `scala-nameof` whenever a type or property name is needed as a string constant
- [`type-strongly-typed-ids-scala-2`](rules/type-strongly-typed-ids-scala-2.md) - all entity identifiers must be strongly typed (Scala 2, `tagging`)
- [`type-strongly-typed-ids-scala-3`](rules/type-strongly-typed-ids-scala-3.md) - all entity identifiers must be strongly typed (Scala 3, `neotype`)

### Domain Modeling (MEDIUM)
- [`dom-prove-inductive-properties`](rules/dom-prove-inductive-properties.md) - when a domain property holds inductively over a type's fields (e.g. structural projection), encode it as a derived typeclass for a compile-time proof
- [`dom-sized-collections`](rules/dom-sized-collections.md) - use sized collections when sizes of two or more collections are related by a compile-time constraint (Scala 2: `typequux`; Scala 3: `tightbound`)

### Patterns (MEDIUM)
- [`patterns-chimney`](rules/patterns-chimney.md) - never share case classes across application layers; use Chimney for boilerplate-free conversions between per-layer models
- [`patterns-cats`](rules/patterns-cats.md) - Cats type class methods over hand-rolled equivalents

## How to Use

This skill provides rule identifiers for quick reference. When generating or reviewing Scala code:

1. **Check relevant category** based on task type
2. **Apply rules** with matching prefix
3. **Prioritize** CRITICAL > HIGH > MEDIUM > LOW
4. **Read rule files** in `rules/` for detailed examples

### Code Review Constraints

When reviewing or assessing code, **always include the rule identifier** for each recommendation. Only raise discipline-related recommendations that map to a specific rule. Bug reports and correctness findings are always in scope regardless of rule coverage.

### Rule Application by Task

| Task | Primary Categories |
|------|-------------------|
| New function | `api-`, `type-` |
| New class/case class/trait/sealed trait/API | `api-`, `type-`, `dom-` |
| Cross-layer data transfer | `patterns-` |
| Effectful / async code | `patterns-` |
| Code review | `api-`, `type-`, `dom-`, `patterns-` |