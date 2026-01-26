---
name: self-improve
description: Meta-prompting for creating commands, improving commands, and managing Claude memory. Use when the user wants to create automation, improve existing workflows, or persist learnings to CLAUDE.md files.
allowed-tools: Read, Write, Edit, Glob, Grep, AskUserQuestion
---

# Self-Improvement Skill

This skill provides meta-prompting capabilities for improving Claude's own tooling and memory. Use it when:

- Creating new slash commands to automate workflows
- Improving existing commands based on usage patterns
- Updating CLAUDE.md files to persist learnings
- Analyzing conversations for improvement opportunities

## When to Apply Self-Improvement

### Signals for New Commands

| Pattern | Action |
|---------|--------|
| Repeated multi-step processes | Create command to automate |
| User says "I keep having to..." | Create command |
| Complex operations requiring multiple tools | Create command |
| User asks "Can you make a command for..." | Create command |

### Signals for Command Improvements

| Pattern | Action |
|---------|--------|
| "The X command should also do Y" | Improve command |
| Workarounds needed after running command | Improve command |
| Command failed on edge case | Improve command |
| "I wish I could pass Z to the command" | Add argument |

### Signals for Memory Updates

| Pattern | Action |
|---------|--------|
| Corrections: "Actually, you should do X" | Update CLAUDE.md |
| Repeated clarifications | Update CLAUDE.md |
| User preferences: "I prefer...", "Always..." | Update CLAUDE.local.md |
| Project conventions discovered | Update appropriate CLAUDE.md |
| Mistakes that context would prevent | Update CLAUDE.md |

## Core Principles

1. **Clarity over brevity**: Expand vague instructions into specific steps
2. **Defensive design**: Add validation and error handling
3. **User-centric**: Consider what could confuse the executor
4. **Minimal changes**: Don't over-engineer; fix what's broken
5. **Preserve intent**: Improvements should do the same thing, better
6. **Right-size the model**: Use Haiku for simple tasks, Opus for complex reasoning

## Reference Files

- [COMMAND_AUTHORING.md](COMMAND_AUTHORING.md) - Command structure, frontmatter, model selection
- [MEMORY_MANAGEMENT.md](MEMORY_MANAGEMENT.md) - CLAUDE.md hierarchy, scope detection, placement

## Available Commands

This skill includes these slash commands (namespaced with `self-improve:`):

| Command | Purpose |
|---------|---------|
| `/self-improve:create-command` | Create a new slash command |
| `/self-improve:improve-command` | Analyze and improve an existing command |
| `/self-improve:improve-memory` | Update CLAUDE.md files with new learnings |
| `/self-improve:reflect` | Analyze conversation for improvement opportunities |
