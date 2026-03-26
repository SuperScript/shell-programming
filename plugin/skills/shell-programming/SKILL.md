---
name: shell-programming:shell-programming
description: >-
  This skill should be used when the user asks to "create a shell script",
  "write a shell script", "update a shell script", "fix a shell script",
  or when creating or updating files with a `.sh` extension or files that
  invoke `/bin/sh` in the shebang line. Provides POSIX shell conventions
  including quoting rules, variable expansion, and function design. Do not
  use when asked to create a Bash script.
disable-model-invocation: false
user-invocable: true
allowed-tools: Read, Grep, Glob
---

# Shell Programming

## Overview

All shell code should follow strict POSIX Bourne shell syntax plus `local` keyword. Every variable expansion must be quoted. This is the default way to write shell scripts.

**Core principle:** Quote everything, use POSIX sh + `local` only.

## When NOT to Use

When explicitly asked to create a Bash script.

## The Iron Laws
- POSIX sh only (plus `local`) - no bash/zsh/ksh extensions.
- Quote ALL variable expansions, with curly braces around multiletter variable names - "${var}" "${output}" (name does not include "$").
- Use `local` for function-scoped variables.
- Capture `$?` before it changes if you need to use it.
- Indent with two spaces, never tabs.
- Block keywords (do, then, else, done, fi, case, esac) start a new line.
- Short pipelines stay on one line. Break long pipelines with `\` at end of line, `|` starts next line aligned above.
- Braces `{` on same line after `||` or `&&`.

## Quoting Rules

### Always Quote

**All variable expansions must be quoted, with curly braces around multi-character variable names:**

```sh
# ✅ CORRECT
"${var}"
"$@"
"${output}"
"${temp}"
"blahblahblah: $*"
printf '%s\n' "${message}"
"${12}"
for f in "$@"

# ❌ WRONG
"$var"
$@
${output}
printf '%s\n' $message
for f in $@
```

### Even in Tests

```sh
# ✅ CORRECT
test "$a" = "${sep}"
test $# -gt 0
if test -f "$f"
test "$1" -lt $#

# ❌ WRONG
test $a = ${sep}
if test -f $f
```

**Exception:** Numeric comparisons and `$#` don't need quotes:
```sh
test $# -gt 1    # OK - $# is always numeric
test "${count}" -eq 5  # GOOD - quote variables
```

### $@ vs $*

- **`"$@"`** - Each argument as separate word. Use for executing commands:
  ```sh
  run_command() { "$@" || { echo "command failed: $*" >&2; exit 1; }; }
  #               ^^^^ execute command
  ```

- **`"$*"`** - All arguments joined with spaces. Use for messages:
  ```sh
  error() { echo "error: $*" >&2; exit 1; }
  #                      ^^^^ join for message
  ```

### Arithmetic Expansion

Arithmetic expressions don't need quotes (they can't contain spaces):

```sh
i=$(($i + 1))          # No quotes needed
count=$((${count} + 1))  # POSIX compliant
```

The counter variable must be initialized to a numeric value before use in an arithmetic expression.

### Function Arguments

```sh
# ✅ CORRECT - quote "$@"
run_program() {
  "$@"
}

# ✅ CORRECT - quote individual args
write_to() {
  local output="$1"
  shift
  "$@" > "${output}"
}
```

### Command Substitution

```sh
# ✅ CORRECT - quote the substitution
local temp="$(mktemp "${output}.XXXXXX")"
local awk="$(which gawk || which nawk || echo awk)"

# ❌ WRONG
local temp=$(mktemp ${output}.XXXXXX)
```

## Function Design

### Function Structure

```sh
# function_name arg1 arg2
#   Brief description of what it does
function_name() {
  local var1="$1"
  local var2="$2"
  shift 2

  # implementation
  "$@" > "${var1}" || {
    local e=$?
    # cleanup
    exit $e
  }
}
```

**Key patterns:**
- Use `local` for function variables
- Quote all variable expansions
- Capture `$?` before it changes: `local e=$?`
- Use `{ ...; }` for multi-statement conditionals
- Indent with two spaces
- Opening braces `{` on same line after `||` or `&&`

### Formatting Rules

**Indentation:**
- Two spaces per level
- Never use tabs

**Block keywords:**
- `do`, `then`, `else`, `done`, `fi`, `case`, `esac` start a new line

```sh
# ✅ CORRECT
if test -f "${file}"
then
  process "${file}"
else
  echo "not found" >&2
fi

for item in "$@"
do
  process "${item}"
done
```

**Pipeline continuation:**
- Short pipelines stay on one line when clear and concise
- Break with `\` at end of line and `|` starting the next when the pipeline is long, complex, or benefits from separate lines
- When breaking, `|` starts the next line aligned with the line above

```sh
# ✅ Short pipeline - one line
_mode deps "$@" | while read -r f

# ✅ Long or complex pipeline - break for clarity
do_json search --output=summary "$@" \
| jq -c 'foreach .[] as $i (-1; . + 1; $i + {knm_index: .})' \
| sort -u
```

## Common Mistakes

### ❌ Unquoted Variables

```sh
# WRONG
temp=$(mktemp $output.XXXXXX)
if test -f $file
for arg in $@

# CORRECT
temp="$(mktemp "${output}.XXXXXX")"
if test -f "${file}"
for arg in "$@"
```

### ❌ Bashisms

```sh
# WRONG - bash-specific
[[ $x == $y ]]
function foo() { ... }
local -r readonly_var="x"
echo "array: ${arr[@]}"

# CORRECT - POSIX sh
test "$x" = "$y"
foo() { ... }
local readonly_var="x"
# Use separate variables instead of arrays
```

### ❌ Unquoted in Test

```sh
# WRONG
if [ $var = value ]
test $file1 = $file2

# CORRECT
if test "${var}" = "value"
test "${file1}" = "${file2}"
```

### ❌ Missing Braces for Multi-Character Variables

```sh
# WRONG
"$var"
"$output"
"$temp"

# CORRECT
"${var}"
"${output}"
"${temp}"
```

## Quick Reference

| Pattern | Example | Notes |
|---------|---------|-------|
| Quote vars | `"${var}" "$@" "${x}"` | Always quote expansions |
| Numeric test | `test $# -gt 1` | $# doesn't need quotes |
| Numeric values | `i=$(($i + 1))` | Arithmetic doesn't need quotes |
| String test | `test "$a" = "$b"` | Quote both sides |
| Error msg | `echo "error: $*" >&2` | To stderr |
| Exit on error | `exit 1` | Non-zero exit code |
| Local var | `local x="$1"` | Function-local |
| Save exit | `local e=$?` | Capture before it changes |
| Multi-statement | `{ cmd1; cmd2; }` | Grouping for || or && |
| Execute args | `"$@"` | Preserve argument array |
| Join args | `"$*"` | Single string with spaces |

## The Bottom Line

**Quote everything. Use POSIX sh + `local` only.**

This approach values:
- Strict POSIX compliance for portability
- Consistent quoting prevents word-splitting bugs
- Simple, reliable patterns that work everywhere

Follow these conventions and your shell code will be robust, portable, and maintainable.
