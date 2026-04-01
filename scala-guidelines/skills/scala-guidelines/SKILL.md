---
name: scala-guidelines
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
- [`api-refinement-scala-2`](rules/api-refinement-scala-2.md) - refinement types for constrained values (Scala 2, `refined`)
- [`api-refinement-scala-3`](rules/api-refinement-scala-3.md) - refinement types for constrained values (Scala 3, `iron`)
- [`api-enumerations-scala-2`](rules/api-enumerations-scala-2.md) - use `enumeratum` instead of `scala.Enumeration` (Scala 2)

### Type Safety (HIGH)
- [`type-option-semantics`](rules/type-option-semantics.md) - `Option[T]` usage rules
- [`type-literally`](rules/type-literally.md) - compile-time validated string literals for constrained types
- [`type-nameof`](rules/type-nameof.md) - use `scala-nameof` whenever a type or property name is needed as a string constant
- [`type-strongly-typed-ids`](rules/type-strongly-typed-ids.md) - all entity identifiers must be strongly typed

### Domain Modeling (MEDIUM)
- [`dom-prove-properties`](rules/dom-prove-properties.md) - prove domain properties (e.g. structural projection) at compile time via typeclasses

### Patterns (MEDIUM)
- [`patterns-chimney`](rules/patterns-chimney.md) - layer separation and chimney-based conversions
- [`patterns-cats`](rules/patterns-cats.md) - Cats type class methods over hand-rolled equivalents

## How to Use

This skill provides rule identifiers for quick reference. When generating or reviewing Scala code:

1. **Check relevant category** based on task type
2. **Apply rules** with matching prefix
3. **Prioritize** CRITICAL > HIGH > MEDIUM > LOW
4. **Read rule files** in `rules/` for detailed examples

### Rule Application by Task

| Task | Primary Categories |
|------|-------------------|
| New function | `api-`, `type-` |
| New class/case class/trait/sealed trait/API | `api-`, `type-`, `dom-` |
| Cross-layer data transfer | `patterns-` |
| Effectful / async code | `patterns-` |
| Code review | `api-`, `type-`, `dom-`, `patterns-` |