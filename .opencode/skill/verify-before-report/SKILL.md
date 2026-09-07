---
name: verify-before-report
description: Use in Phase 3 on EVERY candidate bug BEFORE it goes into the report. Falsification checklist - execute all 7 steps, discard the candidate if ANY step fails.
---

# Verify before report — run ALL steps, discard on ANY failure

A candidate that fails a single step is a FALSE POSITIVE. Discard silently.
Goal: every reported finding survives maintainer scrutiny in 30 seconds.

## Step 1 — See the line again
Re-read the exact file with a fresh read call (with offset covering the line).
The code must STILL say what you remember. If your memory of it was wrong → discard.

## Step 2 — Name the exact input/state that triggers it
Write one concrete scenario: "when X is nil because caller Y passes lookup miss when Z".
If you cannot name the concrete triggering input/state → discard.
"It might crash sometimes" is NOT a finding.

## Step 3 — Trace ONE full path
Follow that one input through the code with your finger: caller → callee → crash line.
Every hop must be real code you have read this session. If any hop is assumed → discard.

## Step 4 — Check it is not intentional
- Read surrounding comments (the code may document the behavior).
- Grep for tests covering this exact code (`grep -r "function_name" proj/test` or spec dir). If a test ASSERTS this behavior → it is a spec, discard.
- Check git log for the file: `git -C proj log --oneline -5 -- <file>` — recent fix may have introduced OR already fixed it.

## Step 5 — Severity honesty
- critical: data loss/corruption, security, crash in main happy path
- major: crash or wrong result on realistic input, resource leak
- minor: crash only on edge input, cosmetic-but-wrong output
If you had to invent a contrived input no real caller produces → downgrade or discard.

## Step 6 — Fix is obvious and small
You must be able to state the fix in one sentence and it must not change public behavior
for correct inputs. If the "fix" requires redesign → not a bug report, discard.

## Step 7 — One-line repro
Write the repro as a command/snippet a maintainer can paste.
If you cannot write the repro → you do not understand the bug → discard.

## Common false-positive patterns (auto-discard)
- "Method X doesn't handle nil" — but every caller guards nil before calling (grep the callers!)
- "Performance is O(n²)" — not a bug unless data is unbounded and it's a hot path.
- "Encoding could mismatch" — without a concrete mixed-encoding source.
- Test-only code, fixtures, examples, vendored/generated files.
- Behavior already guarded by rescue/try 2 lines above where you stopped reading.
