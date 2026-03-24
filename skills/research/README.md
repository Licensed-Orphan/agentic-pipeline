---
name: research
description: >
  Investigates any topic -- technical, product/market, domain, or exploratory --
  producing flexible, standalone research artifacts with evidence-based findings
  and actionable recommendations. Searches web, Slack, Confluence, Jira, Gmail,
  Gong, Snowflake, and Figma with adaptive depth calibration. Use when: "research",
  "investigate", "explore", "look into", "what's the best approach for", "deep dive
  on", or any task requiring evidence-based analysis. Not for pipeline-specific
  research feeding agentic-prd/architecture/implementation/execution -- use
  /agentic-research for that. Not for recalling prior research or answering
  quick factual questions.
---

# Research Engine

Produce evidence-based research artifacts adapted to the topic and use case.
Output format flexes to match what the research demands -- no fixed downstream
workflow assumed.

## Core Principles

1. **Search first, conclude second.** Always search and read sources BEFORE forming conclusions. Never write a finding then look for citations to justify it.
2. **Decompose into MECE sub-questions.** Break every research question into Mutually Exclusive, Collectively Exhaustive sub-questions. No overlapping coverage, no gaps.
3. **Lead with the answer.** Follow the Pyramid Principle: assertion first, grouped supporting arguments second, evidence third.
4. **Triangulate across source types.** Cross-reference findings across web, internal docs, data, and conversations. Convergence = high confidence. Divergence = a finding in itself.
5. **Adapt output to the question.** Structure the artifact around what the user needs to know, not a rigid template. A comparison needs a matrix. A landscape survey needs a map. A deep dive needs depth.

## Workflow

### Phase 1: Scoping

Before researching, establish scope. Ask scoping questions using AskUserQuestion
UNLESS the topic is already specific and well-defined.

**Scoping questions (pick 2-4 max, skip what's obvious):**

1. **Intent**: What will this research inform? A decision, a presentation, understanding a space, or something else?
2. **Known context**: What do you already know or assume? Any prior work to build on?
3. **Constraints**: Hard constraints? (timeline, technology, budget, compliance)
4. **Depth**: Broad landscape survey or deep on a narrow topic?
5. **Internal context**: Search internal sources (Slack, Confluence, Jira, Gong, Snowflake) or focus on external?
6. **Output preference**: Any specific format you want? (comparison table, narrative brief, decision doc, slide-ready bullets, etc.)

Skip scoping entirely when the topic is specific, the user provided clear context,
and there are no ambiguous constraints. State assumptions briefly instead.

See [references/scoping-guide.md](references/scoping-guide.md) for the full
question bank by research type.

### Phase 2: Planning & Wide Research

#### 2a. Research Plan

Decompose into MECE sub-questions. For each, identify:
- Best source types (web, Slack, Confluence, Gong, Snowflake, Figma, etc.)
- Specific searches/queries to run
- Expected output shape (comparison, evidence list, landscape map, etc.)

Apply intelligent source routing:
- **Web (WebSearch + WebFetch)**: External knowledge, papers, docs, analysis, competitive intel
- **Slack**: Internal discussions, decisions, tribal knowledge, recent context
- **Confluence**: Internal docs, design docs, past research, architecture decisions
- **Jira**: Active work, priorities, known issues, planned features
- **Gmail**: Stakeholder communications, vendor/partner context
- **Gong**: Customer voice, sales objections, pain points, feature requests
- **Snowflake**: Usage data, adoption metrics, quantitative evidence
- **Figma**: Design patterns, UI/UX decisions, design system components, prototypes

#### 2b. Execute Wide Research

Spawn parallel sub-agents for independent research threads. Each sub-agent:
- Focuses on one MECE sub-question
- Searches 2-3 relevant source types
- Returns a condensed summary (under 2,000 tokens) with inline citations
- Flags contradictions or surprising findings

Apply the SIFT method during research: Stop, Investigate the source, Find better
coverage, Trace back to originals.

See [references/source-evaluation.md](references/source-evaluation.md) for full
evaluation criteria and citation formatting.

### Phase 3: Mid-Research Checkpoint

Present a landscape map to the user:
- Most promising threads (1-2 sentence summaries)
- Surprising findings
- Contradictions detected
- Threads to deprioritize
- Proposed deep-dive plan

Wait for user confirmation before proceeding. The user may confirm, add threads,
remove threads, or redirect entirely.

### Phase 4: Deep Research

For each confirmed thread:
- Follow citation chains to primary sources
- Search for disconfirming evidence (not just confirming)
- Use ACH (Analysis of Competing Hypotheses) when sources conflict -- rate evidence as Consistent, Inconsistent, or Neutral against each hypothesis
- Track confidence levels per finding
- When sources conflict, investigate WHY (different contexts, methodologies, definitions) rather than picking a winner

See [references/methodology.md](references/methodology.md) for detailed frameworks.

**Stopping criteria:** Stop a thread when new searches return information already
captured, you have sufficient evidence for a recommendation, or marginal value
drops below marginal cost.

### Phase 5: Synthesis & Artifact Generation

Write the artifact to `./research/RESEARCH-[topic-slug].md`.

**Adapt the output structure to match the research.** Choose and combine sections
based on what the topic demands:

See [references/output-guide.md](references/output-guide.md) for the section
menu and formatting guidance.

**Required in every artifact:**
- YAML frontmatter (title, date, status, confidence, sources_consulted)
- Executive Summary (5-10 lines, Pyramid Principle: assertion → arguments → evidence)
- Key Findings with confidence levels (High/Medium/Low)
- Recommendations (actionable, prioritized, with confidence and evidence)
- Source Appendix (numbered citations with narrative context)

**Include when relevant (pick what fits):**
- Comparison matrix / trade-off table
- Landscape overview / market map
- Domain concepts and rules
- Constraints and boundaries
- Risk assessment
- Open questions
- Timeline / historical context
- Quantitative analysis

**Confidence scoring:**
- **High** -- multiple independent sources agree, primary evidence, triangulated
- **Medium** -- credible sources but limited corroboration, or secondary sources
- **Low** -- single source, conflicting evidence, or inference-based

### Phase 6: Refinement

After delivering the artifact, drive a structured refinement conversation:

1. Surface open questions and ask the user's perspective
2. Probe the 2-3 biggest assumptions
3. Clarify unresolved trade-offs
4. Check for missing coverage
5. Confirm recommendation priorities

Update the artifact after each round. When the user is satisfied, update the
frontmatter `status` from `draft` to `final`.

## Source Management

Use inline numbered references `[1]` mapping to the Source Appendix.

**Source tiers:**
- **Tier 1**: Official docs, peer-reviewed papers, primary data (Snowflake, Gong)
- **Tier 2**: Expert blogs, conference talks, well-sourced journalism, Confluence docs
- **Tier 3**: Community discussions, marketing materials, undated/anonymous content

Always note tiers. If a finding rests entirely on Tier 3 sources, flag it explicitly.

See [references/source-evaluation.md](references/source-evaluation.md) for full
citation format and tier classification.

## Constraints

- Do NOT produce pipeline-aware output sections (no "→ Downstream consumer" annotations, no pipeline handoff checklists) -- use /agentic-research for that
- Do NOT follow a rigid template -- adapt structure to the topic
- Do NOT dump everything found -- synthesize ruthlessly, every paragraph earns its place
- Do NOT write findings then look for citations -- always search first
- Do NOT silently pick a winner when sources conflict -- present both sides
- Do NOT let a Tier 3 source be the sole basis for a recommendation

## Critical Rules (Reinforcement)

- **Search THEN conclude.** Never reverse this order.
- **Every claim needs a citation.** Flag unsourced claims as inference with confidence level.
- **Adapt when evidence demands it.** Pivot if early findings invalidate the plan.
- **Present conflicts, don't hide them.** Investigate why sources disagree and present both sides.
- **Match output to the question.** A comparison question gets a comparison. A landscape question gets a map. Never force-fit a template.
