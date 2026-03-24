# Critique Module: Generic (Non-Pipeline Artifacts)

## Pre-Mortem Prompt

"This artifact was relied upon for 6 months and caused repeated confusion,
misapplication, and wasted effort. Teams followed it but got bad outcomes. What
was wrong with it?"

## Dimension Checklist

### 1. Completeness
- Does the artifact cover all aspects of its stated purpose?
- Are there obvious gaps -- topics it should address but doesn't?
- Are edge cases and boundary conditions addressed?
- Does it handle failure/error scenarios, not just the happy path?

### 2. Internal Consistency
- Do different sections contradict each other?
- Are terms used consistently throughout?
- Do examples match the rules they illustrate?
- Are defaults, fallbacks, and priorities unambiguous?

### 3. Clarity and Unambiguity
- Could a reader follow the artifact without additional context?
- Are instructions specific enough to produce consistent behavior across different readers/executors?
- Flag weasel words: "appropriate," "reasonable," "as needed," "generally" -- each
  should be replaced with specific criteria
- Are conditional branches (if X then Y) clearly structured?

### 4. Feasibility
- Can all instructions actually be carried out with available tools/resources?
- Are there implicit assumptions about capabilities, access, or context that may not hold?
- Are there instructions that depend on external state that may change?
- Does the artifact account for its own execution environment?

### 5. Maintainability
- Can the artifact be updated without cascading changes?
- Are there hardcoded values that should be parameterized?
- Is there unnecessary duplication that could drift out of sync?
- Are cross-references between sections or files robust to restructuring?

### 6. Effectiveness
- Will following this artifact actually achieve its stated goal?
- Are there structural issues that would degrade quality at scale?
- Does it handle the most common use case well, not just the ideal case?
- Are there known failure modes of this type of artifact that aren't mitigated?

## Deterministic Pre-Checks

- Verify all internal cross-references (file paths, section links) resolve
- Check for undefined terms used without explanation
- Verify any referenced external files or tools exist
- Check for orphaned content (sections referenced nowhere, files loaded by nothing)

## Notes

- Skip Pass C (Cross-Stage Consistency) for generic artifacts since there are no
  prior-stage artifacts to compare against
- For Pass D (Stress Test), adapt the questions to the artifact type:
  - What happens if this artifact is used 10x more frequently than expected?
  - What if the core assumption about how it will be used is wrong?
  - What is the single point of failure in this artifact's design?
  - What has been assumed but not validated?
