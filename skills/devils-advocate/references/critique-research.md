# Critique Module: Research Stage

## Pre-Mortem Prompt

"It's 3 months later. The product direction based on this research turned out to
be completely wrong. The team built something nobody wanted, using technology that
didn't work. Why did the research fail to prevent this?"

## Dimension Checklist

### 1. Source Credibility
- Classify each cited source: Tier 1 (official docs, peer-reviewed, primary data),
  Tier 2 (expert blogs, conference talks, internal docs), Tier 3 (community forums,
  marketing materials, undated/anonymous)
- Flag any finding that rests solely on Tier 3 sources
- Check for funding/affiliation bias -- does any source have commercial incentive?
- Verify author authority -- institutional affiliation, domain expertise
- Apply SIFT: Stop, Investigate the source, Find better coverage, Trace to originals

### 2. Coverage Gaps (MECE Completeness)
- List all plausible stakeholder groups, technical domains, and use-case categories
- Check each against coverage -- what is conspicuously absent?
- Verify the research decomposition is Mutually Exclusive (no overlaps) and
  Collectively Exhaustive (no gaps)
- Search for "negative space" -- topics that adjacent research would cover but this doesn't

### 3. Confirmation Bias Detection
- Count confirming vs. disconfirming evidence cited -- ratio >80% confirming is a red flag
- For each major conclusion, identify the strongest counter-argument -- if none found, the research is suspect
- Check if competing hypotheses were considered and evidence mapped against them (ACH)
- Look for cherry-picked sources -- are there obvious authoritative sources that were not cited?

### 4. Evidence Strength
- Grade each recommendation: Strong (multiple independent high-tier sources, empirical data),
  Moderate (2+ credible sources, some support), Weak (single source, anecdotal), Speculative (extrapolation)
- Flag any recommendation graded Weak or Speculative as S1
- Check for quantitative backing -- are claims supported by numbers or only qualitative assertions?

### 5. Recency and Relevance
- Plot source dates -- flag if >50% older than 2 years
- For each technical recommendation, verify referenced tools/APIs/frameworks still exist and are maintained
- Check if newer sources update or contradict the findings

### 6. Missing Perspectives
- Build a stakeholder matrix: end users, operators, security, business, regulators, partners
- Check which perspectives appear in the research
- Flag any unrepresented stakeholder group whose concerns could materially change conclusions

## Deterministic Pre-Checks

- Verify every URL in the source appendix is formatted correctly
- Check that every inline citation [N] has a corresponding source appendix entry
- Verify frontmatter fields exist: title, date, status, confidence, sources_consulted
- Count sources -- fewer than 10 for a comprehensive research artifact is a coverage concern
