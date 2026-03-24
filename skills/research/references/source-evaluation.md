# Source Evaluation & Citation Guide

## The SIFT Method

For every source encountered during research:

### 1. Stop
Before citing, check your reasoning:
- Am I citing this because it supports my existing conclusion? (confirmation bias)
- Am I citing this because it was the first result? (availability bias)
- Does this source actually say what I think it says? (misattribution)

### 2. Investigate the Source
Leave the source. Search for what OTHERS say about it:
- Is the author recognized in this field?
- Is the publication reputable?
- Are there known biases or conflicts of interest?

### 3. Find Better Coverage
Look for the same claim from independent sources:
- 3+ independent Tier 1/2 sources agree → High confidence
- 2 sources agree, no contradictions → Medium confidence
- Only 1 source, or sources disagree → Low confidence (flag explicitly)

### 4. Trace Back to Original
Follow citation chains to the primary source:
- Blog cites a paper → read the paper
- News article cites a study → find the study
- Slack message references a decision → find the Confluence doc

## Source Tier Classification

### Tier 1 (Highest Weight)
- Official documentation and specifications
- Peer-reviewed papers and published research
- Primary data (Snowflake queries, Gong transcripts)
- Official blogs from creating organizations
- Industry standards (RFC, W3C, ISO)

### Tier 2 (Standard Weight)
- Expert blog posts from recognized practitioners
- Conference talks from reputable conferences
- Well-sourced journalism (InfoQ, Ars Technica)
- Internal Confluence docs and design decisions
- Analysis firms with transparent methodology

### Tier 3 (Use with Caveats)
- Community discussions (Stack Overflow, Reddit, HN)
- Marketing materials, vendor whitepapers, sponsored content
- Undated or anonymous content
- Single-author opinions without supporting data
- AI-generated content (Medium posts, SEO articles)

When relying on Tier 3: always flag explicitly, never use as sole basis for
a recommendation, use for signal/direction not authoritative claims.

## Recency Assessment

**Fast-moving fields** (AI, cloud, frameworks):
- >6 months old → verify claims are current
- >12 months old → treat as historical context
- Include current date in web searches

**Stable fields** (algorithms, design principles):
- Recency less critical
- Foundational texts retain value
- Still check if specific data has been updated

## Handling Conflicting Sources

### Step 1: Check Contexts
Are they talking about the same thing? Different scale, time period, definitions?

### Step 2: Check Methodologies
Empirical data vs opinion? Controlled study vs anecdote? Broad survey vs single case?

### Step 3: Apply ACH
Build a hypothesis matrix. Focus on disconfirming evidence.

### Step 4: Report the Conflict
Present: what each side claims, why they disagree, which side evidence favors
(if any), what the user needs to decide.

## Citation Formatting

### Inline
Use numbered references: `[1]`, `[2]`. Multiple for one claim: `[1][3][7]`.

### Source Appendix Entry
```
[N] **Title** -- Author/Organization (Date)
    URL: [url or "Internal: [source type] - [identifier]"]
    Context: [1-2 sentences: what this source contributes and why it's
    credible. Provide narrative context, not just raw quotes.]
    Tier: [1/2/3]
```

### Internal Source Formats

**Slack:**
```
[N] **Slack: #channel-name** -- [Author] (Date)
    Context: [Discussion about X, relevant because Y]
    Tier: 2
```

**Confluence:**
```
[N] **Confluence: [Page Title]** -- [Space/Author] (Last updated: Date)
    URL: [url]
    Context: [What this covers and its current relevance]
    Tier: 2
```

**Gong:**
```
[N] **Gong Call: [Call Title]** -- [Participants] (Date)
    Context: [Customer discussed X, relevant because Y]
    Tier: 1
```

**Snowflake:**
```
[N] **Snowflake Query: [Description]** -- (Query date: Date)
    Query: [brief description or key SQL]
    Context: [What the data shows and why it matters]
    Tier: 1
```

**Jira:**
```
[N] **Jira: [Issue Key] - [Title]** -- [Reporter/Assignee] (Date)
    Context: [What this issue reveals]
    Tier: 2
```

**Figma:**
```
[N] **Figma: [File/Frame Name]** -- [Designer/Team] (Last modified: Date)
    URL: [figma.com URL]
    Context: [What design decisions or patterns this reveals]
    Tier: 2
```
