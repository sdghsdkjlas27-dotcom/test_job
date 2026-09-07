---
name: bughunt-method
description: MASTER workflow for every bug-hunting session. Load this FIRST and follow it step by step in order. Do not skip steps.
---

# Bug-hunt method (follow IN ORDER)

You are a senior code auditor. Your only deliverable is `agent-findings.json`.
Work in phases. Never report a finding that has not passed the VERIFY phase.

## Phase 0 — Orient (2 tool calls max)
1. Read `proj/README.md` (first 100 lines) — what the project IS.
2. `git -C proj log --oneline -20` — what changed recently. Recent code is buggier code.

## Phase 1 — Map the system
1. List source files: glob `proj/**/*.{rb,java,js,ts,go,py}` (only real source dirs, skip tests/fixtures).
2. Pick the 6-10 files that are: (a) most recently changed, (b) imported/required by many others, (c) entry points (bin/, lib main file).
3. For each picked file, note: what it exports/defines, what it requires from OTHER files. This is your call map.

## Phase 2 — Hunt (the core loop)
For EACH picked file, run BOTH checks:
- **In-file check**: load the language pitfall skill (`ruby-pitfalls` / `java-pitfalls` / `js-ts-pitfalls` / `go-pitfalls`) and scan every function against its checklist line by line.
- **Cross-file check**: load `contract-mismatch` and verify every call boundary: what caller passes vs what callee expects (types, nil/null, units, ranges, error paths).

Keep a written list of candidate bugs (use todowrite). Aim for 3-6 candidates before verifying.

## Phase 3 — Verify (kill false positives)
For EACH candidate, load `verify-before-report` and execute its checklist completely.
A candidate that fails ANY verify step is DISCARDED silently — do not report it, do not mention it.
Expected survival rate: 30-50% of candidates. That is normal and good.

## Phase 4 — Dedup
For each surviving finding, compare its file+essence against the known reports list given in the task.
If an existing issue already describes the same defect in the same file — discard.

## Phase 5 — Report
1. Load `patch-quality` and write a minimal patch per finding.
2. Load `report-format` and write `agent-findings.json` EXACTLY in its schema.
3. Read the file back. If findings survived verification but the file is empty/invalid — rewrite it.

## Rules
- NEVER report style, naming, missing docs, or "could be refactored" — only wrong runtime behavior.
- NEVER invent APIs. If you did not see the exact line with your own read call, it does not exist.
- If a tool call fails, use a different approach (grep instead of glob, read with offset) — never abandon the phase.
- Budget: ~40-60 tool calls total. Stop hunting when you have 3 verified findings.
