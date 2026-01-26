---
description: Improve an existing agent definition using meta-prompting
model: claude-opus-4-5-20251101
allowed-tools: Read, Write, Glob, Grep, AskUserQuestion
argument-hint: <agent-name> [--dry-run]
---

Analyze and improve an existing agent definition using meta-prompting techniques.

## Arguments

Parse the following arguments from: `$ARGUMENTS`

| Argument | Description | Default |
|----------|-------------|---------|
| `agent-name` | Name of the agent to improve (without path or extension) | - |
| `--dry-run` | Show proposed changes without writing | `false` |
| `--plugin <name>` | Search within a specific plugin | All plugins |

### Examples

```bash
# Improve an agent by name
/improve-agent code-reviewer

# Preview changes without writing
/improve-agent silent-failure-hunter --dry-run

# Improve an agent from a specific plugin
/improve-agent type-design-analyzer --plugin pr-review-toolkit
```

## Steps

1. **Parse arguments** from `$ARGUMENTS`:
   - Extract agent name (required)
   - Check for `--dry-run` flag
   - Check for `--plugin` filter

2. **Locate the agent file**:
   - Search in these locations (in order):
     - `~/.claude/agents/{name}.md` (personal agents)
     - `.claude/agents/{name}.md` (project agents)
     - `~/.claude/plugins/*/agents/{name}.md` (plugin agents)
   - If `--plugin` specified, only search that plugin
   - If not found, list available agents and exit

3. **Read and analyze the agent** for:

   | Aspect | What to Check |
   |--------|---------------|
   | **Frontmatter** | Required fields present (`name`, `description`), valid model choice |
   | **Name** | Lowercase, hyphens only, 3-50 chars, descriptive |
   | **Description** | Starts with "Use this agent when...", has trigger phrases |
   | **Examples** | 2-4 `<example>` blocks with context, user, assistant, commentary |
   | **System Prompt** | Clear role, numbered responsibilities, step-by-step process |
   | **Model Selection** | Appropriate for task complexity (haiku/sonnet/opus/inherit) |
   | **Tools** | Minimal set needed, or omitted for full access |
   | **Color** | Appropriate for agent purpose |
   | **Output Format** | Clear expectations for agent output |
   | **Edge Cases** | Guidance for unusual scenarios |

4. **Evaluate the system prompt quality**:

   | Criterion | Good | Needs Improvement |
   |-----------|------|-------------------|
   | **Length** | 500-3,000 words | Too short (<300) or too long (>4,000) |
   | **Structure** | Clear sections with headers | Wall of text |
   | **Specificity** | Concrete instructions | Vague guidance ("do your best") |
   | **Boundaries** | Clear scope limits | Ambiguous responsibilities |
   | **Methodology** | Specific techniques/patterns | Generic advice |

5. **Check triggering quality**:
   - Description should enable accurate auto-triggering
   - Examples should cover:
     - Direct requests ("create an X", "review the Y")
     - Indirect requests (user describes need without naming agent)
     - Proactive triggering (assistant decides to use agent)
   - Avoid false positive triggers (too broad)
   - Avoid false negatives (too narrow)

6. **Generate improvements** using meta-prompting:
   - Identify specific weaknesses with explanations
   - Propose concrete fixes for each issue
   - Preserve the original intent and domain expertise
   - Enhance system prompt with:
     - Better structured responsibilities
     - More specific methodologies
     - Clear output format expectations
     - Edge case handling
   - Improve triggering examples if needed

7. **Present the analysis**:
   ```
   ## Analysis of {agent-name}

   ### Agent Overview
   - **Location**: {file path}
   - **Current Model**: {model}
   - **Tools**: {tools or "all"}
   - **Color**: {color}

   ### Strengths
   - [What the agent does well]

   ### Issues Found
   1. [Issue]: [Explanation]
      Fix: [Proposed solution]

   ### Recommended Changes
   - Model: {current} → {recommended} (reason)
   - Tools: {current} → {recommended} (reason)
   - Description: [specific improvements]
   - System Prompt: [key changes]
   ```

8. **Show the diff**:
   - Display a clear before/after comparison
   - Highlight key changes to frontmatter
   - Show system prompt improvements
   - Explain reasoning for significant modifications

9. **Ask for confirmation**:
   - If `--dry-run`: Show changes and exit
   - Otherwise: Ask user to confirm before writing
   - Options: "Apply changes", "Apply with modifications", "Cancel"

10. **Write the improved agent** (if confirmed):
    - Write improved version to original location
    - Confirm completion with file path

## Agent Quality Reference

### Frontmatter Fields

| Field | Purpose | Required |
|-------|---------|----------|
| `name` | Agent identifier (lowercase, hyphens) | Yes |
| `description` | Triggering conditions and examples | Yes |
| `model` | Model to use (haiku/sonnet/opus/inherit) | No |
| `color` | Display color in UI | No |
| `tools` | Tool restrictions (array) | No |

### Model Selection Guide

| Model | Use When |
|-------|----------|
| `haiku` | Simple, focused tasks with clear steps |
| `sonnet` | Most agents - balanced capability and speed |
| `opus` | Complex reasoning, architecture decisions, nuanced judgment |
| `inherit` | Let parent context decide (good default) |

### Color Selection Guide

| Color | Use For |
|-------|---------|
| `blue`/`cyan` | Analysis, review, inspection |
| `green` | Generation, creation, building |
| `yellow` | Validation, warnings, caution |
| `red` | Security, critical issues, destructive |
| `magenta` | Transformation, creative, refactoring |

### System Prompt Structure

A well-structured system prompt includes:

1. **Role Statement**: Expert identity and domain knowledge
2. **Core Responsibilities**: Numbered list of primary duties
3. **Detailed Process**: Step-by-step methodology
4. **Quality Standards**: Criteria for good output
5. **Output Format**: Expected structure of results
6. **Edge Cases**: Handling unusual scenarios

## Meta-Prompting Principles

When analyzing and improving agents:

1. **Trigger Accuracy**: Improve description to trigger correctly (not too broad, not too narrow)
2. **Domain Expertise**: Enhance system prompt with deeper domain knowledge
3. **Actionable Instructions**: Replace vague guidance with specific techniques
4. **Structured Output**: Add clear output format expectations
5. **Preserve Intent**: The improved agent should do the same thing, better
6. **Minimal Changes**: Don't over-engineer; fix what's actually broken

## Notes

- This command uses Opus for high-quality meta-analysis of agents
- Agents are more complex than commands - take time to understand the domain
- Focus on improving triggering accuracy and system prompt quality
- The description field is crucial for auto-triggering - optimize it carefully
- Test improved agents with the scenarios in their examples
