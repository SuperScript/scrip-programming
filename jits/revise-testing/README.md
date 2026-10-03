---
id: revise-testing
template: templates/mvp
phase: verify
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

No entry remains.
<!-- instruct: decisions -->
## Decisions

<!-- instruct: verify -->
## Verify

### Round 1

- F1 [open] unobserved. Proof check: a user reports applying the skill out of band to a project that does not use a tests/output versus tests/expected basis, and reports the result. Check: read jits/revise-testing/README.md lines 32-34, jits/revise-testing/relax-the-testing-convention/README.md, and plugin/skills/scrip-programming/SKILL.md. None holds that report. The skill file is skill text, not an application report.

Verdict: can't tell
