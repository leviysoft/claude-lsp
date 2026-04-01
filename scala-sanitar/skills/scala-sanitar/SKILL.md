---
name: scala-sanitar
description: >-
  Use this skill when a Scala compiler error matches one of these specific messages: "match may not be exhaustive", "discarded non-Unit value", or "Reference to uninitialized value". Trigger on any of these exact error types — whether the user asks how to fix them, pastes the compiler output, or asks why the build is failing and the error falls into one of these categories. Do not trigger for other Scala compiler errors not listed here.
---

## Non-exhaustive match

**Error example:**
```
[error] /Path/to/file.scala:38:54: match may not be exhaustive.
[error] It would fail on the following inputs: <some cases>
```

The match expression doesn't cover all possible inputs, so a `MatchError` can occur at runtime.

**Possible solutions (pick the most appropriate):**

- If matching against an open trait hierarchy, make the trait(s) `sealed` so the compiler can verify exhaustiveness
- Add cases covering all inputs listed by the compiler
- Add a wildcard (`_`) case — it must always be at the **end** of the match block
- Replace `filter + map` (or `filter + find`) with `collect` / `collectFirst` using a partial function
- If matching against a Scala `Enumeration`, either rewrite it using [enumeratum](https://github.com/lloydmeta/enumeratum) or add `@unchecked` (not generally recommended)
- In test sources only, `@nowarn("cat=other-match-analysis")` on the test class is acceptable

## Value discarding

**Error example (simple value):**
```
[error] /Path/to/file.scala:119:55: discarded non-Unit value of type akka.actor.Cancellable
```

A non-`Unit` value that is not a `Future` is being discarded. The compiler is being cautious, but this is usually harmless.

**Solutions:**
- Prepend `val _ =` to the expression
- Change the enclosing method's return type away from `Unit` if the value is actually useful
- In tests, a common fix is to change the return type from `Unit` to `Assertion`

---

**Error example (Future):**
```
[error] /Path/to/file.scala:62:41: discarded non-Unit value of type scala.concurrent.Future[..]
```

A discarded `Future` is a concurrency hazard: it executes asynchronously and its result (including failures) is silently lost. The same applies to `FutureErrorOr` and related types.

**Solutions:**
- If the discard is unintentional (e.g. `.map` used instead of `.flatMap`, or a `Future` accidentally placed after `yield`): rewrite using `.flatMap` or yield the unwrapped value
- If the `Future` is intentionally fire-and-forget: add an `.onComplete` block to handle the result (or at minimum log errors)

## Reference to uninitialized value

**Error example:**
```
[error] /Path/to/file.scala:4:11: Reference to uninitialized value {value name}
```

A `val` is read before it has been assigned, typically due to initialization order issues in class/object bodies.

**Solution:** Ensure the highlighted value is fully initialized before any code that accesses it. Common fixes include reordering definitions, using `lazy val`, or refactoring into a companion object.
