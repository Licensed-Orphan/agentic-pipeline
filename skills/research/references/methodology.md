# Research Methodology Patterns

Mental models for conducting research. Use as frameworks, not rigid procedures.

## MECE Decomposition

Break every research topic into Mutually Exclusive, Collectively Exhaustive
sub-questions before searching.

**Process:**
1. State the overarching research question
2. Identify 3-7 sub-questions that together fully answer it
3. Verify mutual exclusivity: no two sub-questions overlap
4. Verify collective exhaustiveness: answering all fully answers the parent
5. For each sub-question, identify the best source types

**Example:**
```
Research question: "What's the best approach for real-time collaboration in our editor?"

Sub-questions (MECE):
1. What are the fundamental algorithms? (domain) → papers, docs, web
2. What production systems use each approach? (prior art) → web, blogs, case studies
3. What are performance/scalability characteristics? (technical) → benchmarks, papers
4. What libraries exist in our tech stack? (technical) → web, GitHub, package registries
5. What do our users expect? (product) → Gong, support tickets, Slack
6. What are infrastructure costs? (constraints) → docs, pricing, Snowflake
```

## The Pyramid Principle

Structure all output top-down:

```
Level 1: ASSERTION (the answer)
  ├── Level 2: ARGUMENT 1 (supporting reason)
  │     ├── Level 3: EVIDENCE (data, sources, examples)
  │     └── Level 3: EVIDENCE
  ├── Level 2: ARGUMENT 2 (supporting reason)
  │     └── Level 3: EVIDENCE
  └── Level 2: ARGUMENT 3 (supporting reason)
        └── Level 3: EVIDENCE
```

**Rules:**
- Ideas at any level summarize the ideas grouped below them
- Ideas in each group are MECE
- Ideas in each group are logically ordered (time, structure, or importance)

**Anti-pattern:** Bottom-up information dumping -- presenting all evidence
chronologically and letting the reader form conclusions. Actively resist this.

## Analysis of Competing Hypotheses (ACH)

When evidence points in multiple directions:

1. List all plausible hypotheses
2. List all significant evidence
3. Rate each piece of evidence against EACH hypothesis:
   - **C** = Consistent | **I** = Inconsistent | **N** = Neutral
4. Focus on INCONSISTENCIES -- fewest I ratings = most defensible hypothesis
5. Identify which evidence is most diagnostic (inconsistent with the most hypotheses)

**Why this works:** Humans and LLMs naturally seek confirming evidence. ACH forces
attention to disconfirming evidence, which is far more informative.

## Source Routing

Route queries to the most appropriate source types:

| Signal Needed | Primary Source | Secondary Source | Avoid |
|--------------|---------------|-----------------|-------|
| How technology X works | Official docs, papers | Expert blogs | Marketing, Reddit |
| What customers want | Gong, support tickets | Slack #feedback | Internal assumptions |
| What competitors do | Product demos, their docs | Industry analysis | Their marketing claims |
| Usage/adoption data | Snowflake, analytics | Internal dashboards | Anecdotal reports |
| Past internal decisions | Confluence, ADRs | Slack threads | Individual memory |
| Active work/priorities | Jira, Slack | Confluence roadmaps | Stale planning docs |
| Industry trends | Research papers, analysts | Conference talks | Hype articles |
| Implementation patterns | GitHub repos, docs | Stack Overflow, blogs | Tutorial sites |
| Regulatory requirements | Official regulatory texts | Legal team | Secondary summaries |
| UI/UX patterns | Figma files, design system | Competitor screenshots | Marketing mockups |

## The Funnel Technique

Start broad, then narrow:

```
Phase 1: WIDE SCAN → survey full space, identify categories, note surprises, flag contradictions
Phase 2: CHECKPOINT → present landscape map, propose threads, get user confirmation
Phase 3: DEEP DIVE → follow citations, seek disconfirming evidence, build hypothesis matrix
Phase 4: SYNTHESIS → apply Pyramid Principle, resolve conflicts via ACH, score confidence
```

## When to Stop (Information Foraging Theory)

**Stop a thread when:**
- New searches return information already captured (saturation)
- 3+ independent sources support the finding (sufficient evidence)
- Finding is clear enough for a recommendation with stated confidence
- Further research adds precision but won't change direction

**Keep going when:**
- Key claims rest on a single source
- Major contradictions remain unresolved
- The user's core question hasn't been answered
- A promising thread was identified but not followed

**Pivot when:**
- Early findings invalidate the research plan
- A sub-question turned out to be the wrong question
- User feedback at checkpoint redirects the research
