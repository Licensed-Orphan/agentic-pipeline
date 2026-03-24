# Feedback Loop: Triage to Revision

## Overview

After the user triages findings (Approved / Rejected / Deferred), construct a
revision prompt that feeds back into the appropriate pipeline skill.

## Revision Prompt Template

```
## Revision Request: {artifact name}

The following findings were identified by adversarial review and triaged by the
project lead. Revise the artifact to address approved findings while preserving
all aspects covered by rejected findings.

### APPROVED FINDINGS (you MUST address these):

{For each approved finding:}
**DA-{NNN} [{severity}]: {title}**
- Problem: {claim}
- Location: {section} > "{quote}"
- Required fix: {suggested_fix}

### REJECTED FINDINGS (do NOT change anything related to these):

{For each rejected finding:}
**DA-{NNN}: {title}**
- The reviewer confirmed this is NOT a problem: {rejection_reason}
- Do NOT modify: {location}

### DEFERRED FINDINGS (acknowledged, no action now):

{For each deferred finding:}
**DA-{NNN}: {title}** -- Deferred. No changes needed.
```

## Processing Rules

- Construct the revision prompt only after the user has triaged ALL findings.
  If the user triages in bulk (e.g., "approve all Gate Check, defer the rest"),
  apply the stated decision to the specified group and "Deferred" to all
  unaddressed findings.
- Include rejected findings as explicit guardrails -- they prevent the revision
  model from "helpfully" changing things the human explicitly blessed
- Group approved findings into 1-2 revision passes by severity tier:
  - **Pass 1:** All S0 + S1 approved findings (must-fix)
  - **Pass 2:** All S2 + S3 approved findings (should-fix)
- If >10 approved findings exist, split into passes to avoid overwhelming the
  revision model. Research shows 1-3 iterations capture most gains; more degrades quality.
- After revision, do NOT re-run the devil's advocate on the revised artifact
  in the same session. Single-pass critique only. The user may invoke a fresh
  critique in a new session if desired.

## Invoking Revision

Feed the revision prompt to the pipeline skill that owns the artifact:

| Artifact | Revision Skill |
|----------|---------------|
| RESEARCH-*.md | agentic-research (refinement phase) |
| PRD.md | agentic-prd (adapt phase) |
| ARCHITECTURE.md | agentic-architecture (step 7 delivery update) |
| IMPLEMENTATION.md | agentic-implementation (step 7 delivery update) |
| EXECUTION.md | agentic-execution (step 9 delivery update) |
| Wave output | agentic-execution (adaptive re-planning) |
| Other | User-directed revision (present the revision prompt and let the user decide how to apply it) |

State which skill should receive the revision prompt and offer to invoke it.
