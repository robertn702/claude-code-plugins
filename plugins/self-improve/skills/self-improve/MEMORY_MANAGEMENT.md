# Memory Management Reference

Complete reference for managing CLAUDE.md memory files.

## File Hierarchy

```
/CLAUDE.md                    # Project-wide: architecture, commands, code style
/CLAUDE.local.md              # User-private: personal prefs, local setup (gitignored)
/apps/*/CLAUDE.md             # App-specific: components, routing, state
/packages/*/CLAUDE.md         # Package-specific: rules, API contracts
```

**Precedence:** More specific files override general ones.

## Scope Detection

| Scope | File | When to Use |
|-------|------|-------------|
| `local` | `/CLAUDE.local.md` | **Default.** Personal preferences, coding patterns, local environment |
| `app:<name>` | `/apps/<name>/CLAUDE.md` | App-specific patterns, routes, components |
| `package:<name>` | `/packages/<name>/CLAUDE.md` | Package rules, API contracts, dependencies |
| `project` | `/CLAUDE.md` | Only when explicitly requested or truly project-wide |

### Detection Heuristics

| Signal | Scope |
|--------|-------|
| Mentions specific app/package name | Target that app/package |
| "I prefer...", "personally" | `local` |
| Import rules for specific package | Package-level |
| UI patterns, routing, pages | App-level |
| General coding patterns | `local` (default) |
| Architecture, team conventions | `project` (only if explicit) |

**Important:** Default to `local` scope unless there's a clear reason for broader scope.

## Content Placement

| Content Type | Suggested Section |
|--------------|-------------------|
| Coding standards | Code Style |
| Architecture decisions | Architecture |
| Tool usage | Commands / Guidelines |
| Import rules | Package Import Rules |
| Testing patterns | Testing & Validation |
| Personal preferences | Coding Preferences (in local) |

## Writing Guidelines

### Format

- Use bullet points for guidelines
- Keep entries actionable: "Do X" not "X is important"
- Include examples where helpful
- Match existing file's formatting style

### Good Examples

```markdown
## Coding Preferences

- Use `for...of` instead of `forEach`
- Prefer `!arr.length` over `arr.length === 0`
- Use Drizzle for all new database calls
```

### Bad Examples

```markdown
## Notes

- forEach is bad (too vague)
- Remember to check array length properly (not actionable)
- Database stuff is important (meaningless)
```

## Before Adding Content

1. **Check for duplicates**: Search existing CLAUDE.md files
2. **Prefer updates**: Update existing content vs adding new
3. **Right-size scope**: Use most specific applicable scope
4. **Keep it lean**: Only add what would prevent mistakes

## Common Sections

### CLAUDE.md (Project)

```markdown
# Project Name

## Quick Reference
## Boundaries (Always/Ask First/Never)
## Architecture
## Code Style
## Guidelines
## Testing & Validation
```

### CLAUDE.local.md (Personal)

```markdown
# Local Claude Instructions

## Interaction Preferences
## Coding Preferences
## Domain Terminology
## Context Management
## Additional Commands
```

### Package CLAUDE.md

```markdown
# Package Name

## Purpose
## Key Files
## Dependency Constraints
## Patterns
```

## AGENTS.md Symlinks

Many projects symlink `AGENTS.md` to `CLAUDE.md` for cross-agent compatibility:

```bash
# Don't edit AGENTS.md directly - it's a symlink
ls -la apps/app/AGENTS.md  # -> CLAUDE.md
```

Edit the `CLAUDE.md` file; the symlink will reflect changes.

## Workflow

1. **Identify the learning**: What should Claude remember?
2. **Detect scope**: Where does this belong?
3. **Check existing**: Search for related content
4. **Determine placement**: Which section?
5. **Draft content**: Match style, keep actionable
6. **Confirm with user**: Show diff before writing
7. **Write change**: Use Edit tool for surgical updates
