---
name: devils-advocate
description: >
  Structured adversarial review of agentic pipeline artifacts at any stage --
  Research, PRD, Architecture, Implementation, Execution, or post-wave. Combines
  pre-mortem analysis, dimension-specific checklists, cross-stage consistency
  verification, and stress testing to surface blockers, risks, and gaps. Produces
  severity-rated findings with actionable fixes for guided human triage. Triggers
  on "devil's advocate", "critique this", "challenge", "cross-check", "red team",
  "review the [artifact]". Not for building artifacts -- use the stage-specific
  agentic skills for that.
---

# Devil's Advocate

Adversarial critique engine for the agentic development pipeline. One single-pass
review grounded in external verification, not self-critique.

## Critical Rules

- Ground every finding in verifiable evidence -- cite specific sections, line numbers, or cross-stage references
- Every finding at severity S2 (Major) or above MUST include a concrete suggested fix
- Cap total findings at 15-20 per session -- if more exist, state the artifact needs significant rework
- Never adopt an adversary persona -- use structured analysis techniques that force genuine engagement
- Strip authorship signals -- do not reference who or what generated the artifact under review
- Produce findings in a single invocation -- do not re-review or iterate on your own critique. The 4 review passes (A-D) within Step 5 are part of this single invocation.

## Workflow

### 1. Determine Stage

Identify the pipeline stage to critique. Use the explicit stage provided by the user.
If not specified, auto-detect by checking which artifacts exist:

- `./research/RESEARCH-*.md` exists → research stage
- `PRD.md` exists → PRD stage
- `ARCHITECTURE.md` exists → architecture stage
- `IMPLEMENTATION.md` exists → implementation stage
- `EXECUTION.md` exists → execution stage
- User mentions "wave" or wave number → wave stage
- If no pipeline stage matches, use the **generic** stage. Load
  [references/critique-generic.md](references/critique-generic.md) and skip
  Pass C (cross-stage consistency) since there are no prior-stage artifacts.

### 2. Load Context

Read the artifact under review. If the primary artifact exceeds 3000 lines, chunk
the review: read and review each major section independently, then synthesize
findings. For each chunk, carry forward only the running findings list, not the
full prior-chunk content.

Then load prior-stage artifacts for cross-stage consistency checking -- read only
the executive summary and key decisions sections of prior artifacts to minimize
context consumption while maintaining traceability.

Read [references/cross-stage-consistency.md](references/cross-stage-consistency.md)
for the minimum-context loading strategy per stage.

If a `{project-root}/devils-advocate/` directory exists with prior findings files,
read those to check for unresolved findings and recurring patterns. Determine
`{project-root}` via `git rev-parse --show-toplevel`, or use the user's working
directory if not in a git repo.

If 3+ prior DA files exist, calculate historical acceptance rate by category. If
any category has >50% rejection rate, lower default confidence for that category
by one level and note the calibration in the output header.

### 3. Run Deterministic Pre-Checks

Before LLM-based critique, run verifiable checks using tools:

- **File existence**: Verify all file paths referenced in the artifact actually exist
- **Cross-references**: Verify all section references to prior artifacts are valid
- **Schema validation**: Check the artifact has required sections per its stage template
- **Dependency graph**: For architecture/implementation, verify the DAG has no cycles (trace dependencies manually)

Flag any failures as high-confidence (C1) findings.

### 4. Load Stage-Specific Reference Module

Read the appropriate critique reference file:

| Stage | Reference |
|-------|-----------|
| Research | [references/critique-research.md](references/critique-research.md) |
| PRD | [references/critique-prd.md](references/critique-prd.md) |
| Architecture | [references/critique-architecture.md](references/critique-architecture.md) |
| Implementation | [references/critique-implementation.md](references/critique-implementation.md) |
| Execution | [references/critique-execution.md](references/critique-execution.md) |
| Wave | [references/critique-wave.md](references/critique-wave.md) |
| Generic | [references/critique-generic.md](references/critique-generic.md) |

### 5. Execute the Review

Run these passes sequentially against the artifact. Use the stage-specific
checklists from the loaded reference module for passes B-D.

Before executing passes, read [references/anti-patterns.md](references/anti-patterns.md)
and actively monitor your output for these failure modes during all passes.

**Pass A -- Pre-Mortem:** Apply the stage-specific pre-mortem prompt from the
reference module. Assume the project has failed because of this artifact. Generate
the 3-5 most plausible failure causes.

**Pass B -- Dimension Checklist:** Walk through each critique dimension from the
reference module. For each dimension, check the artifact against the specific
tests listed. Record findings only where genuine issues exist.

**Pass C -- Cross-Stage Consistency:** Compare the artifact against prior-stage
artifacts using the protocol in [references/cross-stage-consistency.md](references/cross-stage-consistency.md).
Look for contradictions, drift, and unaddressed items.

**Pass D -- Adversarial Stress Test:** Actively try to break the artifact:
- What happens at 10x scale?
- What if the core assumption is wrong?
- What is the single point of failure?
- What edge case would cause cascading failure?
- What has been assumed but not validated?

### 6. Synthesize Findings

Read [references/severity-framework.md](references/severity-framework.md) and
[references/output-schema.md](references/output-schema.md).

- Deduplicate findings across passes
- Assign severity (S0-S4) and confidence (C1-C3) per the framework
- Derive action type from the severity x confidence matrix
- Generate a concrete suggested fix for every S0-S2 finding
- Check for escaped findings (issues that should have been caught in an earlier stage) -- escalate these by one severity level

### 7. Present for Triage

Present findings in 4-pass triage order:

**Summary line first:** "X findings: N blockers, N critical, N major, N minor, N notes"

**Pass 1 -- Gate Check:** S0 + S1/C1-C2 findings. These determine go/no-go.
**Pass 2 -- Investigation Queue:** S1/C3 findings (labeled "Investigate") + escaped findings.
**Pass 3 -- Improvement Queue:** S2-S3 findings grouped by category, severity-descending.
**Pass 4 -- Context:** S4 findings (present as collapsible/brief).

For each finding, present: ID, severity, confidence, action type, title, claim,
evidence, suggested fix, and downstream impact.

After presenting, ask the user to mark each finding as **Approved**, **Rejected**
(with reason), or **Deferred**.

### 8. Persist Findings

Write findings to `{project-root}/devils-advocate/DA-{STAGE}.md` (e.g.,
`DA-RESEARCH.md`, `DA-PRD.md`, `DA-WAVE-3.md`). Determine `{project-root}` via
`git rev-parse --show-toplevel`, or use the user's working directory if not in a
git repo. Include the triage decisions. If a prior file exists for this stage,
append a dated section rather than overwriting.

### 9. Construct Revision Prompt

After triage, read [references/feedback-loop.md](references/feedback-loop.md)
and construct the revision prompt. Feed it back to the appropriate pipeline skill
to revise the artifact.

## Pre-Flight: Model Diversity

**Before invoking this skill**, consider whether the artifact under review was
generated by the same model that will run the critique. Cross-model review reduces
self-preference bias (Cohen's d > 5.2) and produces higher-quality findings. If
the same model will both generate and critique, switch to a different model for
the critique pass (e.g., use Sonnet to critique an Opus-generated artifact, or
vice versa). If you proceed with the same model, the output header will note this
limitation.

## Constraints

- Do NOT adopt personas ("You are a security expert") -- use task-specific checklists instead
- Do NOT iterate on your own critique -- single invocation only, no meta-review
- Do NOT suggest new features or alternative approaches unless they fix a specific identified defect
- Do NOT exceed 20 findings -- if the artifact has more issues, state it needs rework
- Do NOT suppress high-severity findings due to low confidence -- reclassify as "Investigate"
- Do NOT present findings without suggested fixes for S0-S2 severity
- Do NOT critique style, formatting, or conventions unless they cause functional ambiguity
