---
id: revise-testing
template: templates/mvp
phase: sequence
created: 2026-10-03
updated: 2026-10-03
status: wip
children:
  - relax-the-testing-convention
description: "revise-testing"
steps:
  - intent
  - value
  - sequence
  - verify
---

# revise-testing

<!-- instruct: intent -->
## Intent

**Why:** The testing convention imposes an overly rigid requirement, and that requirement needs to be relaxed.

**Payoff:** The skill applies cleanly to a broader set of projects.

**Who:** Any user of this repo uses the result in the context of their project.

<!-- instruct: value -->
## Proof of Value

- A user reports applying the skill to a project that does not use the current tests/output versus tests/expected basis for tests. The user applies the skill out of band and reports the result.

No value: the application to a project with different testing semantics fails.

<!-- instruct: sequence -->
## Sequence

1. slice — Relax the testing convention. Change the skill so it does not require a tests/output versus tests/expected basis, and so it applies to a project with different testing semantics. `revise-testing/relax-the-testing-convention` done.

<!-- instruct: decisions -->
## Decisions

<!-- instruct: verify -->
## Verify
