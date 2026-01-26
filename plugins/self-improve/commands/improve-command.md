---
description: Improve an existing slash command using meta-prompting
model: claude-opus-4-5-20251101
allowed-tools: Read, Write, Glob, AskUserQuestion
argument-hint: <command-name>
---

Analyze and improve an existing slash command using meta-prompting techniques.

## Arguments

Parse the following arguments from: `$ARGUMENTS`

| Argument | Description | Default |
|----------|-------------|---------|
| `command-name` | Name of the command to improve (without path or extension) | - |
| `--dry-run` | Show proposed changes without writing | `false` |

### Examples

```bash
# Improve the create-worktree command
/improve-command create-worktree

# Preview changes without writing
/improve-command pr-comments --dry-run
```

## Steps

1. **Parse arguments** from `$ARGUMENTS`:
   - Extract command name (required)
   - Check for `--dry-run` flag

2. **Locate the command file**:
   - Search in `.claude/commands/` for:
     - `{name}.local.md` (local command)
     - `{name}.md` (project command)
   - If not found, report error and list available commands

3. **Read and analyze the command** for:

   | Aspect | What to Check |
   |--------|---------------|
   | **Structure** | Clear sections (Arguments, Steps, Notes), logical flow |
   | **Clarity** | Unambiguous instructions, no vague language ("etc.", "as needed") |
   | **Completeness** | All edge cases handled, error scenarios covered |
   | **Model selection** | Appropriate model for task complexity |
   | **Tools coverage** | All required tools listed in `allowed-tools` |
   | **Argument docs** | Clear descriptions, defaults specified, examples provided |
   | **Frontmatter** | All required fields present (`description`, `allowed-tools`) |
   | **Consistency** | Argument names match between docs and usage |

4. **Generate improvements** using meta-prompting:
   - Identify specific weaknesses with explanations
   - Propose concrete fixes for each issue
   - Preserve the original intent and functionality
   - Maintain the author's style where possible
   - Add missing edge case handling
   - Clarify ambiguous instructions
   - Fix any tool pattern issues (e.g., `Bash(git:*)` format)

5. **Present the analysis**:
   ```
   ## Analysis of /{command-name}

   ### Strengths
   - [What the command does well]

   ### Issues Found
   1. [Issue]: [Explanation]
      Fix: [Proposed solution]

   ### Suggested Model
   Current: [model]
   Recommended: [model] - [reason]
   ```

6. **Show the diff**:
   - Display a clear before/after comparison
   - Highlight key changes
   - Explain the reasoning for significant modifications

7. **Ask for confirmation**:
   - If `--dry-run`: Show changes and exit
   - Otherwise: Ask user to confirm before writing
   - Options: "Apply changes", "Apply with modifications", "Cancel"

8. **Write the improved command** (if confirmed):
   - Back up original (optional, ask user)
   - Write improved version
   - Confirm completion with file path

## Meta-Prompting Principles

When analyzing and improving commands, apply these principles:

1. **Clarity over brevity**: Expand vague instructions into specific steps
2. **Defensive design**: Add validation and error handling
3. **User-centric**: Consider what could confuse the executor
4. **Minimal changes**: Don't over-engineer; fix what's broken
5. **Preserve intent**: The improved command should do the same thing, better

## Notes

- This command uses Opus for high-quality meta-analysis
- Large commands may require multiple improvement passes
- Always preserve the original command's core functionality
