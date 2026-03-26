---
name: scrip-programming:scrip-programming
description: >-
  This skill should be used when the user asks to "create a scrip program",
  "use scrip components", "write a program with scrip", "add a scrip include",
  "borrow a scrip component", or when creating or updating programs that use
  scrip shell (.sh), make (.mk), or awk (.awk) components, the scrip error
  handling vocabulary (shout, barf, usage, safe, catch), or the scrip #include
  system.
disable-model-invocation: false
user-invocable: true
allowed-tools: Read, Grep, Glob, Bash(scrip help *), Bash(scrip docs *), Bash(scrip deps *), Bash(scrip code *), Bash(scrip list *), Bash(scrip path *)
---

# Shell Programming for scrip

**First read the shell-programming skill if you haven't already.**

## The `scrip` program

The `scrip` command-line tool manages scrip library files and builds programs
from source. Use it throughout the session.

**First step:** run `scrip path` to see the library search path, then
`scrip list` to see available modules.

### Subcommands

| Command | Purpose |
|---------|---------|
| `scrip path` | Print the library search path (colon-separated directories) |
| `scrip list [regex]` | List library files, optionally filtered by regex on basename |
| `scrip deps [file...]` | List all included dependencies in order of encounter |
| `scrip code [file...]` | Print source with all `#include` directives resolved |
| `scrip borrow destdir file...` | Copy included dependencies into a project directory |
| `scrip prog script file...` | Build an executable from source (mode 0755) |
| `scrip docs [file...]` | Print `#_#` help documentation from files |
| `scrip help` | Print help for scrip subcommands |

When no file arguments are given, `scrip code` and `scrip deps` read from
standard input. This means you can pipe a template containing `#include`
directives and get back fully resolved code:

    printf '#include "shout.sh"\n#include "barf.sh"\n' | scrip code

This is useful for creating standalone programs that bundle code from the
scrip libraries.

### When to use each

- **Starting a session:** `scrip path` then `scrip list` to discover
  what library modules are available
- **Understanding a module:** Read the file directly from the path
  returned by `scrip list`, or use `scrip docs file` for its help text
- **Writing new code:** `scrip deps file` to check what a source file
  already includes; `scrip code file` to see fully resolved source
- **Using a component (default):** Use `scrip code file` to produce
  fully resolved source, then place the relevant code into the target
  file. Or pipe a template with `#include` lines to `scrip code` to
  produce a standalone script from scratch.
- **Borrowing (alternative):** `scrip borrow lib/ file` to copy a
  library module and its dependencies into the project's `lib/`
  directory — only when the project will maintain its own scrip-style
  structure with `#include` statements and separate component files.
  **Ask the user to confirm before choosing this approach.**
- **Building:** `scrip prog output source` to build an executable from
  a source file (only relevant to borrowed/scrip-style projects)

Always prefer reading actual library source over relying on examples in
this document. The library is the authoritative reference.

### Component Types

The scrip library contains components in three languages, each
identified by its file extension:

| Extension | Language | Examples |
|-----------|----------|----------|
| `.sh` | POSIX shell | shout.sh, barf.sh, safe.sh, pipeline.sh |
| `.mk` | make | help.mk, test.mk, needvar.mk, tex-pdf.mk |
| `.awk` | awk | shout.awk, barf.awk |

All three types use the same `#include` system and `#_#` documentation
convention. Some functions have parallel implementations across
languages (e.g. shout and barf exist as both `.sh` and `.awk`
components). Use the extension matching the language of the file you
are working in.

## Overview

The scrip project provides infrastructure for building shell programs using a consistent error handling vocabulary (shout/barf/usage/safe/catch) and modular composition via #include. This skill documents scrip-specific conventions and patterns.

**Core principle:** Quote everything (see shell-programming), fail loudly, compose from lib/ modules.

## When NOT to Use

When asked explicitly to create a Bash script, or when writing shell scripts that don't use scrip infrastructure.

## The Iron Laws
- Follow the shell-programming skill for POSIX compliance and quoting
- Use project error handling - shout/barf/usage/safe/catch
- Document with #_# for help extraction
- Test with output diff against tests/expected
- Include dependencies via #include

## Error Handling Patterns

The project uses a consistent error handling vocabulary:

### shout - Print to stderr

```sh
#include "shout.sh"
shout() { printf '%s\n' "$0: $*" >&2; }

# Usage
shout "warning: file not found"
```

### barf - Fatal error (exit 111)
Exit 111 for temporary errors.

```sh
#include "barf.sh"
barf() { shout "fatal: $*"; exit 111; }

# Usage
test -f "${file}" || barf "missing required file: ${file}"
```

### usage - Usage error (exit 100)
Exit 100 for permanent errors.

```sh
#include "usage.sh"
usage() { shout "usage: $*"; exit 100; }

# Usage
test $# -gt 1 || usage "$0 sep prog [sep prog ...]"
```

### safe - Execute or barf

```sh
#include "safe.sh"
safe() { "$@" || barf "cannot $*"; }

# Usage
safe mkdir -p "$dir"
safe mv "${temp}" "${output}"
```

### catch - Fail if stderr matches regex
The `catch` function can run a command that executes a pipeline and detect errors in any pipeline component that prints a known pattern to standard error, such as commands called with `safe`.

```sh
#include "catch.sh"

# Usage - exit 111 if command writes pattern to stderr
catch "error:" some_command arg1 arg2
```

### Error Hierarchy

```
shout    → stderr message (continues)
usage    → stderr + exit 100 (permanent error)
barf     → stderr + exit 111 (temporary error)
safe     → run command, barf on failure
catch    → run command, barf if stderr matches pattern
```

## Function Design

### Naming Conventions

**Library functions:** Descriptive names
```sh
atomic_to()
have_args()
pipewith()
```

**Command routing:** `do_` prefix for subcommands
```sh
do_help()
do_code()
do_deps()
do_run()
```

**Internal helpers:** `_` prefix
```sh
_mode()
```

### do_ Prefix Pattern

Programs use `do_` prefix for subcommand routing:

```sh
#!/bin/sh
#include "usage.sh"
#include "do_help.sh"

do_foo() {
  # implementation
}

do_bar() {
  # implementation
}

# Route to do_$1 function
test $# -lt 1 && usage "$0 foo|bar|help [args...]"
"do_$@"
```

The final line `"do_$@"` expands to `"do_foo" arg1 arg2` which calls the function.

### Help Documentation

Use `#_#` prefix for help text extracted by `do_help`:

```sh
#_# help
#_#   Print this helpful message
#_#
do_help() {
  sed -n 's/^#_#/ /p' "$0"
}

#_# code file...
#_#   Print the code in files, resolving includes
#_#
do_code() {
  _mode code "$@"
}

#_# pipeline sep prog [sep prog ...]
#_#
```

Help is extracted by `sed -n 's/^#_#/ /p'` - strips `#_#` prefix and prints.

Always end a block of help lines with a line containing only `#_#`.

### Function Comments

Use `#` for function comments that should not appear in `help` output. Use `#_#` for function comments that should appear in `help` output.

## Code Organization

### Using Components Inline (Preferred)

The normal way to use scrip program components is inline: resolve
`#include` directives with `scrip code`, then place the output into
the target file. This avoids creating a `lib/` directory and build
step.

**Workflow:**

1. Find the component: `scrip list` then read its source
2. Produce resolved code: `scrip code file` writes the file with all
   includes rendered to stdout
3. Place the output into the target file (which usually already exists)

**From a template:**

```sh
printf '#include "shout.sh"\n#include "barf.sh"\n' | scrip code
```

This produces a standalone block of code that can be inserted into any
shell script.

### Borrowing into lib/ (Alternative)

Use borrowing only when the project will maintain its own scrip-style
structure — multiple programs sharing modules, ongoing development with
a `src/` → `bin/` build step, or when it is important to keep the
`#include` statements and separate component files. **If you think
borrowing is the appropriate choice, ask the user to confirm before
proceeding.**

#### When to Create lib/ Modules

Create new component files in `lib/` (`.sh`, `.mk`, or `.awk`
depending on the language) for:

1. **Reusable functions** - Used by multiple programs
2. **Composable utilities** - Small, focused purpose
3. **Error handling** - `shout`, `barf`, `usage`, `safe`, `catch`
4. **Command patterns** - `do_help`, `do_run`, `do_xrun`

**Each module does ONE thing:**
```
lib/barf.sh          - Fatal error reporting
lib/safe.sh          - Safe command execution
lib/atomic_to.sh     - Atomic file writes
lib/pipewith.sh      - Dynamic pipeline construction
```

### Include Pattern

**Source files** use `#include "filename"`, with the extension matching
the language of the including file:

```sh
#!/bin/sh
#include "usage.sh"
#include "do_help.sh"
#include "pipeline.sh"
```

```make
#include "help.mk"
#include "test.mk"
```

**Include resolution:**
- Searches `SCRIP_PATH` (colon-separated directories)
- Falls back to `../lib` relative to script location
- Each file included only once (deduplication)
- Recursive - includes can contain includes
- Absolute paths or `./` prefix bypass `SCRIP_PATH`

A project that uses borrowing can copy files located along `SCRIP_PATH`
into a `lib/` subdirectory with `scrip borrow lib/ file`.

### Directory Structure (Borrowed Projects)

This structure applies to scrip-style projects that use borrowing.
Inline usage does not require this layout.

```
lib/           # Reusable modules (included via #include)
src/           # Source files (contain #include directives)
bin/           # Generated executables (built from src/)
tests/         # Test suite
```

## Testing

### Test Structure

```
tests/
  run           # Executable that runs all tests
  expected      # Expected output
  output        # Actual output (generated)
  basedir/      # Test fixtures
```

### Adding Tests

**1. Add test case to `tests/run`:**

```sh
#!/bin/sh
# Test cases print to stdout
echo "==== Test description ===="
./bin/program arg1 arg2
echo ""
```

**2. Run and capture expected output:**

```sh
make build
tests/run > tests/expected
```

**3. Verify tests pass:**

```sh
make tests
# Runs: tests/run > tests/output && diff tests/output tests/expected
```

### Test Principles

- **Output-based testing** - Compare stdout to expected
- **Self-configured** - Tests include setup
- **Grouped Appropriately** - Neighboring tests may share config
- **Deterministic** - Same input always produces same output
- **Documented** - Echo description before each test

**Example from tests/run:**

```sh
echo "==== scrip deps ===="
bin/scrip deps lib/scrip.sh
echo ""

echo "==== scrip code ===="
bin/scrip code src/pipeline.sh | head -1
echo ""
```

## File Names

Library components use the file extension to identify their language:
`.sh` for shell, `.mk` for make, `.awk` for awk. The extension tells
both the `#include` system and the reader which language the component
is written in.

Top-level command-line programs never include an extension, because
that is an implementation detail and should not be exposed in the
command-line interface.

## Real-World Examples

### Atomic File Write

```sh
#include "atomic_to.sh"

# Write output from program to path atomically
atomic_to() {
  local output="$1"
  shift
  local temp="$(mktemp "${output}.XXXXXX")"
  "$@" > "${temp}" && mv "${temp}" "${output}" || {
    local e=$?
    rm -f "${temp}"
    exit $e
  }
}

# Usage
atomic_to "output.txt" command args
```

### Pipeline Construction

```sh
#include "pipewith.sh"

# Build dynamic pipeline with custom prefix
pipewith_cmd() {
  local sep="$2"
  shift 2

  local cmd=''
  local i=3
  local p='"$1"'
  for a in "$@"
  do
    if test "$a" = "${sep}"
    then
      cmd="${cmd} |"
      p='"$1"'
    else
      cmd="${cmd} ${p} \"\${$i}\""
      p=''
    fi
    i=$(($i + 1))
  done

  printf '%s\n' "${cmd}"
}

pipewith() {
  eval "$(pipewith_cmd "$@")"
}
```

### Argument Validation

```sh
#include "usage.sh"
#include "have_args.sh"

# Return 0 if args has at least n entries
have_args() {
  test $# -ge 1 || usage "have_args count [args...]"
  test "$1" -lt $#
  return $?
}

# Usage in functions
do_process() {
  have_args 2 "$@" || usage "$0 process file1 file2"
  # process files
}
```

## Common Mistakes

### ❌ Wrong Error Handler

```sh
# WRONG - inconsistent error reporting
echo "error: missing file" >&2
exit 1

# CORRECT - use project vocabulary
barf "missing file"
```

### ❌ Missing #include

```sh
# WRONG - barf not defined
barf "error"

# CORRECT - include dependencies
#include "barf.sh"
barf "error"
```

### ❌ Test Without Building

```sh
# WRONG - stale binaries
./bin/program  # might be old version

# CORRECT - build first
make build
./bin/program
```

## Quick Reference

| Pattern | Example | Notes |
|---------|---------|-------|
| Error msg | `shout "warning"` | To stderr, continues |
| Fatal error | `barf "fatal"` | To stderr, exit 111 |
| Usage error | `usage "$0 args"` | To stderr, exit 100 |
| Safe exec | `safe mv "$a" "$b"` | Barf on failure |
| Include | `#include "foo.sh"` | .sh, .mk, .awk — resolved via SCRIP_PATH |
| Help doc | `#_# help text` | Extracted by do_help |
| Subcommand | `do_foo() { ... }` | Called via "do_$@" |
| Lib modules | `lib/foo.sh`, `lib/bar.mk` | Reusable components |
| Programs | `bin/program` | No extension |
| Test diff | `diff tests/output tests/expected` | Verify tests |

## The Bottom Line

**Follow shell-programming for POSIX and quoting. Use scrip's error vocabulary. Prefer inline components via `scrip code`; borrow into lib/ only when the project calls for it.**

This project values:
- Explicit error vocabulary makes failures clear
- Modular composition through #include
- Output-based testing for reliability
