# Source Evaluation & Citation Guide

## The SIFT Method (Primary Evaluation Framework)

For every source you encounter during research, apply SIFT:

### 1. Stop
Before citing a source, pause. Check your reasoning:
- Am I citing this because it supports my existing conclusion? (confirmation bias)
- Am I citing this because it was the first result? (availability bias)
- Does this source actually say what I think it says? (misattribution)

### 2. Investigate the Source
Leave the source. Search for what OTHERS say about it:
- Is the author recognized in this field?
- Is the publication reputable?
- Are there known biases or conflicts of interest?
- Has this source been cited, critiqued, or debunked elsewhere?

### 3. Find Better Coverage
Look for the same claim from independent sources:
- If 3+ independent Tier 1/2 sources agree → High confidence
- If 2 sources agree, no contradictions → Medium confidence
- If only 1 source, or sources disagree → Low confidence (flag explicitly)

### 4. Trace Back to Original
Follow citation chains to the primary source:
- Blog post cites a paper → read the paper
- News article cites a study → find the study
- Someone on Twitter cites "research shows" → find the actual research
- Internal Slack message references a decision → find the Confluence doc or meeting notes

## Source Tier Classification

### Tier 1 (Highest Weight)
Assign Tier 1 when the source is:
- Official documentation from the technology/platform vendor
- Peer-reviewed academic paper or published research
- Primary data you collected (Snowflake queries, Gong transcript analysis)
- Official blog posts from the creating organization (Anthropic, Google, etc.)
- Industry standards documents (RFC, W3C, ISO)

### Tier 2 (Standard Weight)
Assign Tier 2 when the source is:
- Expert blog posts from recognized practitioners with verifiable credentials
- Conference talks from reputable conferences (Strange Loop, QCon, WWDC, etc.)
- Well-sourced journalism from technical publications (InfoQ, The Verge, Ars Technica)
- Internal Confluence docs, design documents, and architectural decision records
- Reputable analysis firms with transparent methodology

### Tier 3 (Use with Caveats)
Assign Tier 3 when the source is:
- Community discussions (Stack Overflow, Reddit, Hacker News)
- Marketing materials, vendor whitepapers, or sponsored content
- Undated content or content without clear authorship
- Single-author opinions without supporting evidence or data
- AI-generated content (Medium posts, SEO-optimized articles)

**When relying on Tier 3 sources:**
- Always flag them explicitly: "Note: this finding is based on community
  discussion [N] and should be verified before acting on it."
- Never let a Tier 3 source be the sole basis for a recommendation
- Use Tier 3 sources for signal/direction, not for authoritative claims

## Recency Assessment

For fast-moving fields (AI, cloud platforms, frameworks):
- Sources older than 6 months → verify claims are still current
- Sources older than 12 months → treat as historical context, not current truth
- Always include the current date in web searches to surface recent information
- When citing an older source, note the date and whether the information has
  been superseded

For stable fields (algorithms, design principles, domain knowledge):
- Recency is less critical
- Foundational texts retain value regardless of age
- Still check if the specific claims/data have been updated

## Handling Conflicting Sources

When sources disagree, investigate WHY before choosing sides:

### Step 1: Check Contexts
Are the sources talking about the same thing?
- Different scale/use case (what works for startups may not work for enterprises)
- Different time period (the landscape may have changed)
- Different definitions (sources may define key terms differently)

### Step 2: Check Methodologies
How did each source arrive at its conclusion?
- Empirical data vs. opinion
- Controlled study vs. anecdotal experience
- Broad survey vs. single case study

### Step 3: Apply ACH (Analysis of Competing Hypotheses)
Build a matrix:

| Evidence | Hypothesis A | Hypothesis B | Hypothesis C |
|----------|-------------|-------------|-------------|
| [Finding 1] | C (Consistent) | I (Inconsistent) | N (Neutral) |
| [Finding 2] | N | C | C |
| [Finding 3] | I | C | N |

Focus on DISCONFIRMING evidence -- the hypothesis with the least inconsistent
evidence is most likely correct. This counters confirmation bias.

### Step 4: Report the Conflict
In the research output, present conflicts transparently:
- What each side claims
- Why they disagree (context, methodology, definitions)
- Which side the evidence favors (if any)
- What the user needs to decide (if the conflict can't be resolved with evidence)

## Citation Formatting

### Inline Citations
Use numbered references in the text: `[1]`, `[2]`, etc.

Multiple citations for one claim: `[1][3][7]`

When a finding is synthesized from multiple sources, cite all of them.

### Source Appendix Entry Format
```
[N] **Title** -- Author/Organization (Date)
    URL: [url or "Internal: [source type] - [identifier]"]
    Context: [1-2 sentences: what this source contributes and why it's
    credible. NEVER just a raw quote -- provide narrative context per
    the user's feedback preference.]
    Tier: [1/2/3]
    Used in: [which sections reference this source]
```

### Internal Source Citations
For non-web sources, use these formats:

**Slack:**
```
[N] **Slack: #channel-name** -- [Author] (Date)
    Context: [Discussion about X, relevant because Y]
    Tier: 2
```

**Confluence:**
```
[N] **Confluence: [Page Title]** -- [Space/Author] (Last updated: Date)
    URL: [confluence URL]
    Context: [What this document covers and its current relevance]
    Tier: 2
```

**Gong:**
```
[N] **Gong Call: [Call Title]** -- [Participants] (Date)
    Context: [Customer discussed X, relevant because Y]
    Tier: 1 (primary customer data)
```

**Snowflake:**
```
[N] **Snowflake Query: [Description]** -- (Query date: Date)
    Query: [brief description or key SQL]
    Context: [What the data shows and why it matters]
    Tier: 1 (primary data)
```

**Jira:**
```
[N] **Jira: [Issue Key] - [Title]** -- [Reporter/Assignee] (Date)
    Context: [What this issue reveals about the problem space]
    Tier: 2
```

**Figma:**
```
[N] **Figma: [File/Frame Name]** -- [Designer/Team] (Last modified: Date)
    URL: [figma.com URL]
    Context: [What design decisions, patterns, or components this reveals
    and how it informs the research]
    Tier: 2 (design decisions) or Tier 1 (if it's the canonical design system)
```
