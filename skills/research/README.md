---
name: research
description: >
  Deep research skill for investigating any topic -- technical, product/market,
  or domain -- producing structured, pipeline-aware research artifacts optimized
  for feeding into the agentic PRD → Architecture → Implementation → Execution
  pipeline. Supports web search, internal tools (Slack, Confluence, Jira, Gmail,
  Gong, Snowflake, Figma), and adaptive depth calibration. Use when: researching a
  topic before building, investigating technical approaches, exploring a problem
  space, conducting competitive analysis, or any task prefixed with "research".
  Triggers on phrases like "research", "investigate", "explore", "look into",
  "what's the best approach for", "deep dive on".
license: MIT
metadata:
  version: "1.0"
  author: wesleydubose
---

# Agentic Research Engine

Produce structured, evidence-based research artifacts that serve as the genesis
of the agentic development pipeline. Research outputs are optimized for downstream
consumption by the agentic-prd, agentic-architecture, agentic-implementation,
and agentic-execution skills.

## Core Principles

1. **Retrieval-first reasoning** -- derive claims FROM sources, never generate
   claims then find citations to justify them. Up to 57% of LLM citations are
   post-rationalized. Combat this by searching first, reading results, THEN
   forming conclusions.
2. **MECE decomposition** -- break every research question into Mutually Exclusive,
   Collectively Exhaustive sub-questions. No overlapping coverage, no gaps.
3. **Pyramid Principle output** -- lead with the answer/insight, support with
   grouped arguments, back with evidence. Conclusions first, details on demand.
4. **Triangulation** -- cross-reference findings across source types (web, internal
   docs, data, conversations). Convergence = high confidence. Divergence = a
   finding in itself.
5. **Progressive disclosure** -- summary (5-10 lines) → key findings with
   recommendations → detailed evidence and analysis → source appendix.
6. **Pipeline awareness** -- structure output sections to map directly to what
   downstream agentic skills need.

## Workflow

### Phase 1: Scoping (Guided by Default)

Before researching, establish scope. Ask the user scoping questions using the
AskUserQuestion tool UNLESS the topic is specific enough that scoping would be
redundant (e.g., "research CRDTs vs OT for collaborative editing" is already
well-scoped; "research indoor positioning" needs scoping).

**Determine research type(s) by analyzing the topic:**
- **Technical** -- evaluating technologies, frameworks, architectures, protocols
- **Product/Market** -- competitive landscape, user needs, market dynamics
- **Domain** -- understanding a problem space, science, industry mechanics
- **Blend** -- most real topics combine multiple types

**Scoping questions to ask (adapt based on what's already clear):**

1. **Intent**: What decision or action will this research inform? Are you building
   something, evaluating options, or understanding a space?
2. **Audience**: Who will consume this beyond the agentic pipeline? (Just you?
   A team? Stakeholders?)
3. **Known context**: What do you already know or assume? What prior work exists?
4. **Constraints**: Are there hard constraints (timeline, technology mandates,
   budget, compliance requirements)?
5. **Depth preference**: Should I go broad (landscape survey) or deep (exhaustive
   on a narrow topic)?
6. **Internal context**: Should I search internal sources (Slack, Confluence,
   Jira, Gong, Snowflake) for relevant context, or focus on external research?

Do NOT ask questions whose answers are obvious from the topic. Skip redundant
questions. Aim for 2-4 questions maximum. If the user provided detailed context
upfront, you may skip scoping entirely and proceed to planning.

See [references/scoping-guide.md](references/scoping-guide.md) for the full
scoping question bank organized by research type.

### Phase 2: Planning & Wide Research

After scoping, execute in two sub-phases:

#### 2a. Generate Research Plan

Decompose the topic into MECE sub-questions. For each sub-question, identify:
- Which source types are most relevant (web, Slack, Confluence, Gong, Snowflake, etc.)
- What specific searches/queries to run
- What the expected output is (landscape map, comparison matrix, evidence list, etc.)

Apply intelligent source routing:
- **Web (WebSearch + WebFetch)**: External knowledge, academic papers, documentation,
  blog posts, industry analysis, competitive intelligence
- **Slack**: Internal discussions, decisions, tribal knowledge, recent context
- **Confluence**: Internal documentation, design docs, past research, architecture decisions
- **Jira**: Active work, priorities, known issues, planned features
- **Gmail**: Stakeholder communications, external vendor/partner context
- **Gong**: Customer voice, sales objections, pain points, feature requests
- **Snowflake**: Usage data, adoption metrics, quantitative evidence
- **Figma**: Existing design patterns, UI/UX decisions, design system components,
  competitive design references, wireframes, and prototypes. Use get_design_context
  and get_screenshot to extract design intent from Figma files the user references

#### 2b. Execute Wide Research

Run parallel research across sub-questions using the Agent tool to spawn
sub-agents for independent research threads. Each sub-agent should:
- Focus on one MECE sub-question
- Search 2-3 source types relevant to that sub-question
- Return a condensed summary (under 2,000 tokens) with inline citations
- Flag any contradictions or surprising findings

**Source evaluation during research -- apply the SIFT method:**
1. **Stop** -- don't accept claims at face value
2. **Investigate the source** -- search for what others say about the source
3. **Find better coverage** -- look for corroboration from independent sources
4. **Trace back** -- find the original/primary source of claims

**For technical research**, prioritize:
- Official documentation over blog posts
- Primary papers over summaries
- Benchmarks and data over opinions
- Recent sources (include current date in searches)

**For product/market research**, prioritize:
- Direct customer voice (Gong, support tickets) over analyst reports
- Quantitative data (Snowflake, usage metrics) over qualitative impressions
- Competitive products' actual behavior over marketing claims
- Existing Figma designs and prototypes for understanding current UX patterns,
  design decisions, and design system components

**For domain research**, prioritize:
- Academic papers and textbooks over popularizations
- Expert practitioners over generalists
- Multiple perspectives over single authoritative sources

### Phase 3: Mid-Research Checkpoint

After wide research completes, present the user with a **landscape map**:

```
## Research Landscape

I've completed the initial survey. Here's what I found across [N] sub-questions:

### Most Promising Threads
1. **[Thread name]** -- [1-2 sentence summary, why it's promising]
2. **[Thread name]** -- [1-2 sentence summary, why it's promising]
3. **[Thread name]** -- [1-2 sentence summary, why it's promising]

### Surprising Findings
- [Anything unexpected that emerged]

### Contradictions Detected
- [Source A says X, but Source B says Y -- worth investigating]

### Threads I'd Deprioritize
- [Thread name] -- [why it's less relevant than initially expected]

### Proposed Deep-Dive Plan
I recommend going deep on threads [1, 2, 3]. Should I proceed, or would you
like me to reprioritize?
```

Wait for user confirmation before proceeding to deep research. The user may:
- Confirm the plan as-is
- Add threads they want explored
- Remove threads they don't care about
- Redirect the research entirely based on what surfaced

### Phase 4: Deep Research

For each confirmed thread, conduct exhaustive research:

- Follow citation chains -- when a source references another source, fetch the original
- Search for disconfirming evidence (not just confirming evidence)
- Maintain a hypothesis matrix: rate evidence as **Consistent (C)**, **Inconsistent (I)**,
  or **Neutral (N)** against competing hypotheses (Analysis of Competing Hypotheses method)
- Track confidence levels: how well-supported is each finding?
- When sources conflict, investigate WHY they disagree (different contexts, methodologies,
  definitions, timeframes) rather than picking a winner

**Conflict resolution framework:**
- If evidence clearly favors one side → synthesize a recommendation with confidence level
- If genuinely ambiguous → present the tension explicitly with context for each position
- If the "right answer" depends on constraints only the user knows → flag as an
  open question for the refinement phase

**Stopping criteria (information foraging theory):**
Stop researching a thread when:
- New searches return information you've already captured (diminishing returns)
- You have enough evidence to support a recommendation with stated confidence
- The marginal value of another search is lower than the cost of additional context

### Phase 5: Synthesis & Artifact Generation

Compile all findings into a structured research artifact. Write the file to
`./research/RESEARCH-[topic-slug].md` using the output template.

See [references/output-template.md](references/output-template.md) for the
complete template with all pipeline-aware sections.

**Output structure (pipeline-aware):**

```
YAML frontmatter (metadata for machine consumption)
├── Executive Summary (5-10 lines -- the Pyramid Principle top)
├── Problem Space (→ feeds PRD problem statement & user stories)
├── Technical Landscape (→ feeds Architecture skill decisions)
│   ├── Approaches Evaluated (MADR-compatible)
│   ├── Technology Options (Adopt/Trial/Assess/Hold classification)
│   └── Trade-off Matrix
├── Prior Art & Competitive Analysis (→ feeds PRD positioning)
├── Domain Insights (→ feeds PRD requirements & Architecture constraints)
├── Constraints & Boundaries (→ feeds PRD boundaries, Architecture constraints)
│   ├── Hard Constraints (MUST / MUST NOT -- RFC 2119)
│   ├── Recommendations (SHOULD / SHOULD NOT)
│   └── Options (MAY)
├── Risk Register (→ feeds Implementation recovery procedures)
├── Recommendations (→ feeds PRD phase structure)
│   ├── Synthesized recommendations with confidence levels
│   └── Open questions requiring human resolution
├── Open Questions (→ feeds PRD interview phase)
└── Source Appendix (layered citations with narrative context)
```

**Confidence scoring per finding:**
- **High** -- multiple independent sources agree, primary evidence available,
  verified through triangulation
- **Medium** -- supported by credible sources but limited corroboration, or
  sources are secondary
- **Low** -- single source, or conflicting evidence with no clear resolution,
  or based on inference rather than direct evidence

### Phase 6: Refinement Conversation

After delivering the artifact, initiate a structured refinement dialogue.
Do NOT just ask "does this look good?" Instead, proactively identify the
areas most likely to need clarification:

1. **Surface open questions** -- present each open question from the research
   and ask the user's perspective
2. **Probe assumptions** -- identify the 2-3 biggest assumptions in the research
   and ask the user to validate or challenge them
3. **Clarify trade-offs** -- for each unresolved tension, present the trade-off
   and ask the user's preference
4. **Check completeness** -- ask if there are aspects of the problem the user
   expected to see covered that are missing
5. **Confirm priority** -- ask the user to rank the recommendations by importance

After each round of feedback, update the research artifact file. Continue the
conversation until the user indicates they're satisfied and ready to proceed
to PRD drafting.

When refinement is complete, update the YAML frontmatter `status` field from
`draft` to `refined`.

## Source Management

### Citation Format (Inline + Appendix)

Use inline numbered references `[1]` in the body text. Each number maps to
a full entry in the Source Appendix.

**Appendix entry format:**
```
[N] **Title** -- Author/Organization (Date)
    URL: [url]
    Context: [1-2 sentences explaining what this source contributes and why
    it's credible -- not just a raw quote]
    Confidence: [High/Medium/Low based on SIFT evaluation]
```

### Source Quality Tiers

Tier 1 (highest weight):
- Official documentation and specifications
- Peer-reviewed papers and published research
- Primary data (your own Snowflake queries, Gong transcripts)
- Authoritative industry sources (Anthropic blog, official framework docs)

Tier 2:
- Expert blog posts from recognized practitioners
- Conference talks and technical presentations
- Well-sourced journalism and analysis
- Internal Confluence docs and design decisions

Tier 3 (use with caveats):
- Community discussions (Stack Overflow, Reddit, HN)
- Marketing materials and vendor claims
- Undated or anonymous sources
- Single-author opinions without supporting evidence

Always note the tier when citing. If a finding rests entirely on Tier 3
sources, flag it explicitly and recommend verification.

## Anti-Patterns to Avoid

Based on the DEFT failure taxonomy (14 failure modes of AI research agents):

1. **Content piling** -- don't dump everything found. Synthesize ruthlessly.
   Every paragraph must earn its place.
2. **Structural disorganization** -- follow the template strictly. Don't let
   the document drift into stream-of-consciousness.
3. **Evidence misalignment** -- every claim must be supported by a cited source.
   Every source must be relevant to the claim it supports.
4. **Strategic fabrication** -- if you don't know, say you don't know. Flag
   gaps explicitly rather than filling them with plausible-sounding text.
5. **Rigid planning** -- if early findings invalidate your research plan,
   adapt. Don't follow a stale plan to completion.
6. **Limited scope** -- ensure MECE decomposition is truly exhaustive. Check
   for blind spots by asking "what am I NOT searching for?"
7. **Post-rationalized citations** -- ALWAYS search first, read results, THEN
   form conclusions. Never write a finding and then look for sources.

## Reference Files

- [references/output-template.md](references/output-template.md) -- Complete research artifact template
- [references/scoping-guide.md](references/scoping-guide.md) -- Scoping question bank by research type
- [references/source-evaluation.md](references/source-evaluation.md) -- Source evaluation criteria and citation guide
- [references/methodology.md](references/methodology.md) -- Research methodology patterns and frameworks
