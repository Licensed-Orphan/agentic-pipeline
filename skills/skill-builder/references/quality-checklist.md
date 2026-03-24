# Skill Quality Checklist

Use this checklist to audit any Claude Code skill. Score each item as Pass, Warn, or Fail.

---

## Description (Highest Leverage)

- [ ] **D1. Third person voice** — "Processes X when Y", never "I help you" or "Use this to"
- [ ] **D2. Trigger keywords included** — Contains specific quoted phrases that users would say
- [ ] **D3. Specifies WHAT and WHEN** — Both what the skill does and when it should be triggered
- [ ] **D4. Under 1024 characters** — Stays within the character limit
- [ ] **D5. Negative triggers if needed** — Specifies when NOT to use the skill if ambiguity exists

## Structure

- [ ] **S1. SKILL.md under 500 lines** — Body content stays under the line limit (~2,000 tokens)
- [ ] **S2. Critical instructions at start and end** — Most important content is not buried in the middle
- [ ] **S3. Markdown headings + bullets** — Consistent use of structured formatting throughout
- [ ] **S4. One concept per bullet** — Each bullet point contains a single action or constraint
- [ ] **S5. References one level deep** — No nested reference chains (SKILL.md → ref, never ref → ref)

## Content

- [ ] **C1. Imperative directives** — "Parse the file" not "You should parse the file"
- [ ] **C2. Explicit constraints** — Includes what NOT to do (Constraints section or inline)
- [ ] **C3. Consistent terminology** — Same terms used throughout (not mixing "endpoint"/"URL"/"route")
- [ ] **C4. No time-sensitive info** — No "if before date X" or references to temporary states
- [ ] **C5. Defaults with escape hatches** — Provides one recommended approach, not "use X or Y or Z"

## Frontmatter

- [ ] **F1. Name is kebab-case** — Lowercase, hyphens, max 64 characters
- [ ] **F2. Required fields present** — `name` and `description` both exist
- [ ] **F3. Invocation control correct** — `disable-model-invocation` and `user-invocable` match intended behavior
- [ ] **F4. Only needed fields included** — No unnecessary frontmatter (omit defaults)

## Completeness

- [ ] **X1. Passes the intern test** — Someone unfamiliar could follow the instructions with no additional context
- [ ] **X2. Examples for complex tasks** — If skill produces formatted output or uses tools non-obviously, examples are provided
- [ ] **X3. Evaluation scenarios exist** — 3 test cases (happy path, edge case, failure) are defined or can be generated
- [ ] **X4. Supporting files organized** — References in `references/`, examples in `examples/`, scripts in `scripts/`, assets in `assets/`

---

## Scoring Guide

- **18-22 Pass**: Skill is well-built, ready for use
- **13-17 Pass**: Functional but has room for improvement
- **Below 13 Pass**: Needs significant revision before deployment

## Common Failure Patterns

| Pattern | Symptom | Fix |
|---------|---------|-----|
| Stuffed context | SKILL.md over 500 lines | Move detail to `references/` |
| Weak description | Claude doesn't auto-trigger | Rewrite with trigger keywords, third person |
| Buried constraints | Claude violates rules mid-task | Move constraints to start or end of body |
| Linter delegation | Using LLM for deterministic tasks | Offload to scripts or tools |
| Multiple options | "Use X or Y or Z" | Pick one default, mention alternatives as escape hatch |
| Nested references | ref file loads another ref file | Flatten to one level |
| Missing negatives | Claude does things it shouldn't | Add explicit "Do NOT" statements |
| Vague bullets | "Handle errors appropriately" | Specify exactly how: "Log the error, skip the item, continue" |
