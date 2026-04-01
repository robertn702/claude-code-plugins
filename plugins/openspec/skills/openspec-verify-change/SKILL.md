---
name: openspec-verify-change
description: Validate implementation against OpenSpec artifacts before archiving. Use when the user wants to check their work, verify implementation matches specs/design, or before archiving a change.
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.2.0"
---

Validate implementation against change artifacts across three dimensions: completeness, correctness, and coherence.

**Input**: Optionally specify a change name. If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

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
   - `schemaName`: The workflow being used (e.g., "spec-driven")
   - `artifacts`: List of artifacts with their status (`done` or other)

   **If no artifacts exist:**
   - Inform the user there's nothing to verify yet
   - Suggest creating artifacts with `/opsx:propose` or `/opsx:continue`
   - Stop here

3. **Read all artifacts**

   Read every existing artifact for the change. The files depend on the schema, but typically:
   - `openspec/changes/<name>/proposal.md` — what and why
   - `openspec/changes/<name>/specs/` — requirements and scenarios
   - `openspec/changes/<name>/design.md` — how (architecture, patterns, decisions)
   - `openspec/changes/<name>/tasks.md` — implementation steps

   Use the artifact paths from the status output rather than assuming file names.

4. **Scan the implementation**

   Build a picture of what was actually implemented:
   - Read the tasks file to understand what work items exist
   - For each task, identify the files and code areas it touches
   - Read those implementation files to understand what the code actually does
   - Note any code that doesn't correspond to a task or spec requirement

   **Important**: This is a semantic review, not a structural one. You're checking whether the *intent* of the artifacts is reflected in the *reality* of the code.

5. **Validate: Completeness**

   Check that everything planned was actually done:

   - **Task completion**: Are all tasks in tasks.md marked as `- [x]`? List any incomplete tasks.
   - **Requirement coverage**: For each requirement in specs, does corresponding code exist? Flag requirements with no implementation.
   - **Scenario coverage**: Are scenarios from specs tested or handled in code? Flag untested scenarios.

   Mark each check as ✓ (pass), ⚠ (warning), or ✗ (critical).

6. **Validate: Correctness**

   Check that the implementation matches what was specified:

   - **Spec intent**: Does the code do what the specs describe? Look for subtle mismatches (e.g., spec says "reject invalid input" but code silently ignores it).
   - **Edge cases**: Are edge cases from scenarios explicitly handled? Look for missing guards, unchecked error paths.
   - **Error states**: Do error behaviors match what specs define? Check that errors are surfaced, not swallowed.

   Mark each check as ✓ (pass), ⚠ (warning), or ✗ (critical).

7. **Validate: Coherence**

   Check that the implementation reflects the design:

   - **Architecture alignment**: Does the code structure match what design.md describes? (e.g., if design says "event-driven", is the code actually event-driven?)
   - **Naming consistency**: Do code names (functions, classes, variables) align with terminology in design.md?
   - **Pattern consistency**: Are the patterns used in code the ones the design specified? Flag where implementation diverged from design.

   Mark each check as ✓ (pass), ⚠ (warning), or ✗ (critical).

8. **Report findings**

   Present a structured report covering all three dimensions.

**Output**

```
## Verify: <change-name>

**Schema:** <schema-name>

### Completeness
✓ All N tasks in tasks.md are checked
✓ All requirements in specs have corresponding code
⚠ Scenario "Session timeout after inactivity" not tested

### Correctness
✓ Implementation matches spec intent
✓ Edge cases from scenarios are handled
✓ Error states match spec definitions

### Coherence
✓ Design decisions reflected in code structure
✓ Naming conventions consistent with design.md
⚠ Design mentions "event-driven" but implementation uses polling

### Summary
─────────────────────────────
Critical issues: 0
Warnings: 2
Ready to archive: Yes (with warnings)

### Recommendations
1. Add test for session timeout scenario
2. Consider refactoring to event-driven as designed, or update design.md
```

**Issue Classification**

| Level | Meaning | Examples |
|-------|---------|---------|
| ✗ Critical | Spec requirement not implemented at all | Missing feature, unhandled required error state |
| ⚠ Warning | Implementation diverges from artifacts | Design/code mismatch, untested scenario, naming inconsistency |
| ✓ Pass | Artifacts and implementation align | Requirement implemented, design reflected in code |

**Ready to Archive Assessment**

| Condition | Assessment |
|-----------|------------|
| No critical issues, no warnings | Ready to archive |
| No critical issues, some warnings | Ready to archive (with warnings) |
| Any critical issues | Not ready — address critical issues first |

**Guardrails**
- Must read all artifacts AND scan actual implementation code before reporting
- Distinguish critical issues from warnings — critical means something is missing, warning means something diverges
- Suggest specific fixes for each issue: "Add test for X" or "Update design.md to reflect Y"
- Advisory only — verify surfaces issues but never blocks archive
- If artifacts are incomplete (not all created yet), verify what exists and note what's missing
- If tasks file has no tasks, note this but don't treat as a failure
- Don't fabricate issues — if implementation looks good, say so clearly
- Be concrete — reference specific files, functions, spec requirements by name

**Fluid Workflow Integration**

This skill supports the "actions on a change" model:

- **Fits between apply and archive**: `apply → verify → archive`
- **Can be invoked anytime**: After partial implementation, after all tasks complete, or mid-implementation to check progress
- **Non-blocking**: Surfaces issues and recommendations but never prevents archiving
- **Suggests next steps**: If issues found, suggest whether to fix code or update artifacts; if clean, suggest archive
