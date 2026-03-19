# Research Methodology Patterns

This file documents the research methodologies and frameworks the research
skill draws from. Use these as mental models during research, not as rigid
procedures.

## Research Decomposition: MECE Sub-Questions

Every research topic should be decomposed into Mutually Exclusive, Collectively
Exhaustive sub-questions before any searching begins.

**Process:**
1. State the overarching research question
2. Identify 3-7 sub-questions that, together, fully answer it
3. Verify mutual exclusivity: no two sub-questions overlap in scope
4. Verify collective exhaustiveness: answering all sub-questions fully answers
   the parent question
5. For each sub-question, identify the best source types

**Example:**
```
Research question: "What's the best approach for real-time collaboration in our editor?"

Sub-questions (MECE):
1. What are the fundamental algorithms for real-time collaboration? (domain)
   → Sources: academic papers, documentation, web
2. What production systems use each approach and what are their lessons? (prior art)
   → Sources: web, engineering blogs, case studies
3. What are the performance/scalability characteristics of each approach? (technical)
   → Sources: benchmarks, papers, production data
4. What libraries/frameworks implement each approach in our tech stack? (technical)
   → Sources: web, GitHub, npm/package registries
5. What do our users expect from real-time collaboration? (product)
   → Sources: Gong calls, support tickets, Slack discussions
6. What are the infrastructure requirements and cost implications? (constraints)
   → Sources: documentation, pricing pages, Snowflake (current infra usage)
```

## Output Structure: The Pyramid Principle

Structure all research output top-down:

```
Level 1: ASSERTION (the answer)
  ├── Level 2: ARGUMENT 1 (supporting reason)
  │     ├── Level 3: EVIDENCE (data, sources, examples)
  │     └── Level 3: EVIDENCE
  ├── Level 2: ARGUMENT 2 (supporting reason)
  │     ├── Level 3: EVIDENCE
  │     └── Level 3: EVIDENCE
  └── Level 2: ARGUMENT 3 (supporting reason)
        └── Level 3: EVIDENCE
```

**Rules:**
- Ideas at any level must be summaries of the ideas grouped below them
- Ideas in each group must be MECE
- Ideas in each group must be logically ordered (time, structure, or importance)

**Anti-pattern:** Bottom-up information dumping -- presenting all evidence
chronologically and letting the reader form their own conclusions. This is
the natural tendency of research agents and must be actively resisted.

## Conflict Resolution: Analysis of Competing Hypotheses (ACH)

When evidence points in multiple directions, use ACH rather than picking the
most intuitive answer.

**Process:**
1. List all plausible hypotheses (not just the two most obvious)
2. List all significant evidence and arguments
3. For each piece of evidence, rate it against EACH hypothesis:
   - **C** = Consistent (supports the hypothesis)
   - **I** = Inconsistent (contradicts the hypothesis)
   - **N** = Neutral (neither supports nor contradicts)
4. Focus on INCONSISTENCIES -- the hypothesis with the fewest I ratings is
   the most defensible
5. Identify which evidence is most diagnostic (inconsistent with the most
   hypotheses)

**Why this works:** Humans (and LLMs) naturally seek confirming evidence.
ACH forces attention to disconfirming evidence, which is far more informative.
A finding that's consistent with 3 hypotheses tells you little. A finding
that's inconsistent with 2 of 3 hypotheses is highly diagnostic.

**When to use:** Whenever you have competing technical approaches, conflicting
market signals, or disagreement between sources about best practices.

## Source Routing: Intelligent Query Distribution

Route research queries to the most appropriate source types:

### Decision Matrix

| Signal Needed | Primary Source | Secondary Source | Avoid |
|--------------|---------------|-----------------|-------|
| How technology X works | Official docs, papers | Expert blogs | Marketing, Reddit |
| What customers want | Gong calls, support tickets | Slack #customer-feedback | Internal assumptions |
| What competitors do | Product demos, their docs | Industry analysis | Their marketing claims |
| How much usage/adoption | Snowflake, analytics | Internal dashboards | Anecdotal reports |
| Past internal decisions | Confluence, ADRs | Slack threads | Individual memory |
| Active work/priorities | Jira, Slack | Confluence roadmaps | Stale planning docs |
| Industry trends | Research papers, analyst reports | Conference talks | Hype articles |
| Implementation patterns | GitHub repos, docs | Stack Overflow, blogs | Tutorial sites |
| Regulatory requirements | Official regulatory texts | Legal team (Slack/email) | Secondary summaries |

### Query Formulation by Source

**WebSearch:** Include current date. Be specific. Use quotes for exact phrases.
- Good: `"CRDT vs OT" collaborative editing production experience 2025`
- Bad: `real time collaboration`

**Slack:** Search for channel discussions, decisions, mentions of the topic.
- Search keywords related to the domain, technology names, project names
- Check #engineering, #product, #architecture channels

**Confluence:** Search for design docs, ADRs, past research, technical specs.
- Look for pages in relevant spaces
- Check for existing research that can be built upon

**Gong:** Search for customer calls mentioning relevant pain points or features.
- Focus on actual customer language and specific examples
- Look for patterns across multiple calls, not single anecdotes

**Snowflake:** Query for quantitative evidence (usage data, adoption metrics).
- Use the connection details and schema guide from project memory
- Focus on aggregate patterns, not individual data points

## Adaptive Research: The Funnel Technique

Start broad, then narrow based on what you discover:

```
Phase 1: WIDE SCAN (landscape)
├── Survey the full space
├── Identify major categories/approaches
├── Note surprising findings
└── Flag contradictions

Phase 2: CHECKPOINT (with user)
├── Present landscape map
├── Propose which threads to pursue
└── Get user confirmation/redirection

Phase 3: DEEP DIVE (focused)
├── Follow citation chains
├── Seek disconfirming evidence
├── Build hypothesis matrix
└── Track confidence levels

Phase 4: SYNTHESIS (convergence)
├── Apply Pyramid Principle
├── Resolve conflicts via ACH
├── Score confidence per finding
└── Flag open questions
```

## Research Termination: When to Stop

Apply information foraging theory -- stop when marginal value drops below
marginal cost:

**Signals to stop a research thread:**
- New searches return information already captured (saturation)
- You have 3+ independent sources supporting the finding (sufficient evidence)
- The finding is clear enough to support a recommendation with stated confidence
- Further research would only add precision, not change the direction

**Signals to keep going:**
- Key claims rest on a single source (insufficient triangulation)
- Major contradictions remain unresolved
- The user's core question hasn't been answered
- A promising thread was identified but not followed

**Signals to pivot:**
- Early findings invalidate the research plan (adapt, don't persist)
- A sub-question turned out to be the wrong question
- User feedback at the checkpoint redirects the research

## Pipeline Handoff Checklist

Before finalizing the research artifact, verify these pipeline-aware requirements:

### For agentic-prd consumption:
- [ ] Problem Space section frames the problem as a job-to-be-done
- [ ] Constraints include explicit non-goals and boundaries
- [ ] Recommendations are prioritized and actionable
- [ ] Open questions are framed as things to resolve before specifying

### For agentic-architecture consumption:
- [ ] Technical Landscape includes MADR-compatible option evaluations
- [ ] Trade-off matrix covers relevant architectural criteria
- [ ] Domain insights include rules/invariants that constrain the architecture
- [ ] Hard constraints use RFC 2119 keywords (MUST/MUST NOT)

### For agentic-implementation consumption:
- [ ] Risk register includes detection and mitigation strategies
- [ ] Effort signals accompany recommendations (S/M/L)
- [ ] Constraints are specific enough to enforce (not vague principles)

### For agentic-execution consumption:
- [ ] Risks are tagged with likelihood and impact
- [ ] Known failure modes are documented with recovery approaches
- [ ] Performance/scale constraints include specific thresholds where known
