---
name: "OPSX: Verify"
description: Validate implementation against artifacts before archiving
category: Workflow
tags: [workflow, verify, experimental]
---

Validate implementation against change artifacts across three dimensions: completeness, correctness, and coherence.

**Input**: Optionally specify a change name (e.g., `/opsx:verify add-auth`). If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

**Steps**

1. **Select the change**

   If a name is provided, use it. Otherwise:
   - Infer from conversation context if the user mentioned a change
   - Auto-select if only one active change exists
   - If ambiguous, run `openspec list --json` to get available changes and use the **AskUserQuestion tool** to let the user select

   Always announce: "Using change: <name>" and how to override (e.g., `/opsx:verify <other>`).

2. **Load change context**

   ```bash
   openspec status --change "<name>" --json
   ```

   Parse the JSON to understand:
   - `schemaName`: The workflow being used
   - `artifacts`: List of artifacts with their status

   Read all existing artifacts for the change (proposal, specs, design, tasks, etc.) from the paths reported by status.

3. **Scan the implementation**

   Identify what code was changed for this change by:
   - Reading the tasks file to understand what was implemented
   - Examining the referenced files and code areas
   - Building a picture of what the implementation actually does

4. **Validate across three dimensions**

   **Completeness**
   - Are all tasks in the tasks file checked off?
   - Do all requirements in specs have corresponding code?
   - Are all scenarios covered (tested or handled)?

   **Correctness**
   - Does the implementation match the spec intent?
   - Are edge cases from scenarios handled?
   - Do error states match spec definitions?

   **Coherence**
   - Are design decisions from design.md reflected in code structure?
   - Are naming conventions consistent with design.md?
   - Are patterns consistent with what was planned?

5. **Report findings**

   Present a structured report with per-dimension results.

**Output**

```
## Verify: <change-name>

**Schema:** <schema-name>

### Completeness
✓ All N tasks in tasks.md are checked
✓ All requirements in specs have corresponding code
⚠ Scenario "X" not tested

### Correctness
✓ Implementation matches spec intent
✓ Edge cases from scenarios are handled
✓ Error states match spec definitions

### Coherence
✓ Design decisions reflected in code structure
✓ Naming conventions consistent with design.md
⚠ Design mentions "X" but implementation uses "Y"

### Summary
─────────────────────────────
Critical issues: 0
Warnings: 2
Ready to archive: Yes (with warnings)

### Recommendations
1. Add test for scenario "X"
2. Consider refactoring to match design, or update design.md
```

**Guardrails**
- Must read all artifacts AND scan actual implementation code before reporting
- Distinguish critical issues (spec requirement missing entirely) from warnings (design/code mismatch)
- Suggest specific fixes for each issue found
- Advisory only — verify won't block archive, but surfaces issues worth addressing first
- If no change exists or artifacts are missing, guide user to the right command
- If tasks file has no tasks, note this but don't treat as a failure

**Fluid Workflow Integration**

This command supports the "actions on a change" model:

- **Fits between apply and archive**: `/opsx:apply → /opsx:verify → /opsx:archive`
- **Can be invoked anytime**: After partial implementation, after all tasks complete, or even mid-implementation to check progress
- **Non-blocking**: Surfaces issues and recommendations but never prevents archiving
