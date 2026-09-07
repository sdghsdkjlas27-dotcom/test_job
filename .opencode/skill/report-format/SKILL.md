---
name: report-format
description: Use in Phase 5 to write agent-findings.json. Exact schema, field rules and severity rubric.
---

# Report format — agent-findings.json

Write a valid JSON OBJECT, no markdown fences, no commentary. Exact shape:

```json
{
  "findings": [
    {
      "title": "Short imperative summary of the wrong behavior",
      "file": "lib/module/file.rb",
      "line": 42,
      "severity": "major",
      "description": "What is wrong and WHY it is a bug: the contract broken, the state that triggers it, the consequence.",
      "evidence": "The exact 1-5 code lines from the file",
      "repro": "Concrete steps/command to observe the bug",
      "fix_idea": "One-sentence description of the fix",
      "patch": "--- a/lib/module/file.rb\n+++ b/lib/module/file.rb\n@@ ..."
    }
  ]
}
```

## Field rules
- `title`: <= 80 chars, names the BEHAVIOR not the file ("RateLimit mutates cached body when resources key missing", not "Bug in rate_limit.rb"). No "Bug:", no severity word.
- `file`: path from project root, forward slashes, NO proj/ prefix, NO absolute path.
- `line`: integer line of the defective statement (from your re-read, not memory).
- `description`: 2-5 sentences. First sentence states the wrong behavior. Then the trigger condition. Then the consequence. No speculation words ("maybe", "could", "might") — you verified it.
- `evidence`: verbatim snippet copied from the file. Include ONLY the guilty lines.
- `repro`: pasteable command or 2-4 step scenario with concrete values.
- `fix_idea`: one sentence, names the mechanism ("Guard nil from dig before assignment").
- `patch`: unified diff per patch-quality skill.

## Writing
- Write findings in English.
- Order findings by severity (critical first).
- If no findings survived verification: `{"findings": []}` — honest empty is better than one false positive.
- After writing, READ THE FILE BACK and confirm valid JSON with all fields present.
