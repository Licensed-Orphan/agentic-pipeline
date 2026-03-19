# Research Artifact Template

Use this template for all research output files. Adapt section depth based on
research complexity -- narrow technical evaluations may skip competitive analysis,
pure market research may have a lighter technical landscape section.

The key invariant: every section maps to a downstream consumer in the agentic
pipeline. Don't remove sections; mark them "N/A -- [reason]" if not applicable.

---

```markdown
---
title: "[Research Topic]"
date: "[YYYY-MM-DD]"
status: "draft" # draft | refined
research_types:
  - technical    # remove if not applicable
  - product      # remove if not applicable
  - domain       # remove if not applicable
confidence: "medium" # overall: high | medium | low
topic_slug: "[kebab-case-slug]"
sources_consulted: [N]
internal_sources_used: [] # e.g., ["slack", "confluence", "gong", "snowflake"]
---

# Research: [Topic Name]

## Executive Summary

[5-10 lines maximum. Lead with the answer -- what did the research conclude?
Follow the Pyramid Principle: assertion first, then the 2-3 key supporting
arguments. This section alone should give a reader enough to decide whether
to proceed to PRD drafting.]

**Bottom line:** [One sentence -- the single most important takeaway]

**Key findings:**
1. [Finding with confidence level: HIGH/MEDIUM/LOW]
2. [Finding with confidence level]
3. [Finding with confidence level]

**Recommended path forward:** [1-2 sentences on what to build/do based on this research]

---

## Problem Space
*→ Downstream consumer: agentic-prd (problem statement, user stories, non-goals)*

### Problem Definition

[What problem does this research address? Frame as a job-to-be-done: what is
the user/customer/system trying to accomplish, and what obstacles exist?]

### Who Is Affected

[Identify the stakeholders, users, or systems impacted. Include roles,
responsibilities, and relationships.]

### Current State

[How is this problem handled today? What tools, processes, or workarounds
exist? What's broken or suboptimal about them?]

### Impact & Urgency

[Why does this matter now? What's the cost of inaction? Include quantitative
data from Snowflake, Gong, or other sources when available.]

### Jobs-to-be-Done

| Job Statement | Current Solution | Pain Points |
|--------------|-----------------|-------------|
| [When I..., I want to..., so I can...] | [How it's done today] | [What's broken] |

---

## Technical Landscape
*→ Downstream consumer: agentic-architecture (technology choices, patterns, constraints)*

### Approaches Evaluated

[For each approach considered, use MADR-compatible structure:]

#### Option A: [Approach Name]

- **Description**: [What it is, how it works]
- **Decision drivers**: [What factors favor this approach]
- **Good, because**: [Advantage 1]
- **Good, because**: [Advantage 2]
- **Bad, because**: [Disadvantage 1]
- **Bad, because**: [Disadvantage 2]
- **Confidence**: [HIGH/MEDIUM/LOW]
- **Sources**: [citation numbers]

#### Option B: [Approach Name]

[Same structure]

#### Option C: [Approach Name]

[Same structure]

### Technology Radar

[Classify technologies/approaches using ThoughtWorks ring system:]

| Technology/Approach | Ring | Rationale |
|--------------------|------|-----------|
| [Name] | ADOPT | [Proven, recommended for production use] |
| [Name] | TRIAL | [Worth pursuing, not fully proven at scale] |
| [Name] | ASSESS | [Worth exploring, understand impact] |
| [Name] | HOLD | [Proceed with caution, better alternatives exist] |

### Trade-off Matrix

| Criterion | Option A | Option B | Option C |
|-----------|----------|----------|----------|
| [Criterion 1] | [Rating + notes] | [Rating + notes] | [Rating + notes] |
| [Criterion 2] | [Rating + notes] | [Rating + notes] | [Rating + notes] |
| [Criterion 3] | [Rating + notes] | [Rating + notes] | [Rating + notes] |

### Recommended Technical Direction

**Recommendation**: [Which option and why -- reference the trade-off matrix]
**Confidence**: [HIGH/MEDIUM/LOW]
**Key assumption**: [What must be true for this recommendation to hold]

---

## Prior Art & Competitive Analysis
*→ Downstream consumer: agentic-prd (positioning, feature scope, differentiation)*

### Landscape Overview

[2-3 paragraph narrative of the competitive/prior art landscape. Who are the
key players? What approaches dominate? Where are the gaps?]

### Detailed Comparison

| Dimension | [Competitor/Solution A] | [Competitor/Solution B] | [Competitor/Solution C] |
|-----------|----------------------|----------------------|----------------------|
| Approach | | | |
| Strengths | | | |
| Weaknesses | | | |
| Pricing/Cost | | | |
| Target audience | | | |
| Key differentiator | | | |

### Patterns Worth Adopting

[What good ideas from existing solutions should inform our approach?]

- **[Pattern]**: [Where it's used, why it works, how to adapt it]

### Patterns to Avoid

[What anti-patterns or failures did the research uncover?]

- **[Anti-pattern]**: [Where it failed, why, what to do instead]

### Differentiation Opportunity

[Where is the gap in the market or the unmet need that existing solutions
don't address? This is the strategic insight that should drive PRD scope.]

---

## Domain Insights
*→ Downstream consumer: agentic-prd (requirements), agentic-architecture (domain model)*

### Key Concepts

[Define the core domain concepts a developer needs to understand. These will
inform the domain model in the architecture.]

| Concept | Definition | Relevance |
|---------|-----------|-----------|
| [Term] | [What it means] | [Why it matters for this project] |

### Domain Rules & Invariants

[Rules that must always hold true in this domain. These become hard constraints
in the architecture.]

- [Rule]: [Explanation and source]

### Industry Standards & Compliance

[Any standards, regulations, or compliance requirements that apply.]

- [Standard]: [What it requires, how it affects implementation]

---

## Constraints & Boundaries
*→ Downstream consumer: agentic-prd (boundaries), agentic-architecture (constraints)*

### Hard Constraints (MUST / MUST NOT)

[Non-negotiable constraints discovered through research. Use RFC 2119 keywords.]

- The system MUST [constraint] -- [source/evidence]
- The system MUST NOT [constraint] -- [source/evidence]

### Strong Recommendations (SHOULD / SHOULD NOT)

[Evidence-backed recommendations that could be overridden with good reason.]

- The system SHOULD [recommendation] -- [source/evidence, confidence level]
- The system SHOULD NOT [recommendation] -- [source/evidence, confidence level]

### Options (MAY)

[Approaches that are valid but optional, depending on scope and priorities.]

- The system MAY [option] -- [when this makes sense, trade-offs]

### Boundary Classification

| Boundary | Type | Rationale |
|----------|------|-----------|
| [Boundary] | Always | [Why this must always be enforced] |
| [Boundary] | Ask First | [Why this needs human judgment] |
| [Boundary] | Never | [Why this is categorically prohibited] |

---

## Risk Register
*→ Downstream consumer: agentic-implementation (recovery procedures), agentic-execution (adaptive replanning)*

| Risk | Likelihood | Impact | Mitigation | Detection |
|------|-----------|--------|------------|-----------|
| [Risk description] | High/Med/Low | High/Med/Low | [How to prevent or reduce] | [How to detect if it materializes] |

---

## Recommendations
*→ Downstream consumer: agentic-prd (phase structure, feature priorities)*

### Synthesized Recommendations

[Ordered by priority. Each recommendation should be actionable -- something
that translates into a PRD requirement or phase.]

1. **[Recommendation]** (Confidence: HIGH)
   - Evidence: [summary of supporting evidence with citation numbers]
   - Impact: [what this enables or prevents]
   - Effort signal: [Small/Medium/Large -- rough sense of implementation scope]

2. **[Recommendation]** (Confidence: MEDIUM)
   - Evidence: [summary]
   - Impact: [what this enables]
   - Effort signal: [S/M/L]

3. **[Recommendation]** (Confidence: LOW -- needs validation)
   - Evidence: [summary]
   - Impact: [what this enables]
   - What would increase confidence: [specific validation needed]

### Unresolved Tensions

[Trade-offs where the research found legitimate arguments on both sides.
These MUST be resolved during PRD drafting.]

- **[Tension]**: [Side A] vs [Side B]
  - Factors favoring A: [evidence]
  - Factors favoring B: [evidence]
  - Resolution depends on: [what the user/team needs to decide]

---

## Open Questions
*→ Downstream consumer: agentic-prd (interview phase -- these become questions to resolve before specifying)*

[Questions the research could not answer definitively. Each should include
why it matters and what resolving it would change.]

1. **[Question]**
   - Why it matters: [impact on downstream decisions]
   - What the research suggests: [best available answer, if any]
   - How to resolve: [suggested approach -- user decision, prototype, data query, etc.]

---

## Source Appendix

### Source Summary

- Total sources consulted: [N]
- Tier 1 sources: [N]
- Tier 2 sources: [N]
- Tier 3 sources: [N]
- Internal sources: [list which tools were queried]

### Full Citations

[1] **[Title]** -- [Author/Organization] ([Date])
    URL: [url]
    Context: [1-2 sentences: what this source contributes to the research and
    why it's credible. Include relevant qualifications of the author or
    authority of the organization.]
    Tier: [1/2/3]
    Used in: [which sections reference this source]

[2] **[Title]** -- [Author/Organization] ([Date])
    URL: [url]
    Context: [narrative context]
    Tier: [1/2/3]
    Used in: [sections]

[Continue for all sources...]
```
