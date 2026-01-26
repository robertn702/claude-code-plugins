---
description: Analyze conversation to find improvement opportunities
model: claude-opus-4-5-20251101
allowed-tools: AskUserQuestion, Read, Glob
argument-hint: [commands|agents|memory] [--auto]
---

Analyze the current conversation to find opportunities for improving commands, agents, or updating CLAUDE.md memory files.

## Arguments

Parse the following arguments from: `$ARGUMENTS`

| Argument | Description | Default |
|----------|-------------|---------|
| `type` | Filter to specific type: `commands`, `agents`, `memory`, or omit for all | all |
| `--auto` | Automatically proceed with highest-impact suggestion | `false` |

The type can be specified as a positional argument or with `--type`:

### Examples

```bash
# Analyze and show all opportunities
/reflect

# Only look for command opportunities
/reflect commands
/reflect --type commands

# Only look for agent opportunities
/reflect agents

# Only look for memory opportunities
/reflect memory

# Auto-implement the highest-impact suggestion
/reflect --auto

# Combine type filter with auto
/reflect commands --auto
/reflect agents --auto
```

## Steps

1. **Parse arguments** from `$ARGUMENTS`:
   - Check for type filter (first positional arg or `--type`): `commands`, `agents`, `memory`
   - Check for `--auto` flag
   - If no type specified, analyze all types

2. **Analyze the conversation** based on the type filter:

   ### Command Opportunities (when type is `commands` or `all`)

   #### New Command Opportunities
   | Pattern | Signal |
   |---------|--------|
   | Repeated multi-step processes | "We did X, then Y, then Z" appearing 2+ times |
   | Manual workflows | Step-by-step instructions that could be automated |
   | Complex operations | Tasks requiring multiple tool calls in sequence |
   | User requests | "Can you make a command for..." or "I wish there was a way to..." |

   #### Improve Command Opportunities
   | Pattern | Signal |
   |---------|--------|
   | Command limitations hit | "The X command should also do Y" |
   | Workarounds needed | Manual steps after running a command |
   | Edge cases discovered | Command failed or behaved unexpectedly |
   | Missing arguments | "I wish I could pass Z to the command" |

   ### Agent Opportunities (when type is `agents` or `all`)

   #### New Agent Opportunities
   | Pattern | Signal |
   |---------|--------|
   | Complex autonomous tasks | Tasks requiring judgment, multiple steps, domain expertise |
   | Repeated specialized analysis | "Analyze X for Y" done multiple times |
   | Review/validation workflows | Code review, PR analysis, security audits |
   | User requests | "Can you create an agent that..." or "I need something to automatically..." |

   #### Improve Agent Opportunities
   | Pattern | Signal |
   |---------|--------|
   | Agent triggered incorrectly | "I didn't want the X agent to run" (false positive) |
   | Agent missed trigger | "Why didn't the X agent run?" (false negative) |
   | Agent output issues | "The agent should also check for..." or "The output format is confusing" |
   | Missing capabilities | "The agent didn't consider X" |
   | Domain knowledge gaps | Agent made mistakes a domain expert wouldn't |

   ### Memory Opportunities (when type is `memory` or `all`)

   | Pattern | Signal |
   |---------|--------|
   | Corrections made | "Actually, you should do X not Y" |
   | Repeated clarifications | Same explanation given multiple times |
   | User preferences stated | "I prefer...", "Always use...", "Never do..." |
   | Project conventions discovered | Patterns specific to this codebase |
   | Mistakes to prevent | Errors that better context would have avoided |
   | Missing context | "You didn't know about X" |

3. **List existing items** to check for overlaps:
   - If analyzing commands: Read `.claude/commands/*.md` and `.claude/commands/*.local.md`
   - If analyzing agents: Read `~/.claude/agents/*.md`, `.claude/agents/*.md`, `~/.claude/plugins/*/agents/*.md`
   - If analyzing memory: Read CLAUDE.md files in project hierarchy
   - Avoid suggesting items that already exist
   - Identify items that could be improved instead

4. **Score each opportunity** by impact:

   | Factor | Weight |
   |--------|--------|
   | Frequency (how often would this help?) | High |
   | Time saved per occurrence | Medium |
   | Error prevention | High |
   | Complexity of manual alternative | Medium |

5. **Present findings** organized by type:

   ```
   ## Conversation Analysis Results

   ### High Impact Opportunities

   1. **[NEW COMMAND]** `deploy-preview`
      Pattern: Deployed preview 3 times with same 5-step process
      Impact: High - would save ~2 min per deployment
      Action: `/create-command deploy-preview`

   2. **[NEW AGENT]** `api-contract-validator`
      Pattern: Repeatedly validated API responses against schema manually
      Impact: High - complex validation requiring domain expertise
      Action: Create agent in `.claude/agents/api-contract-validator.md`

   3. **[IMPROVE AGENT]** `code-reviewer`
      Pattern: Agent missed TypeScript strict mode violations twice
      Impact: Medium - would catch more issues
      Action: `/improve-agent code-reviewer`

   4. **[MEMORY]** Project uses Drizzle for transactions
      Pattern: Corrected twice about using raw SQL
      Impact: High - prevents future mistakes
      Scope: project (CLAUDE.md)
      Action: `/improve-memory "Always use Drizzle ORM for database transactions, not raw SQL"`

   ### Lower Impact Opportunities

   5. **[IMPROVE COMMAND]** `create-worktree` missing --track flag
      Pattern: Manually ran `git branch --set-upstream` after worktree creation
      Impact: Medium - minor convenience
      Action: `/improve-command create-worktree`

   6. **[MEMORY]** User prefers arrow functions
      Pattern: Mentioned once in passing
      Impact: Low - style preference
      Scope: local (CLAUDE.local.md)
      Action: `/improve-memory "Prefer arrow functions" --scope local`
   ```

6. **Ask user to select** which opportunities to implement using the `AskUserQuestion` tool:
   - Use multi-select mode (`multiSelect: true`) to allow selecting multiple opportunities
   - List each opportunity as an option with a brief description
   - Always include a "Skip all" option
   - Example:
     ```
     AskUserQuestion({
       questions: [{
         question: "Which improvements would you like to implement?",
         header: "Select",
         multiSelect: true,
         options: [
           { label: "1. deploy-preview command", description: "Create new command for deployment workflow" },
           { label: "2. api-contract-validator agent", description: "Create new agent for API validation" },
           { label: "3. Improve code-reviewer agent", description: "Add TypeScript strict mode checks" },
           { label: "4. Drizzle guidance", description: "Add to CLAUDE.md" },
           { label: "Skip all", description: "Don't implement any changes" }
         ]
       }]
     })
     ```

7. **For each selected opportunity**, implement the change directly:
   - For memory updates: Edit the appropriate CLAUDE.md file
   - For new commands: Create the command file in `.claude/commands/`
   - For command improvements: Edit the existing command file
   - For new agents: Create the agent file in `.claude/agents/` (or appropriate location)
   - For agent improvements: Edit the existing agent file
   - If `--auto` flag was passed, implement the highest-impact suggestion without asking

8. **Confirm completion** by summarizing what was changed:
   ```
   ## Changes Made

   1. ✅ Created deploy-preview command
   2. ✅ Created api-contract-validator agent
   3. ✅ Improved code-reviewer agent
   4. ✅ Added Drizzle guidance to CLAUDE.md

   Files modified:
   - .claude/commands/deploy-preview.local.md
   - .claude/agents/api-contract-validator.md
   - ~/.claude/plugins/pr-review-toolkit/agents/code-reviewer.md
   - CLAUDE.md
   ```

## What This Command Looks For

### Conversation Signals → Actions

| You Said | I Suggest |
|----------|-----------|
| "Do X, then Y, then Z" (repeated) | New command to automate X→Y→Z |
| "The command should also..." | Improve that command |
| "I need something to automatically analyze/review/validate X" | New agent |
| "The agent should also check for..." | Improve that agent |
| "The agent triggered when it shouldn't have" | Improve agent description (reduce false positives) |
| "Why didn't the agent run?" | Improve agent description (reduce false negatives) |
| "Actually, always do X" | Add to CLAUDE.md memory |
| "I prefer X over Y" | Add to CLAUDE.local.md |
| "You didn't know about X" | Add X to relevant CLAUDE.md |
| "Remember that..." | Add to memory |
| "Next time, do X" | Add to memory or improve command/agent |
| "Can you make a command/agent..." | Create new command or agent |
| "I keep having to..." | New command, agent, or memory update |

### Type-Specific Focus

When a type filter is provided, the analysis focuses exclusively on that type:

| Type | Focus Areas |
|------|-------------|
| `commands` | New command opportunities, command improvements, command argument additions |
| `agents` | New agent opportunities, agent improvements, triggering accuracy, system prompt quality |
| `memory` | CLAUDE.md updates, project conventions, user preferences, error prevention |

## Notes

- This command analyzes the current conversation context (what you can see)
- Run at the end of productive sessions to capture learnings
- High-impact suggestions are those that would help frequently
- The command doesn't automatically make changes - it suggests and you decide
- Use `--auto` with caution; review suggestions first in important projects
- Use type filters (`commands`, `agents`, `memory`) to focus analysis on specific areas
- Agent opportunities are identified when tasks require autonomous judgment or domain expertise
