---
name: patch-quality
description: Use in Phase 5 when writing the patch field for each finding. Rules for a minimal, applicable unified diff.
---

# Patch quality — rules for the unified diff

The patch must apply cleanly to the current HEAD of the target repo and change the minimum.

## Format
- Standard unified diff, git style: `--- a/<path>` / `+++ b/<path>`, hunk headers `@@`.
- Path RELATIVE TO PROJECT ROOT (same as the `file` field), no `proj/` prefix, no absolute paths.
- Include 3 lines of context around each change.

## Content rules
1. ONE logical fix per patch — no drive-by cleanups, no reformatting, no import reordering.
2. Fix the ROOT cause, not the symptom: if nil comes from a lookup that should never miss, fix the lookup, don't add `&.` one hop away.
3. Prefer the smallest language-idiomatic fix: guard clause, proper default, correct operator.
4. Never change public API signature. Add handling INSIDE the function.
5. If the fix needs a new condition, keep it readable in 1-3 lines.
6. Do not add tests/comments/log lines to the patch unless the fix IS a missing guard.

## Self-check before including
- Line numbers in hununks correspond to the CURRENT file (the one you read in Step 1 of verify).
- Context lines match the file byte-for-byte (copy them from your read, don't retype from memory).
- The patch changes behavior ONLY for the triggering input from your verification.
- Apply mentally: read the patched file top to bottom once — nothing else broke.
