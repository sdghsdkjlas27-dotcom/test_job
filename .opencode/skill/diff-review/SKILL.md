---
name: diff-review
description: Use in Phase 2 for reviewing RECENT COMMITS. How to find bugs in new/changed code - regression patterns, incomplete changes, broken invariants.
---

# Diff review — hunting bugs in recent commits

New code is 10x buggier than old code. Recent commits deserve line-by-line scrutiny.

## How to review
1. `git -C proj log --oneline -15` — list recent commits.
2. `git -C proj show <sha>` for the 3-5 most substantial commits (skip trivial "fix typo").
3. For EVERY hunk, ask the 6 questions below.

## The 6 questions per hunk

1. **Incomplete change**: does the new code reference a variable/method/field that the commit did NOT add? (Half-finished feature.) Grep the new symbol — if defined nowhere → bug now.
2. **Broken callers**: the commit changed a signature/return shape/behavior — do ALL existing callers still match? Grep every call site of the changed symbol.
3. **Inverted condition**: new guards/new comparisons — does the direction make sense? (`<` vs `<=`, `&&` vs `||`, `nil?` vs present). De Morgan check on compound conditions.
4. **Lost cleanup**: commit adds new state (field, key, temp file, cache entry) — is there a path that removes/closes/resets it? Missing teardown = leak/stale-state bug.
5. **Copy-paste rot**: new block copied from a sibling block — check EVERY line for unreplaced names/numbers (variable from the source context, wrong constant, off-by-one copy).
6. **Test tells the truth**: did the commit change tests to expect the new behavior? If tests were deleted/skipped/loosened in the same commit — suspicious, read what they used to assert (`git show <sha> -- test/`).

## High-signal diff patterns
- `+ def old_name` with `+ raise` / deprecation path — callers not migrated.
- `+ if` wrapping an existing block — check the else path still works (previously unconditional code now conditional).
- Changed default value / constant — grep who else depends on the old value.
- New `rescue` / catch added — does it swallow errors the old code propagated?
- Magic number changed — where does the pair value live (other file / config / test)?
