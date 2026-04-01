---
name: scala-guidelines
description: >
  Scala coding guidelines.
  Use when writing, reviewing, or refactoring Scala code. 
  Invoke with /scala-guidelines.
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

### Type Safety (HIGH)
- [`type-option-semantics`](rules/type-option-semantics.md) - `Option[T]` usage rules

## How to Use

This skill provides rule identifiers for quick reference. When generating or reviewing Rust code:

1. **Check relevant category** based on task type
2. **Apply rules** with matching prefix
3. **Prioritize** CRITICAL > HIGH > MEDIUM > LOW
4. **Read rule files** in `rules/` for detailed examples

### Rule Application by Task

| Task | Primary Categories |
|------|-------------------|
| New function | `type-`, `err-` |
| New struct/API | `api-`, `type-`, `doc-` |
| Async code | `async-`, `own-` |
| Error handling | `err-`, `api-` |
| Code review | `anti-`, `type-`, `lint-` |