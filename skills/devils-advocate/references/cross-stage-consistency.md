# Cross-Stage Consistency Checking

## Purpose

Verify the current artifact is consistent with all prior pipeline stages. Detect
contradictions, drift, and unaddressed items that indicate the pipeline is diverging
from its own decisions.

## Minimum-Context Loading Strategy

Load only what is needed from prior artifacts to maintain cross-stage traceability
without overwhelming context. Read only the specified sections.

### When Critiquing Research
- No prior artifacts to check (research is the first stage)

### When Critiquing PRD
- **From Research:** Executive Summary, Recommendations, Constraints & Boundaries, Open Questions
- **Check:** Every recommendation maps to at least one PRD requirement. No open question
  is left unaddressed without explicit acknowledgment. Constraints carry forward.

### When Critiquing Architecture
- **From Research:** Executive Summary, Technical Landscape (approaches evaluated)
- **From PRD:** Phase structure, acceptance criteria, non-goals, constraints
- **Check:** Architecture supports all PRD phases. Technology choices align with research
  findings. Non-goals are respected (architecture doesn't over-build). Constraints
  from research and PRD are reflected in architecture decisions.

### When Critiquing Implementation Plan
- **From PRD:** Acceptance criteria, phase structure
- **From Architecture:** Module Boundary Map, Dependency DAG, Interface Contracts,
  Integration Checkpoints
- **Check:** Every architecture module has corresponding implementation tasks. Task
  file manifests match architecture file ownership. DAG tiers match implementation
  wave structure. Interface contracts appear in spawn prompts.

### When Critiquing Execution Plan
- **From Implementation:** Task Breakdown (IDs and sizing), Execution Schedule,
  Verification Gates, Merge Strategy
- **Check:** Every implementation task appears in the execution dispatch plan. Wave
  structure matches implementation schedule. Verification gates are executable.
  Budget projections account for all tasks.

### When Critiquing Wave Completion
- **From Execution:** Wave dispatch plan for this specific wave, gate criteria,
  budget ceiling
- **From Implementation:** Task definitions and success criteria for wave tasks
- **From Architecture:** Interface contracts relevant to this wave's modules
- **Check:** Wave output matches execution plan expectations. Contracts implemented
  correctly. Budget within ceiling. Gates passed legitimately.

## Consistency Check Protocol

For each cross-stage comparison:

1. **Identify the claim in the prior artifact** -- quote the specific text
2. **Find the corresponding element in the current artifact** -- quote it
3. **Classify the relationship:**
   - **Consistent:** Current artifact faithfully implements/extends the prior decision
   - **Evolved:** Current artifact intentionally changed the prior decision with stated rationale
   - **Contradicted:** Current artifact conflicts with prior decision without acknowledgment
   - **Missing:** Prior artifact specified something the current artifact does not address
4. **For contradictions and missing items**, generate a finding with:
   - Cross-stage reference citing both artifacts
   - Severity based on downstream impact
   - Suggested fix: either update the current artifact to align, or update the prior
     artifact to reflect the intentional change

## Unresolved Prior Findings

If `./devils-advocate/DA-{prior-stage}.md` files exist, check:

- Were any "Approved" findings actually addressed in the current artifact?
- Are any "Deferred" findings now relevant and still unaddressed?
- Do any recurring patterns emerge across stages (same category of finding appearing repeatedly)?

Flag unresolved approved findings as "escaped" with severity escalation.
