# Command Authoring Reference

Complete reference for creating and improving Claude Code slash commands.

## Command File Structure

```markdown
---
description: <short description for /help and autocomplete>
model: <model-id>
allowed-tools: <comma-separated tools>
argument-hint: <hint shown in autocomplete>
---

<Brief intro of what the command does>

## Arguments (if command accepts arguments)

Parse the following arguments from: `$ARGUMENTS`

| Argument | Description | Default |
|----------|-------------|---------|
| ... | ... | ... |

### Examples

```bash
/command-name arg1
/command-name arg1 --flag
```

## Steps

1. <First step>
2. <Second step>
...

## Notes (optional)
```

## Frontmatter Fields

| Field | Purpose | Required |
|-------|---------|----------|
| `description` | Brief text shown in `/help` and autocomplete | Yes |
| `model` | Model to use (see Model Selection below) | No (inherits) |
| `allowed-tools` | Tools the command can use | Yes if using tools |
| `argument-hint` | Shows expected args in autocomplete (e.g., `<file> [--verbose]`) | No |
| `disable-model-invocation` | Prevent Claude from auto-invoking | No (default: false) |

## Model Selection

Choose based on task complexity:

| Model | Model ID | Use When |
|-------|----------|----------|
| **Haiku** | `claude-haiku-4-5-20251001` | Simple tasks: scripts, file ops, shell commands, clear step-by-step instructions |
| **Sonnet** | `claude-sonnet-4-5-20250929` | Medium: code generation, refactoring, debugging, moderate analysis |
| **Opus** | `claude-opus-4-5-20251101` | High: architecture, complex reasoning, planning, design decisions |

**Default to Haiku when in doubt.** Most commands should use Haiku.

## Tool Patterns

Format `allowed-tools` with specific patterns:

```yaml
# Bash with specific commands
allowed-tools: Bash(git:*), Bash(bun run:*), Bash(npm install:*)

# File tools
allowed-tools: Read, Write, Edit, Glob, Grep

# Interactive
allowed-tools: AskUserQuestion

# Combined
allowed-tools: Read, Write, Glob, Bash(git:*), AskUserQuestion
```

## Argument Syntax

Commands can use these variables:

| Variable | Purpose |
|----------|---------|
| `$ARGUMENTS` | All arguments as one string |
| `$1`, `$2`, `$3` | Individual positional arguments |

### Inline Execution

Use exclamation mark + backticks to include command output in the prompt:

```markdown
Current branch: `!git branch --show-current`
```

### File Inclusion

Use `@path/to/file` to include file contents in the prompt.

## Command Locations

| Location | Path | Invocation |
|----------|------|------------|
| Project | `.claude/commands/name.md` | `/name` |
| Project (local) | `.claude/commands/name.local.md` | `/name` |
| Personal | `~/.claude/commands/name.md` | `/name` |
| Plugin | `plugin/commands/name.md` | `/plugin:name` |

**Precedence:** Personal > Project local > Project > Plugin

## Quality Checklist

When creating or improving commands:

- [ ] Clear, specific steps (no "etc." or "as needed")
- [ ] All edge cases handled
- [ ] Error scenarios covered
- [ ] Appropriate model for complexity
- [ ] All required tools in `allowed-tools`
- [ ] Arguments documented with defaults
- [ ] Examples provided if arguments used
- [ ] Consistent argument names between docs and usage

## Common Patterns

### Interactive Command

```markdown
---
description: Do something with user input
model: claude-sonnet-4-5-20250929
allowed-tools: Read, Write, AskUserQuestion
---

## Steps

1. Ask the user what they want using AskUserQuestion
2. Process their response
3. Confirm before making changes
```

### Git Workflow Command

```markdown
---
description: Automate a git workflow
model: claude-haiku-4-5-20251001
allowed-tools: Bash(git:*)
---

## Steps

1. Check current status with `git status`
2. Perform operations
3. Confirm completion
```

### File Processing Command

```markdown
---
description: Process files matching a pattern
model: claude-haiku-4-5-20251001
allowed-tools: Glob, Read, Write
argument-hint: <pattern>
---

## Arguments

| Argument | Description | Default |
|----------|-------------|---------|
| `pattern` | Glob pattern for files | Required |

## Steps

1. Find files matching `$ARGUMENTS`
2. Process each file
3. Report results
```
