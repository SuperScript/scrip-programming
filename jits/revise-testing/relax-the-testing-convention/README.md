---
id: revise-testing/relax-the-testing-convention
template: templates/slice
phase: archive
created: 2026-10-03
updated: 2026-10-03
status: done
children: []
description: "Relax the testing convention"
steps:
  - detail
  - work
---

# Relax the testing convention

<!-- instruct: detail -->
## Detail

**Delivers:** The skill file plugin/skills/scrip-programming/SKILL.md does not require a tests/output versus tests/expected basis. It tells the reader to use the project's own testing semantics.

**Proof:**
- Reading that file shows the Iron Laws, the Testing section, the Quick Reference test row, and the closing values do not require that basis, and that the skill tells the reader to use the project's own testing semantics. The check fails while any of those still require the basis and offer no other.

**Detail:**
- Edit only plugin/skills/scrip-programming/SKILL.md.
- The basis is required by the Iron Law "Test with output diff against tests/expected", the Testing section, the Quick Reference test row, and the closing value "Output-based testing for reliability".
- Relax those passages so the basis is an example, not a requirement, and the reader uses the project's own testing semantics.
- Leave the rest of the skill unchanged.

<!-- instruct: work -->
## Work

The skill file plugin/skills/scrip-programming/SKILL.md tells the reader to use the project's own testing semantics. An output diff of tests/output against tests/expected is an example in the Iron Law on testing, the Testing section, the Quick Reference test row, and the closing values.

<!-- instruct: learnings -->
## Learnings

- The skill file names a tests/output versus tests/expected basis only in the Iron Law on testing, the Testing section, the Quick Reference test row, and the closing value "Output-based testing for reliability". The Directory Structure block names a tests/ directory and does not name that basis. Evidence: a read of plugin/skills/scrip-programming/SKILL.md; those four passages are the only matches for tests/expected, tests/output, and "Output-based".

<!-- instruct: decisions -->
## Decisions

- detail: The slice delivers a revision of plugin/skills/scrip-programming/SKILL.md that does not require a tests/output versus tests/expected basis and tells the reader to use the project's own testing semantics. Basis: the parent sequence entry for this slice, and that file is where the skill states the basis.
- detail: The proof is a read of that file's Iron Laws, Testing section, Quick Reference test row, and closing values, and it fails while any of those still require the basis and offer no other. Basis: those passages are what require the basis, and the check runs while the item is at work/wip.
- detail: The work edits only that file, keeps the output-diff layout as an example, and tells the reader to use the project's own testing semantics. Basis: the requirement lives in those passages of one file, and the sequence asks to relax it rather than replace it with another required basis.
- detail: Keep the item a slice. Basis: the proof is one result in one file, and the passages are one convention, not unrelated places.
- capture: none. Basis: the README's only learning names where this skill states the testing basis, which is item substance, not a reusable process.
- capture: commit plugin/skills/scrip-programming/SKILL.md jits/revise-testing/README.md jits/revise-testing/relax-the-testing-convention/README.md in /home/web/src/dev/scrip-programming as "./revise-testing/relax-the-testing-convention: Relax the testing convention". Basis: the item is archived below the scope root.

<!-- instruct: verify -->
## Verify

### Round 1

Verdict: yes
