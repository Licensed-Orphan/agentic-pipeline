# Skill Patterns Reference

Four proven patterns for Claude Code skills, each with a starter template. Select based on the skill's purpose and complexity.

---

## Pattern 1: Stateful Workflow

**Use when**: The skill performs a multi-step operation with state, side effects, or external interactions. Examples: deploy, migrate, process data, send messages.

**Characteristics**:
- Step-by-step numbered workflow
- State persistence between steps (JSON files, git state)
- Confirmation gates before destructive actions
- Error handling at every step with graceful degradation
- Explicit boundaries (Always / Ask First / Never)

**Template**:
```markdown
---
name: <verb-noun>
description: "<Third person. Trigger keywords. What and when.>"
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash
---

# <Skill Title>

<One sentence objective.>

## Prerequisites
- <Required state, tools, or access>

## Workflow

### Step 1: Gather Context
- <Read relevant files or state>
- <Validate prerequisites are met>
- If prerequisites fail, stop and report what's missing

### Step 2: Plan
- <Determine what needs to happen>
- <Present plan to user for approval before proceeding>

### Step 3: Execute
- <Perform the operation>
- <Log each action taken>
- If any step fails, stop and report the error with context

### Step 4: Verify
- <Confirm the operation succeeded>
- <Report results to user>

## Constraints
- Do NOT <destructive action> without user confirmation
- Do NOT proceed if <prerequisite> is missing
- Always <safety measure>

## Error Handling
- If <error A>: <recovery action>
- If <error B>: <recovery action>
- If unknown error: stop, report full error context, ask user
```

---

## Pattern 2: Knowledge/Expertise

**Use when**: The skill provides domain knowledge, conventions, or guidelines that Claude should apply when relevant. Examples: style guides, API conventions, review criteria, coding standards.

**Characteristics**:
- Pure instructional content
- No state management or side effects
- Domain-specific dos and don'ts
- Pattern references with examples
- Often auto-triggered (no `disable-model-invocation`)

**Template**:
```markdown
---
name: <domain-conventions>
description: "<Third person. When to apply this knowledge.>"
---

# <Domain> Guidelines

Apply these guidelines when <trigger condition>.

## Principles
- <Core principle 1>
- <Core principle 2>
- <Core principle 3>

## Do
- <Specific positive pattern with brief example>
- <Specific positive pattern with brief example>

## Do Not
- <Specific anti-pattern with brief reason why>
- <Specific anti-pattern with brief reason why>

## Examples

<example>
<input>User asks to <scenario></input>
<output>
<Correct application of guidelines>
</output>
</example>
```

---

## Pattern 3: Tool-Integrated

**Use when**: The skill orchestrates multiple tools, APIs, or external services. Examples: API integrations, MCP tool workflows, CLI orchestration.

**Characteristics**:
- Multiple tool backends with fallback chains
- Helper scripts in `scripts/`
- Detailed tool descriptions with examples
- Error recovery strategies
- Security considerations for API keys/credentials

**Template**:
```markdown
---
name: <service-action>
description: "<Third person. Trigger keywords. What and when.>"
allowed-tools: Read, Grep, Glob, Bash, <MCP tools>
---

# <Skill Title>

<One sentence objective.>

## Available Tools

### Primary: <Tool Name>
- **What**: <What it does>
- **When**: <When to use it>
- **Example**: `<example invocation>`

### Fallback: <Alternative Tool>
- **What**: <What it does>
- **When**: Use when primary tool is unavailable or errors

## Workflow

1. Attempt operation with primary tool
2. If primary fails, try fallback
3. If both fail, report error with context and suggest manual steps

## Input Format
- <Expected input format with example>

## Output Format
- <Expected output format with example>

## Constraints
- Do NOT store credentials in skill files
- Do NOT make destructive API calls without confirmation
- Always validate API responses before acting on them
```

---

## Pattern 4: Analysis

**Use when**: The skill analyzes code, data, or systems and produces a report or recommendations. Examples: audits, reviews, investigations, diagnostics.

**Characteristics**:
- Read-only phases (Analyze → Report → Recommend)
- Structured output templates
- No side effects during analysis
- Optional action phase after user reviews findings

**Template**:
```markdown
---
name: <analyzing-domain>
description: "<Third person. Trigger keywords. What and when.>"
allowed-tools: Read, Grep, Glob
---

# <Skill Title>

<One sentence objective.>

## Phase 1: Analyze
- <What to read/scan/search>
- <What patterns to look for>
- <What metrics to collect>

## Phase 2: Report

Present findings in this format:

```
## <Analysis> Report

### Summary
<1-3 sentence overview>

### Findings
1. <Finding with evidence>
2. <Finding with evidence>

### Recommendations (ordered by impact)
1. <Highest impact recommendation>
2. <Next>
```

## Phase 3: Act (Optional)
- Ask user: "Want me to apply any of these recommendations?"
- Apply only what user approves
- Verify each change after applying

## Constraints
- Do NOT modify files during analysis phases
- Do NOT recommend changes without evidence from the analysis
- Present findings before taking any action
```

---

## Pattern Selection Guide

| Signal | Pattern |
|--------|---------|
| "deploy", "send", "migrate", "process", "create" | Stateful Workflow |
| "conventions", "style", "guidelines", "standards" | Knowledge/Expertise |
| "API", "integrate", "connect", "sync", "MCP" | Tool-Integrated |
| "audit", "review", "analyze", "investigate", "diagnose" | Analysis |
| Multiple signals | Start with the dominant one; add elements from others as needed |
