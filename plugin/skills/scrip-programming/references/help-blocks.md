# `#_#` help blocks

A `#_#` block states usage, effect, and restrictions. Nothing else.

## Form

The first line is the usage line: the verb path as typed, with `<required>`,
`[optional]`, `{a | b}` for alternatives, and a trailing `...` for repeats.
A dispatcher names its subcommands in braces rather than writing
`<subcommand>`.

Body lines are `#_#` followed by three spaces. The block closes with a line
holding only `#_#`.

Body lines stay at or under 74 columns. The usage line runs as long as the
command surface needs.

No examples, no tables, no indented displays. The body is body lines.

## Content

The body carries the effect, then any restriction. It gives no rationale, no
mechanism, and no contrast with other verbs; a body line does not explain
itself with "since", "so", or "rather than".

One or two body lines. A second line earns its place with a further effect
or a restriction that constrains use. A sentence wraps when it exceeds the
width.

Verbs are imperative and present, with the subject dropped: Print, Read,
Add, Create, Run, Calculate, Bind.

The body reuses the usage line's metavariables — `<model>`, `<target>`,
`<metadata>` — rather than prose nouns standing in for them. It does not
otherwise restate the usage line.

Name the stream and the destination: "on standard input", "to standard
output", "via <target>", "to config".

Say what happens and where it lands. Never say what does not happen.

Defaults and name lists go bare after a colon:

    #_#   Keys provided by default: project, database, template_directory.

Give the shape of an output line as a literal pattern:

    #_#   The format is <varname>: <description>.

Name a config key exactly as config spells it, with the usage line's
metavariable in place: `server.<server>.uri`, `add.template_directory`.

A failure reads "Fail when ...", never naming the error vocabulary that
produces it. State a failure only where it constrains use; most go unstated.

Plain text throughout: no backticks, no markdown.

## Examples

    #_# help
    #_#   Print this helpful message
    #_#

    #_# project new <blueprint> [key=value...]
    #_#   Create a project in the project directory from <blueprint>.
    #_#   Substitute key=value into the blueprint template.
    #_#   Keys provided by default: project, database, template_directory.
    #_#

    #_# vector {make | load | embed | pending} [args...]
    #_#   Calculate and store chunk vectors.
    #_#

    #_# vector make <target> <model>
    #_#   Read chunks from standard input, calculate vectors with <model>,
    #_#   and print vectors to standard output.
    #_#
