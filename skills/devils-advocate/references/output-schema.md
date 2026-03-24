# Output Schema

## Finding Structure

Present each finding using this format:

```
### DA-{NNN}: {title}

| Field | Value |
|-------|-------|
| Severity | S{0-4} ({Blocker/Critical/Major/Minor/Note}) |
| Confidence | C{1-3} ({Confirmed/Likely/Speculative}) |
| Action | {MUST_FIX / MUST_INVESTIGATE / SHOULD_FIX / MAY_FIX / NOTE} |
| Category | {assumption / completeness / consistency / feasibility / risk / quality} |
| Phase | {appropriate / escaped / premature} |
| Downstream | {list of affected pipeline stages, or "none"} |
| Effort | {low / medium / high} (Low: localized text change, <15 min. Medium: structural change to one section, 15-60 min. High: multi-section rework or new research needed, >60 min.) |

**Location:** {Section name in artifact} > "{exact quoted text being critiqued}"

**Claim:** {What the critique asserts, 1-2 sentences}

**Evidence:** {Why this is a problem -- cite specific cross-stage references,
checklist violations, or deterministic check results}

**Suggested Fix:** {Concrete remediation -- specific enough for an LLM to implement
without further clarification. Required for S0-S2; recommended for S3.}
```

## Summary Header

Present this summary before any findings:

```
## Devil's Advocate Review: {artifact name}

**Stage:** {research / prd / architecture / implementation / execution / wave-N}
**Artifact:** {file path}
**Prior findings:** {N unresolved from prior invocations, or "none"}
**Model note:** {If same model generated and is critiquing, note the self-review limitation}

**Summary:** {N} findings: {n} blockers, {n} critical, {n} major, {n} minor, {n} notes
```

## Triage Sections

Present findings in this order:

```
### Gate Check (S0 + S1/C1-C2)
{These determine go/no-go. If any S0 exists, state: "PIPELINE BLOCKED -- resolve before proceeding."}

### Investigation Queue (S1/C3 + escaped)
{Label S1/C3 as "Investigate" -- uncertain but potentially severe.}
{Label escaped findings with: "[ESCAPED from {stage}]"}

### Improvement Queue (S2-S3)
{Group by category. Sort severity-descending within each group.}

### Context (S4)
{Brief one-line summaries. No full finding blocks needed.}
```

## Triage Prompt

After presenting findings, ask:

```
Review each finding above and mark as:
- **Approved** -- I agree, address this in revision
- **Rejected** -- Not a real issue (briefly explain why)
- **Deferred** -- Real issue but not addressing now

You can respond per-finding, per-section, or in bulk (e.g., "Approve all Gate Check,
reject DA-007, defer the rest").
```

## Persistence Format

Write findings to `{project-root}/devils-advocate/DA-{STAGE}.md`:

```markdown
---
stage: {stage}
artifact: {file path}
date: {YYYY-MM-DD}
total_findings: {N}
blockers: {n}
critical: {n}
major: {n}
---

# Devil's Advocate: {Stage} ({date})

## Findings

{Full finding blocks as presented above}

## Triage Decisions

| ID | Severity | Decision | Reason |
|----|----------|----------|--------|
| DA-001 | S1 | Approved | -- |
| DA-002 | S2 | Rejected | Already handled by X |
| DA-003 | S2 | Deferred | Will address in next iteration |
```

If a prior DA file exists for this stage, append a new dated section.
Do not overwrite -- the history enables trend analysis.
