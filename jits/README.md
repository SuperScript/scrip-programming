---
id: .
template: templates/project
phase: done
created: 2026-10-03
updated: 2026-10-03
status: wip
children:
  - revise-testing
description: "Tools for scrip programming"
steps:
  - intent
  - approach
defaults:
  template: templates/project
---

# Tools for scrip programming

<!-- instruct: approach -->
## Approach

The project accumulates programming components that support the scrip style in shell, awk, and Makefile. A skill in the plugin shows an AI how to use those components in the local style.

The root has no fixed phase list. Stand-alone child work items extend and refine the library. The user creates each child on demand. A child adds, removes, refactors, iterates on, and updates components, tests, and skills.

This record directs that work. It does not add, remove, refactor, iterate on, or update components, tests, or skills.

<!-- instruct: decisions -->
## Decisions

- approach: Adding, removing, refactoring, iterating on, and updating components, tests, and skills is out of scope for this root record. Basis: the owner assigned that work to child items created on demand, and this record only directs.
