---
name: Maven Source Fetch
description: This skill should be used when the user or an agent needs to "fetch sources for a Maven dependency", "download source JARs", "get sources for a library", "find source JAR path", "attach sources for a dependency", "download sources via Coursier", or "locate source JARs for debugging".
version: 0.1.0
---

# Maven Source JAR Fetching via Coursier

Use the `cs fetch --sources` command to download source JARs for Maven artifacts and obtain their local cache paths.

## Prerequisites

`cs` (Coursier) must be installed and on `$PATH`. Verify with `cs --version`.

## Command Syntax

```bash
cs fetch --sources <groupId>:<artifactId>:<version>
```

A fully-qualified coordinate (all three parts) is required — unlike `cs complete-dep`, partial coordinates are not supported here.

## Basic Usage

### Fetch sources for a single artifact

```bash
cs fetch --sources org.typelevel:cats-core_3:2.10.0
```

Output is one or more absolute paths to JAR files in the local Coursier cache, one per line. The source JAR ends with `-sources.jar`.

### Fetch sources and filter to the source JAR only

```bash
cs fetch --sources org.typelevel:cats-core_3:2.10.0 | grep -- '-sources\.jar$'
```

Without this filter the output may also include the main JAR or other artifacts.

### Fetch sources for multiple artifacts at once

```bash
cs fetch --sources \
  org.typelevel:cats-core_3:2.10.0 \
  io.circe:circe-core_3:0.14.6
```

Each artifact's source JAR path appears on its own line.

## Scala Cross-Built Artifacts

Scala libraries encode the Scala version in the artifactId. Apply the same suffix rules as when resolving versions:

| Scala version | Suffix  | Example                   |
|---------------|---------|---------------------------|
| Scala 2.12    | `_2.12` | `cats-core_2.12`          |
| Scala 2.13    | `_2.13` | `cats-core_2.13`          |
| Scala 3       | `_3`    | `cats-core_3`             |

Java-only artifacts have no suffix (e.g., `com.zaxxer:HikariCP:5.1.0`).

## Workflow for Fetching Sources

When asked to fetch or locate sources for a dependency:

1. Confirm the full coordinate (`groupId:artifactId:version`). If the version is unknown, use `cs complete-dep` (from the maven-artifact-lookup skill) to resolve it first.
2. Run `cs fetch --sources <coordinate>`.
3. Filter the output with `grep -- '-sources\.jar$'` to isolate the source JAR path.
4. Report the absolute path. The caller can use this path to attach sources in an IDE, inspect the JAR contents, or pass it to other tools.

### Inspecting source JAR contents

```bash
jar tf "$(cs fetch --sources org.typelevel:cats-core_3:2.10.0 | grep -- '-sources\.jar$')"
```

### Reading a specific source file from the JAR

```bash
jar xf "$(cs fetch --sources org.typelevel:cats-core_3:2.10.0 | grep -- '-sources\.jar$')" \
  org/typelevel/cats/Monad.scala
cat org/typelevel/cats/Monad.scala
```

## Error Handling

- **No output / empty result**: the artifact or version does not exist on Maven Central. Verify coordinates with `cs complete-dep`.
- **`Not found` error**: same cause — check groupId, artifactId, and version spelling.
- **Source JAR absent from output**: the publisher did not upload sources for that release. This is uncommon for major libraries but does occur for older versions or proprietary artifacts.
