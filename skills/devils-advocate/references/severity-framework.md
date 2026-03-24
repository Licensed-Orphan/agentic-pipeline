# Severity Framework

## Severity Tiers

| Tier | Label | Definition | Required Action | Time |
|------|-------|-----------|----------------|------|
| S0 | **Blocker** | Will cause project failure or require fundamental rework of completed stages | Must resolve before pipeline progresses | Immediate |
| S1 | **Critical** | Will likely cause significant rework of the current or prior stage if not addressed | Must resolve before current stage is marked complete | Before gate |
| S2 | **Major** | Will degrade quality, increase cost, or create technical debt if not addressed | Should resolve; document rationale if deferred | Before next gate |
| S3 | **Minor** | Quality improvement opportunity with limited impact if ignored | May resolve at discretion | No constraint |
| S4 | **Note** | Observation, question, or suggestion that does not represent a defect | Acknowledge; no action required | None |

## Confidence Levels

| Level | Label | Meaning | Reviewer Behavior |
|-------|-------|---------|-------------------|
| C1 | **Confirmed** | Supported by specific evidence from the artifact, cross-stage reference, or deterministic check | Trust unless you have domain knowledge contradicting it |
| C2 | **Likely** | Reasonable inference based on patterns, but depends on assumptions about context or intent | Verify the claim before accepting |
| C3 | **Speculative** | Possibility worth considering, but cannot be substantiated from available information | Treat as a question, not a defect |

## Severity x Confidence → Action Type

| | C1 (Confirmed) | C2 (Likely) | C3 (Speculative) |
|---|---|---|---|
| **S0-S1** | MUST_FIX | MUST_FIX | MUST_INVESTIGATE |
| **S2** | SHOULD_FIX | SHOULD_FIX | MAY_FIX |
| **S3-S4** | MAY_FIX | NOTE | NOTE |

**Key principle:** Low confidence never reduces a high-severity finding to "ignore."
It changes the action from "fix" to "investigate."

## Phase Containment

Classify each finding by where it belongs in the pipeline:

- **Phase-appropriate:** Found in the stage where it naturally belongs. Apply normal severity.
- **Escaped:** Should have been caught in an earlier stage. Escalate severity by one level. Escaped S0 findings remain S0 -- escalation cannot exceed the maximum severity tier. Instead, flag them as `[ESCAPED + MAX]` to signal that both the finding and the containment failure are critical.
- **Premature:** Concerns a later stage. Classify as S4 (Note) unless it forces rework of the current stage.

## Stage-Specific Severity Calibration

### Research
- S0: Core hypothesis unfounded or contradicted by evidence
- S1: Key source unreliable; key stakeholder perspective missing; recommendation rests on weak evidence
- S2: Scope doesn't fully cover problem space; minor gaps in corroboration
- S3: Minor gaps in supporting evidence; stylistic issues in presentation

### PRD
- S0: Requirements internally contradictory or technically impossible
- S1: Success metrics unmeasurable; key user journey unaddressed; acceptance criteria untestable; security requirements missing
- S2: Edge cases not considered; non-functional requirements underspecified
- S3: User story language ambiguous but intent clear

### Architecture
- S0: Architecture cannot satisfy stated requirements; critical SPOF on core path
- S1: Circular dependencies; significant scalability/security risk; file ownership overlap; contracts missing
- S2: Coupling concerns; missing failure modes; DAG suboptimal
- S3: Naming conventions; documentation gaps

### Implementation Plan
- S0: Plan omits critical work required by architecture
- S1: Dependencies misordered; spawn prompts incomplete; no recovery procedures; effort estimates >3x off
- S2: Test strategy gaps; task granularity inconsistent; integration points underspecified
- S3: Minor sizing inconsistencies

### Execution Plan
- S0: Timeline physically impossible given constraints
- S1: Critical path has no slack; no timeouts; budget has zero padding; merge strategy unsafe
- S2: Risk mitigation missing for identified risks; agent assignment suboptimal
- S3: Communication plan gaps

### Wave Completion
- S0: Delivered work fails acceptance criteria; regression tests failing
- S1: Contract violations; drift >30% from plan; CPI < 0.7; security vulnerabilities introduced
- S2: Test coverage decreased; technical debt increased; new TODOs outnumber resolved
- S3: Minor plan deviations that don't affect outcomes
