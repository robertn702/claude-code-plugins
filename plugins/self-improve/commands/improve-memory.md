---
description: Analyze and update CLAUDE.md files to improve AI memory
model: claude-opus-4-5-20251101
allowed-tools: Read, Edit, Glob, Grep, AskUserQuestion
argument-hint: "<what to remember>" [--scope <scope>]
---

Analyze an improvement or guideline and update the appropriate CLAUDE.md file(s) to persist it in AI memory.

## Arguments

Parse the following arguments from: `$ARGUMENTS`

| Argument | Description | Default |
|----------|-------------|---------|
| `improvement` | What should be remembered (quoted string or text) | - |
| `--scope <scope>` | Force a specific scope: `project`, `app:<name>`, `package:<name>`, `local` | Auto-detected |
| `--dry-run` | Show proposed changes without writing | `false` |

### Examples

```bash
# Auto-detect scope based on content
/improve-memory "Always use Drizzle for complex transactions"

# Force project-wide scope
/improve-memory "Never use default exports" --scope project

# Target a specific app
/improve-memory "Use server components by default" --scope app:app

# Target a specific package
/improve-memory "This package cannot import from @hanover/ui" --scope package:trpc

# Personal preference (CLAUDE.local.md)
/improve-memory "I prefer arrow functions over function declarations" --scope local

# Preview changes
/improve-memory "Add error boundaries around async components" --dry-run
```

## Steps

1. **Parse arguments** from `$ARGUMENTS`:
   - Extract the improvement text
   - Check for `--scope` flag
   - Check for `--dry-run` flag

2. **If no improvement provided**, ask the user:
   - "What guideline or information should be remembered?"
   - "Is this a personal preference or a project standard?"

3. **Analyze the improvement** to determine scope (unless `--scope` provided):

   | Scope | File | When to Use |
   |-------|------|-------------|
   | `local` | `/CLAUDE.local.md` | **Default for root-level improvements.** Personal preferences, local environment, coding patterns |
   | `app:<name>` | `/apps/<name>/CLAUDE.md` | App-specific patterns, routes, components |
   | `package:<name>` | `/packages/<name>/CLAUDE.md` | Package-specific rules, API contracts, dependencies |
   | `project` | `/CLAUDE.md` | Only when explicitly requested with `--scope project` |

   **Detection heuristics:**
   - Mentions specific app/package name → target that app/package
   - Import rules, dependencies for a specific package → package-level
   - UI patterns, routing, pages → app-level
   - **Everything else (general coding patterns, preferences) → `local` (default)**
   - Only use `project` scope when user explicitly passes `--scope project`

4. **Find all CLAUDE.md files** in the project:
   ```
   - /CLAUDE.md (project root)
   - /CLAUDE.local.md (user local)
   - /apps/*/CLAUDE.md (app-specific)
   - /packages/*/CLAUDE.md (package-specific)
   ```

5. **Read the target CLAUDE.md file** to understand:
   - Current structure and sections
   - Existing related content (avoid duplicates)
   - Writing style and format conventions

6. **Search for related content** using Grep:
   - Check if similar guidance already exists
   - If found, propose updating existing content vs adding new

7. **Determine the best placement**:

   | Content Type | Suggested Section |
   |--------------|-------------------|
   | Coding standards | Code Style |
   | Architecture decisions | Architecture |
   | Tool usage | Commands / Guidelines |
   | Import rules | Package Import Rules |
   | Testing patterns | Testing & Validation |
   | Personal preferences | (new section or existing) |

8. **Generate the proposed change**:
   - Match the existing file's formatting style
   - Keep additions concise and actionable
   - Use bullet points for guidelines
   - Include examples where helpful

9. **Present the analysis**:
   ```
   ## Memory Update Analysis

   ### Improvement
   "{improvement text}"

   ### Detected Scope
   {scope} → {file path}
   Reason: {why this scope was chosen}

   ### Existing Related Content
   {any existing content that relates, or "None found"}

   ### Proposed Placement
   Section: {section name}
   Action: {Add new / Update existing / Create section}
   ```

10. **Show the diff**:
    - Display before/after for the affected section
    - Highlight the new content

11. **Ask for confirmation**:
    - If `--dry-run`: Show changes and exit
    - Otherwise: "Apply this change to {file}?" with options:
      - "Yes, apply"
      - "Change scope" (re-analyze with different target)
      - "Edit content" (modify the proposed text)
      - "Cancel"

12. **Write the change** (if confirmed):
    - Use Edit tool for surgical updates
    - Confirm completion with file path and section

## CLAUDE.md File Structure Reference

The project uses this hierarchy:

```
/CLAUDE.md                    # Project-wide: architecture, commands, code style
/CLAUDE.local.md              # User-private: personal prefs, local setup
/apps/app/CLAUDE.md           # Next.js app: components, routing, state
/apps/api/CLAUDE.md           # API: Lambda, tRPC, SST patterns
/apps/cli/CLAUDE.md           # CLI scripts: bash, migrations
/apps/supabase/CLAUDE.md      # Database: migrations, RLS, Drizzle
/packages/*/CLAUDE.md         # Package-specific rules and patterns
```

## Notes

- This command uses Opus for nuanced scope detection and content placement
- Always check for duplicates before adding new content
- Prefer updating existing sections over creating new ones
- Keep entries actionable: "Do X" not "X is important"
- AGENTS.md files are symlinks to CLAUDE.md - no separate updates needed
