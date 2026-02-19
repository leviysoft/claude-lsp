# CLAUDE SCALA LSP

## HOT TO DO THINGS

First of all, install [coursier](https://get-coursier.io/docs/cli-installation)

Then:

    cs install metals

Don't forget to add `export ENABLE_LSP_TOOL=1`

    /plugin marketplace add leviysoft/claude-scala-lsp
    /plugin install claude-scala-lsp@leviysoft

Coursier does not perform automatical updates, so periodically call

    cs update