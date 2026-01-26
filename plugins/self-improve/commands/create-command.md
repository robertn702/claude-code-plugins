---
description: Create a new custom slash command
model: claude-opus-4-5-20251101
allowed-tools: Write, Read, Glob, Bash(ls:*)
argument-hint: <command-name> [--project]
---

Create a new slash command in the `.claude/commands/` directory.

## Arguments

Parse the following arguments from: `$ARGUMENTS`

| Argument | Description | Default |
|----------|-------------|---------|
| `name` | Command name without extension (first positional arg, required) | - |
| `--project` | Create as `.md` (checked into git) instead of `.local.md` | `false` |
| `--model <model>` | Override automatic model selection | Auto-selected |

### Examples

```bash
# Create a local command (default, not committed)
/create-command db-reset

# Create a project command (checked in)
/create-command review-pr --project

# Override model selection
/create-command analyze-deps --model claude-opus-4-5-20251101
```

## Steps

1. **Parse arguments** from `$ARGUMENTS`:
   - Extract command name (required, first positional arg)
   - Check for `--project` flag (default: false, meaning `.local.md`)
   - Extract `--model` if provided

2. **Ask the user** for the following:

   a. **Description** (required):
      - Short phrase shown in `/help` and autocomplete
      - Should be 5-10 words max
      - Example: "Reset local database to clean state"

   b. **What the command should do** (required):
      - Detailed explanation of the command's purpose and behavior
      - What steps should it perform?
      - What output should it produce?

   c. **Arguments** (if any):
      - Does the command accept arguments?
      - What are they called and what do they do?
      - Which are required vs optional?
      - Suggest using `$ARGUMENTS` for simple single-arg commands, or `$1`, `$2`, etc. for multiple distinct args

   d. **Tools needed**:
      - What bash commands will it run? (e.g., `git`, `bun`, `npm`)
      - Will it read/write files? (`Read`, `Write`, `Edit`, `Glob`, `Grep`)
      - Will it need to ask questions? (`AskUserQuestion`)

3. **Select the model** based on task complexity (unless `--model` was provided):

   | Model | Use When |
   |-------|----------|
   | `claude-haiku-4-5-20251001` | Simple tasks: running scripts, copying files, simple shell operations, straightforward file manipulations, commands with clear step-by-step instructions |
   | `claude-sonnet-4-5-20250929` | Medium complexity: code generation, refactoring, debugging, implementing features, moderate analysis, commands requiring some judgment |
   | `claude-opus-4-5-20251101` | High complexity: architecture analysis, complex reasoning, planning, deep codebase understanding, design decisions, commands requiring significant judgment |

   **Default to the lower-complexity model when in doubt.** Most commands should use Haiku.

4. **Generate the command file**:

   File path: `.claude/commands/{name}.local.md` (or `.md` if `--project`)

   Use this structure:

   ```markdown
   ---
   description: <short description>
   model: <selected model>
   allowed-tools: <comma-separated tools>
   argument-hint: <hint for autocomplete, if command takes args>
   ---

   <Brief intro of what the command does>

   ## Arguments (include only if command accepts arguments)

   Parse the following arguments from: `$ARGUMENTS`

   | Argument | Description | Default |
   |----------|-------------|---------|
   | ... | ... | ... |

   ### Examples (if command takes arguments)

   ```bash
   /command-name arg1
   /command-name arg1 --flag
   ```

   ## Steps

   1. <First step>
   2. <Second step>
   ...

   ## Notes (optional, for important caveats)
   ```

5. **Write the file** and confirm:
   - Show the file path created
   - Show the model selected and why
   - Show how to use the command: `/command-name`

## Frontmatter Reference

| Field | Purpose | Required |
|-------|---------|----------|
| `description` | Brief text shown in `/help` and autocomplete | Yes |
| `model` | Model to use (haiku/sonnet/opus) | No (inherits) |
| `allowed-tools` | Tools the command can use | Yes if using bash |
| `argument-hint` | Shows expected args in autocomplete (e.g., `<file> [--verbose]`) | No |
| `disable-model-invocation` | Prevent Claude from auto-invoking | No (default: false) |

## Tool Patterns

Format `allowed-tools` with specific patterns:
- `Bash(git:*)` - any git command
- `Bash(bun run:*)` - bun run with any script
- `Bash(npm install:*)` - npm install specifically
- `Read`, `Write`, `Edit`, `Glob`, `Grep` - file tools
- `AskUserQuestion` - for interactive commands

## Special Syntax in Commands

Commands can use:
- `$ARGUMENTS` - all arguments as one string
- `$1`, `$2`, `$3` - individual positional arguments
- Inline bash execution: exclamation mark followed by command in backticks (output included in prompt)
- `@path/to/file` - include file contents in prompt
