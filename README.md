# CLAUDE Plugins

## Adding this marketplace

    /plugin marketplace add leviysoft/claude-plugins

## Utility Plugins

### prefer-ast-grep

Redirects structural code searches (functions, classes, types) from the `Grep` tool to ast-grep. Requires both the ast-grep tool and the ast-grep skill:

    /plugin marketplace add ast-grep/agent-skill
    /plugin install ast-grep

    /plugin install prefer-ast-grep@leviysoft

### enforce-native-search

Blocks `grep`/`rg` and `find` in Bash commands, enforcing ast-grep and the Glob tool instead. Requires `jq` to be installed.

    /plugin install enforce-native-search@leviysoft

### prefer-jq

Blocks use of Python for JSON processing in Bash commands, enforcing `jq` instead.

    /plugin install prefer-jq@leviysoft

Requires `jq` to be installed.

### maven-artifact-lookup

Find the latest version of a Maven artifact, list available versions, or check artifact existence — all from the command line via Coursier.

    /plugin install maven-artifact-lookup@leviysoft

Requires [Coursier](https://get-coursier.io/docs/cli-installation) (`cs`) to be installed.

## LSP Plugins

### HOW TO SCALA

First of all, install [coursier](https://get-coursier.io/docs/cli-installation)

Then:

    cs install metals

Don't forget to add `export ENABLE_LSP_TOOL=1`

    /plugin install scala-lsp@leviysoft

Coursier does not perform automatical updates, so periodically call

    cs update

### HOW TO HASKELL

Install [haskell-language-server](https://haskell-language-server.readthedocs.io/en/latest/installation.html)

Don't forget to add `export ENABLE_LSP_TOOL=1`

    /plugin install haskell-lsp@leviysoft

### HOW TO IDRIS2

First of all, install [pack](https://github.com/stefan-hoeck/idris2-pack/blob/main/INSTALL.md)

Then:

    pack install-app idris2-lsp

Don't forget to add `export ENABLE_LSP_TOOL=1`

    /plugin install idris2-lsp@leviysoft

### HOW TO F#

Install [FsAutoComplete](https://github.com/ionide/FsAutoComplete):

    dotnet tool install --global fsautocomplete

Don't forget to add `export ENABLE_LSP_TOOL=1`

    /plugin install fsharp-lsp@leviysoft

## Scala Plugins

### scala-sanitar

Guidance for resolving common Scala compilation errors — non-exhaustive matches, discarded values (including Futures), and references to uninitialized values.

    /plugin install scala-sanitar@leviysoft
