# Research Output Guide

Adapt the artifact structure to match what the research demands. This is a menu
of sections -- pick and combine based on the topic, not a fixed template.

## Frontmatter (Always Required)

```yaml
---
title: "[Research Topic]"
date: "[YYYY-MM-DD]"
status: "draft" # draft | final
research_type: "[technical | product | domain | exploratory | blend]"
confidence: "medium" # overall: high | medium | low
topic_slug: "[kebab-case-slug]"
sources_consulted: [N]
internal_sources_used: [] # e.g., ["slack", "confluence", "gong", "snowflake"]
---
```

## Required Sections

### Executive Summary

5-10 lines maximum. Lead with the answer (Pyramid Principle).

```markdown
## Executive Summary

[Assertion: the bottom-line answer to the research question]

**Key findings:**
1. [Finding] (Confidence: HIGH/MEDIUM/LOW)
2. [Finding] (Confidence: HIGH/MEDIUM/LOW)
3. [Finding] (Confidence: HIGH/MEDIUM/LOW)

**Recommended path forward:** [1-2 sentences]
```

### Recommendations

Ordered by priority. Each recommendation is actionable.

```markdown
## Recommendations

1. **[Recommendation]** (Confidence: HIGH)
   - Evidence: [summary with citation numbers]
   - Impact: [what this enables or prevents]

2. **[Recommendation]** (Confidence: MEDIUM)
   - Evidence: [summary]
   - Impact: [what this enables]
```

### Source Appendix

```markdown
## Source Appendix

**Sources:** [N] total ([N] Tier 1, [N] Tier 2, [N] Tier 3)

[1] **Title** -- Author/Organization (Date)
    URL: [url]
    Context: [1-2 sentences: what this source contributes and why it's credible]
    Tier: [1/2/3]
```

## Optional Sections (Pick What Fits)

### Comparison Matrix

Use when evaluating options, technologies, or approaches side-by-side.

```markdown
## Comparison

| Criterion | Option A | Option B | Option C |
|-----------|----------|----------|----------|
| [Criterion] | [Rating + notes] | [Rating + notes] | [Rating + notes] |

**Recommendation**: [Which option and why]
```

### Landscape Overview

Use for broad surveys, market maps, or competitive analysis.

```markdown
## Landscape

[2-3 paragraph narrative of the space. Key players, dominant approaches, gaps.]

### Key Players

| Player | Approach | Strengths | Weaknesses |
|--------|----------|-----------|------------|
| [Name] | [How] | [What's good] | [What's weak] |
```

### Domain Concepts

Use when the research covers a specialized domain the reader needs context for.

```markdown
## Key Concepts

| Concept | Definition | Relevance |
|---------|-----------|-----------|
| [Term] | [What it means] | [Why it matters here] |

### Domain Rules
- [Rule]: [Explanation and source]
```

### Constraints & Boundaries

Use when the research uncovers hard limits, compliance requirements, or guardrails.

```markdown
## Constraints

**Hard constraints:**
- MUST [constraint] -- [source]
- MUST NOT [constraint] -- [source]

**Recommendations:**
- SHOULD [recommendation] -- [source, confidence]
```

### Risk Assessment

Use when the research reveals risks that need to be managed.

```markdown
## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| [Risk] | High/Med/Low | High/Med/Low | [How to mitigate] |
```

### Open Questions

Use when the research surfaced questions it couldn't fully answer.

```markdown
## Open Questions

1. **[Question]**
   - Why it matters: [impact on decisions]
   - Best available answer: [what the research suggests]
   - How to resolve: [suggested approach]
```

### Quantitative Analysis

Use when Snowflake data, metrics, or numerical evidence is central.

```markdown
## Data Analysis

### [Analysis Name]
- **Query/Method**: [what was measured]
- **Key finding**: [the number and what it means]
- **Implication**: [so what]
```

### Timeline / Historical Context

Use when understanding how something evolved matters for the decision.

```markdown
## Timeline

| Date | Event | Significance |
|------|-------|-------------|
| [Date] | [What happened] | [Why it matters] |
```

## Formatting Rules

- Use Markdown tables for structured comparisons (never free-form lists)
- Use numbered citations `[1]` inline, full entries in Source Appendix
- Confidence levels on every finding: HIGH, MEDIUM, or LOW
- Keep sections concise -- if a section adds fewer than 3 sentences of value, fold it into another section or omit it
- Source Appendix entries must include narrative context, not just raw quotes
