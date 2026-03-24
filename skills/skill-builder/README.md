---
name: skill-builder
description: "Creates and optimizes Claude Code skills using research-backed principles from 80+ academic, industry, and practitioner sources. Triggers on \"build a skill\", \"create a skill\", \"make a skill\", \"optimize a skill\", \"audit a skill\", \"improve a skill\", \"refactor a skill\", or discussion of developing, designing, scaffolding, or iterating on Claude Code skills or slash commands. Not for general prompt engineering, CLAUDE.md authoring, or non-skill configuration."
user-invocable: true
---

# Skill Builder

Create and optimize Claude Code skills using research-backed best practices. Operate in two modes: **Bootstrap** (create new skills) and **Optimize** (audit and improve existing skills).

## Mode Detection

Determine which mode to use:

- **Bootstrap**: The user wants to create a new skill from scratch, or has an idea for a skill but no SKILL.md yet.
- **Optimize**: The user has an existing skill (SKILL.md or command .md file) they want audited, improved, or refactored.

If unclear, ask: "Are we building a new skill or improving an existing one?"

---

## Bootstrap Mode

### Phase 1: Intake

Ask these questions one at a time. Do not proceed until each is answered. Skip questions the user has already answered in their initial request.

1. **Purpose**: "What does this skill do? Describe it in one sentence."
2. **Trigger**: "When should Claude load this skill? What phrases or situations should activate it?"
3. **Inputs**: "What information does the skill need from the user? (arguments, file paths, context)"
4. **Outputs**: "What should the skill produce? (files, reports, actions, messages)"
5. **Tools**: "What tools does this skill need? (Read, Grep, Bash, MCP tools, web search, etc.)"
6. **Invocation**: "Should this be user-only (`/skill-name`), auto-triggered by Claude, or both?"
7. **Complexity**: "Is this a simple reference skill (conventions/patterns), a multi-step workflow, or a tool-heavy integration?"

### Phase 2: Assess Degrees of Freedom

Based on intake, classify the skill:

- **High freedom** (heuristic guidance): Multiple valid approaches. Write text instructions with goals and constraints. Examples: style guides, review checklists, analysis frameworks.
- **Medium freedom** (parameterized workflow): Preferred pattern exists with some variation. Write step-by-step instructions with decision points. Examples: deployment workflows, data processing, report generation.
- **Low freedom** (exact scripts): Operations are fragile, consistency is critical. Write exact commands and validation steps. Examples: API integrations, config changes, database operations.

### Phase 3: Select Pattern

Match to one of four proven patterns:

1. **Stateful Workflow**: Step-by-step numbered instructions, state persistence, confirmation gates, error handling. Use for: deploy, migrate, process.
2. **Knowledge/Expertise**: Domain guidance, dos/don'ts, pattern references. Use for: conventions, style guides, review criteria.
3. **Tool-Integrated**: Multiple tool backends, fallback chains, helper scripts. Use for: API integrations, external service interactions.
4. **Analysis**: Read-only phases (Analyze → Report → Recommend), structured output. Use for: audits, reviews, investigations.

### Phase 4: Generate the Description

Generate the description BEFORE the body — highest-leverage output.

**Description rules:**
- Third person ("Processes X when Y", never "I help you" or "Use this to")
- Include specific trigger keywords in quotes
- Specify both WHAT the skill does AND WHEN to use it
- Under 1024 characters
- Include negative triggers if needed ("Not for X")

**Generate 2-3 candidate descriptions. Present them to the user. Let them pick or refine.**

### Phase 5: Generate SKILL.md

Write the SKILL.md following these structural rules:

**Frontmatter:**
```yaml
---
name: <kebab-case, max 64 chars>
description: "<the selected description from Phase 4>"
<include only the frontmatter fields that are needed — omit defaults>
---
```

**Body structure:**
```markdown
# <Skill Title>

<One sentence stating the objective>

## Instructions
- <Imperative bullets — "Parse the config" not "You should parse the config">
- <Each bullet = one action or constraint>

## Constraints
- <What NOT to do — explicit negative instructions>
- <Boundaries and limitations>

## Steps
1. <Understand — gather context>
2. <Investigate — read files, check state>
3. <Execute — perform the task>
4. <Verify — confirm the result>

## Output Format
<Expected output structure — be specific>
```

**Content rules — apply all of these:**
- Target under 2,000 tokens / 500 lines for the body
- Place critical instructions at the START and END (lost-in-the-middle effect)
- Use imperative directives, not narrative prose
- Include explicit constraints (what NOT to do)
- Use Markdown headings + bullet points consistently
- One concept per bullet
- No time-sensitive information
- Consistent terminology throughout
- Provide defaults with escape hatches, not multiple equivalent options
- If the skill involves tool use, include tool descriptions with examples and edge cases

**Progressive disclosure — move detail out of SKILL.md:**
- Detailed reference material → `references/<topic>.md`
- Working examples → `examples/<name>.md`
- Executable utilities → `scripts/<name>.sh`
- Templates → `assets/<name>.md`
- References must be ONE level deep (SKILL.md → reference file, never reference → reference)

### Phase 6: Generate Supporting Files

Based on the skill's complexity:

- **All skills**: Consider if examples are needed (1-5 diverse, realistic, grounded in actual use cases)
- **Workflow skills**: Generate a reference file for error handling / edge cases
- **Tool-integrated skills**: Generate script files for validation or helper operations
- **Analysis skills**: Generate output template in assets/

### Phase 7: Critique and Refine

Run the generate-critique-refine loop. Read `${CLAUDE_SKILL_DIR}/references/quality-checklist.md` and audit the generated skill against every item. Fix any violations before presenting to the user.

After self-critique, present the complete skill to the user with:
1. The directory structure
2. The full SKILL.md content
3. Any supporting files
4. A summary of which principles were applied and why
5. Any trade-offs made

### Phase 8: Evaluation Scenarios

Generate 3 test scenarios for the skill:
1. **Happy path**: Standard use case that exercises the core workflow
2. **Edge case**: Unusual input or unexpected state
3. **Failure case**: What happens when something goes wrong

Present these to the user as manual test cases they can run to validate the skill.

---

## Optimize Mode

### Step 1: Read the Existing Skill

Read the SKILL.md (and any supporting files) that the user wants to optimize.

### Step 2: Audit Against Quality Checklist

Read `${CLAUDE_SKILL_DIR}/references/quality-checklist.md` and score the skill on every item. Classify each as:
- **Pass**: Meets the principle
- **Warn**: Partially meets, could improve
- **Fail**: Violates the principle

### Step 3: Structural Analysis

Check:
- Token count / line count of SKILL.md body
- Whether critical instructions are buried in the middle
- Whether the description contains trigger keywords and is third person
- Whether references are one level deep
- Whether frontmatter fields are correct and complete
- Whether the skill passes the "intern test" — could someone unfamiliar follow the instructions?

### Step 4: Present Findings

Present an audit report:

```
## Skill Audit: <skill-name>

### Score: X/18

### Passes
- <principle>: <why it passes>

### Warnings
- <principle>: <current state> → <recommendation>

### Failures
- <principle>: <current state> → <fix required>

### Recommendations (ordered by impact)
1. <highest impact fix>
2. <next>
3. <next>
```

### Step 5: Apply Fixes

Ask the user: "Want me to apply these fixes? I'll address them in impact order."

Apply fixes one at a time, explaining each change. After all fixes, re-run the audit to verify improvements.

---

## Constraints

- Do NOT generate a skill without reading `${CLAUDE_SKILL_DIR}/references/principles.md` first — it contains the research-backed principles that ground every decision
- Do NOT present a skill to the user without running `${CLAUDE_SKILL_DIR}/references/quality-checklist.md` first — no skill ships without passing audit
- Do NOT create new files when editing an existing skill achieves the goal — avoid file bloat
- Do NOT include unnecessary frontmatter fields — omit defaults
- Do NOT add any line without first asking "Would removing this cause Claude to make mistakes?" — if no, remove it

## Universal Rules

- Always generate the description before the body — highest-leverage output
- Always run the quality checklist before presenting — no skill ships without passing audit
- Challenge every token: justify each line's existence
- Prefer editing existing skills over creating new files
- This skill demonstrates what it teaches — follow every principle it enforces
