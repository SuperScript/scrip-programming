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

**Apply the shell-programming skill.**

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
standard input. Pipe a template containing `#include` directives to get
back fully resolved code:

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
- **Using a component (default):** Pipe a template with `#include`
  lines to `scrip code` for initial assembly. Edit the target file
  directly for subsequent changes unless the set of includes changes.
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

## Error Handling

Read the library source for each error function before use:

| Function | Shell | Awk |
|----------|-------|-----|
| shout | shout.sh | shout.awk |
| barf | barf.sh | barf.awk |
| usage | usage.sh | — |
| safe | safe.sh | — |
| catch | catch.sh | — |

### Error Hierarchy

```
shout    → stderr message (continues)
usage    → stderr + exit 100 (permanent error)
barf     → stderr + exit 111 (temporary error)
safe     → run command, barf on failure
catch    → run command, barf if stderr matches pattern
```

### Protecting a Pipeline with catch

In a pipeline, only the exit status of the last command is visible.
Wrap each component with `safe` so failures produce the `barf` error
prefix on stderr, then use `catch` on the whole pipeline to detect it:

```sh
_do_work() {
  safe producer "$1" | safe filter | safe consumer
}

do_work() {
  catch "$0: fatal:" _do_work "$@"
}
```

`catch` scans stderr for the regex `$0: fatal:` (the prefix `barf`
produces). If any `safe`-wrapped component fails, `catch` exits 111.

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

**Initial assembly:** Use `scrip code` to resolve `#include` directives
into a standalone script. Pipe a template or pass a source file:

```sh
printf '#include "shout.sh"\n#include "barf.sh"\n' | scrip code
```

Place the output into the target file.

**Subsequent edits:** Edit the target file directly. Rerun `scrip code`
only when the set of included library components changes. When edits
affect only the program logic and not the includes, direct edits are
preferred — they avoid overwriting prior modifications to the resolved
code.

### Borrowing into lib/ (Alternative)

Use borrowing only when the project will maintain its own scrip-style
structure — multiple programs sharing modules, ongoing development with
a `src/` → `bin/` build step, or when it is important to keep the
`#include` statements and separate component files. **Ask the user to confirm before choosing this approach.**

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

## Common Mistakes

```sh
# Wrong error handler — use the vocabulary, not raw echo/exit
echo "error" >&2; exit 1             # WRONG
barf "missing file"                   # CORRECT

# Missing #include — every function must be included before use
barf "error"                          # WRONG - barf not defined
#include "barf.sh"                    # CORRECT - include first

# Manual exit — never combine shout with a manual exit
shout "not found"; exit 111           # WRONG
barf "not found"                      # CORRECT

# Bare commands — wrap in safe when failure should be fatal
mkdir -p "${dir}"                     # WRONG - silent failure
safe mkdir -p "${dir}"                # CORRECT

# Intermediate source files — pipe directly with inline use
scrip code src/program.sh > bin/prog  # WRONG - extra file to maintain
cat <<'SRC' | scrip code > bin/prog   # CORRECT - pipe directly
#!/bin/sh
#include "safe.sh"
SRC
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
