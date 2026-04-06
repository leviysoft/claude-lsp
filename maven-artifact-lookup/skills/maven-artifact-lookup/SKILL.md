---
name: Maven Artifact Lookup
description: This skill should be used when the user or an agent needs to "find the latest version" of a Maven artifact, "check if an artifact exists", "list available versions", "look up a dependency version", "find artifact coordinates", "resolve a dependency version for SBT/Gradle/Maven", or "find the latest Scala 3 version of a library".
version: 1.0.0
---

# Maven Artifact Lookup via Coursier

Use the `cs complete-dep` command to resolve Maven artifact information from the command line without leaving the terminal or querying the web.

## Prerequisites

`cs` (Coursier) must be installed and on `$PATH`. Verify with `cs --version`.

## Command Syntax

```
cs complete-dep <groupId>[:<artifactId>[:<version-prefix>]]
```

The argument is a colon-separated coordinate with 1–3 parts. The number of colons determines what is returned.

## Use Cases

### 1. List groupIds matching a prefix

```bash
cs complete-dep org.typelevel
```

Returns all known groupIds that start with `org.typelevel`.

### 2. List artifacts under a groupId

```bash
cs complete-dep com.zaxxer:
```

The trailing colon is required. Returns all artifact IDs published under that groupId.

### 3. List all available versions of an artifact

```bash
cs complete-dep org.typelevel:cats-core_3:
```

The trailing colon after the artifactId is required. Returns all published versions, oldest to newest. **The last line of output is the latest version.**

### 4. Check if a specific version exists

```bash
cs complete-dep org.typelevel:cats-core_3:2.10.
```

Providing a partial version prefix filters the output. If no lines are returned, the version does not exist. If the exact version appears in the output, it exists.

## Finding the Latest Version

To get the latest version of an artifact:

```bash
cs complete-dep com.example:my-library_3: | tail -1
```

The versions are listed in ascending order; `tail -1` gives the latest.

## Scala Cross-Built Artifacts

Scala libraries are cross-published with the Scala version as a suffix in the artifactId:

| Scala version | Suffix  | Example                    |
|---------------|---------|----------------------------|
| Scala 2.12    | `_2.12` | `cats-core_2.12`           |
| Scala 2.13    | `_2.13` | `cats-core_2.13`           |
| Scala 3       | `_3`    | `cats-core_3`              |

When looking up a Scala library, always append the appropriate suffix. If the Scala version is unknown or unspecified, probe all three suffixes and use whichever returns results — or default to `_3` for modern projects:

```bash
cs complete-dep org.typelevel:cats-core_2.12: | tail -1
cs complete-dep org.typelevel:cats-core_2.13: | tail -1
cs complete-dep org.typelevel:cats-core_3:    | tail -1
```

An empty result means that suffix is not published.

Java-only artifacts have no suffix (e.g., `com.zaxxer:HikariCP:`).

## Checking Artifact Existence

Use Case 4 above checks whether a *specific version* exists. To verify that a `groupId:artifactId` combination exists *at all* (regardless of version):

```bash
cs complete-dep com.example:my-library_3:
```

- If the command returns one or more lines → artifact exists.
- If the command returns no output → artifact does not exist (or groupId is wrong).

## Workflow for Dependency Resolution

When asked to find or pin a dependency version:

1. Run `cs complete-dep <groupId>:<artifactId>_<scalaVersion>:` to list all versions.
2. Use `tail -1` to extract the latest, or filter the list for a specific version series (e.g., `grep '^2\.'`).
3. Report the resolved coordinate in the format appropriate for the build tool (infer from build files present in the project — `build.sbt`, `pom.xml`, `build.gradle`, etc. — or ask the user if unclear):

   - **SBT**: `"groupId" %% "artifactId" % "version"` (or `%` for Java artifacts)
   - **Scala CLI / Coursier**: `groupId::artifactId:version` (or `:` for Java)
   - **Maven**: `<groupId>`, `<artifactId>`, `<version>` elements
   - **Gradle**: `'groupId:artifactId:version'`
