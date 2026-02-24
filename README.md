# shell-programming

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that
provides POSIX shell programming conventions for writing `.sh` scripts.

## What it does

When you create or update a shell script, this plugin activates a skill that
guides Claude toward strict POSIX sh compliance. It enforces:

- **POSIX sh + `local` only** -- no bash, zsh, or ksh extensions
- **Mandatory quoting** of all variable expansions (`"${var}"`, `"$@"`)
- **Consistent formatting** -- 2-space indentation, block keywords on new lines, pipeline continuation style
- **Common mistake avoidance** -- bashisms, unquoted variables, missing braces

## When it activates

The skill triggers when:

- Creating or updating a shell script
- Working with files that have a `.sh` extension
- Working with files that use a `#!/bin/sh` shebang

It does **not** activate when explicitly asked to write a Bash script.

## Installation

Add to your Claude Code configuration:

```sh
claude mcp add-plugin shell-programming -- https://github.com/superscript/shell-programming
```

## License

[BSD 3-Clause](LICENSE)
