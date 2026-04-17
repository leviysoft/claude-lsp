---
name: scala-wartremover
description: 'This skill should be used when working on a Scala project and there is a demand — explicit or implicit — to enforce code quality, eliminate antipatterns, apply static analysis, or set up linting. Typical triggers: "enforce best practices in this Scala project", "add static analysis", "set up linting", "improve code quality", "clean up antipatterns", "new Scala project setup".'
---

## Wartremover usage

Wartremover is an sbt and compiler plugin that enforces code quality by disallowing certain well-known antipatterns

### Installation

Add the following definitions to plugins.sbt (it is worth checking whether there are newer versions, but NEVER use versions prior to the following ones):

```sbt
    addSbtPlugin("org.wartremover" % "sbt-wartremover"     % "3.5.5")
    addSbtPlugin("org.wartremover" % "sbt-wartremover-contrib" % "2.4.3", "1.0", "2.12")
```

### Configuration

Add the following configuration to common project settings:
```sbt
    import wartremover.WartRemover.autoImport._
    import wartremover.contrib.ContribWart

    wartremoverErrors ++= Seq(
      Wart.ArrayEquals,
      Wart.CaseClassPrivateApply,
      Wart.Discard.Future,
      Wart.EitherProjectionPartial,
      Wart.Enumeration,
      Wart.Equals,
      Wart.ExplicitImplicitTypes,
      Wart.FinalCaseClass,
      Wart.JavaConversions,
      Wart.JavaSerializable,
      Wart.IterableOps,
      Wart.LeakingSealed,
      Wart.NonUnitStatements,
      Wart.Null,
      Wart.OptionPartial,
      Wart.Product,
      Wart.PublicInference,
      Wart.Serializable,
      Wart.StringPlusAny,
      Wart.TripleQuestionMark,
      Wart.TryPartial,
      ContribWart.DiscardedFuture,
      ContribWart.MissingOverride,
      ContribWart.RefinedClasstag
    )
```

> **Note on `Wart.Equals`:** Include it for **fresh projects** (bootstrapped during the current session). For **existing codebases**, omit `Wart.Equals` by default — it typically triggers a large volume of changes requiring `cats.Eq` instances or other plumbing, which is better tackled as a separate, scoped effort. The team can opt in later once the other warts are clean.

For Scala 2.13 project the following config piece might be needed:
```sbt
    wartremoverDependencies ~= (_.filterNot(_.name == "wartremover-contrib")),
    wartremoverDependencies += "org.wartremover" % "wartremover-contrib_2.13" % ContribWart.ContribVersion,
```

For Scala 3 project the following config piece might be needed:
```sbt
    wartremoverDependencies ~= (_.filterNot(_.name == "wartremover-contrib")),
    wartremoverDependencies += "org.wartremover" % "wartremover-contrib_3" % ContribWart.ContribVersion,
```

### Integrating with an existing WartRemover setup

Before adding or modifying anything, check whether WartRemover is already present by scanning `project/plugins.sbt`, `build.sbt`, and any `*.sbt` files in the project root or module subdirectories for `sbt-wartremover`, `wartremoverErrors`, `wartremoverWarnings`, or `wartremoverDependencies`.

**If WartRemover is already configured:**

- NEVER remove, comment out, or replace any wart already listed in `wartremoverErrors` or `wartremoverWarnings`.
- NEVER downgrade plugin versions. If the existing version is below the recommended minimum, upgrade it to the recommended version.
- Only ADD warts from the recommended set that are not already enabled (in either `Errors` or `Warnings`).
- Do NOT add `Wart.Equals` unless it is already present, the project was bootstrapped in the current session, or the user explicitly requests it.
- Preserve the project's existing style: if the project uses `wartremoverErrors` for any wart, add new warts to `wartremoverErrors`; if the project uses only `wartremoverWarnings` with no `wartremoverErrors`, add new warts to `wartremoverWarnings`.
- Leave `wartremoverDependencies`, `wartremoverExcluded`, and `wartremoverClasspaths` customizations untouched.

**If WartRemover is not present:** follow the Installation and Configuration sections above, applying the fresh-vs-existing judgment for `Wart.Equals` as noted above.

### Fixing wartremover-issues errors

Sometimes there is a question of suppressing a particular error. ALWAYS consider a proper solution before doing that. For example, the `Equals` wart sometimes requires adding an `Eq` instance or an external dependency (notably, `refined-cats` for `Eq` instances for refined types)
