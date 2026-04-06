---
title: "OpenSpace Code Review Guidance: AI-First Review Process"
topic: "Institutional code review norms and PR workflow for AI-assisted development"
type: "domain"
status: "refined"
date: "2026-04-06"
pipeline_target: "agentic-execution"
confidence: "high"
sources_consulted: 1
primary_source: "Confluence ENG space — 'New Code Review Guidance: Embracing AI for Faster, Higher Quality Reviews' (Brayden Parkinson, Jack Angers, 2026-03-26)"
---

# OpenSpace Code Review Guidance

## Executive Summary

OpenSpace has adopted an AI-first code review process where **AI handles technical review (bugs, logic errors, edge cases) and humans handle judgment calls (intent, approach, architectural fit)**. The single most impactful behavior change is PR description quality — the description *is* the review now. Claude Code Review is the standard automated reviewer. Human reviewers should clear their queue daily and bias toward merging unless the change is high-risk. This document captures the operational norms any pipeline execution at OpenSpace should follow when preparing PRs, gating review readiness, and automating post-merge workflow.

### Source & Authority

This guidance comes from an official Engineering Process Update authored by Brayden Parkinson and Jack Angers, published to the ENG Confluence space on 2026-03-26. It represents current organizational policy, not a proposal.

---

## Tooling Standardization

OpenSpace has standardized on the Anthropic/Claude ecosystem:

- **Claude Code** — primary AI development tool
- **Claude Code Review** — automated PR review (enabled org-wide)
- **Cursor agents** — being disabled
- **Codex** — backup only if Anthropic is down; not a primary workflow driver

**Pipeline implication:** All agent workflows, skill configurations, and review automation should target Claude Code. Do not build workflows around alternative providers.

---

## The AI/Human Review Split

### What AI Reviews (Claude Code Review)

- Logic errors and bugs
- Security problems
- Edge cases and error handling gaps
- Technical correctness

### What Humans Review

- **Intent** — Does this change make sense? Is it what we agreed to build?
- **Approach** — Is this the right way to solve the problem?
- **Judgment** — Are there trade-offs the author didn't consider?

### Dangerous Areas Requiring Deep Human Review

These categories **always** require a domain expert to review the actual code, regardless of AI review results:

| Category | Examples | Why |
|----------|----------|-----|
| Auth | Authentication flows, session handling, permissions | Security-critical, hard to test exhaustively |
| Database migrations | Schema changes, data migrations | Irreversible in production |
| Data model changes | Entity relationships, field semantics | Cascade effects across the system |
| Billing | Payment flows, subscription logic | Financial impact |

**Pipeline implication:** The devil's advocate skill should flag PRs touching these areas and recommend the `high risk` label. The execution stage should never auto-merge changes in these categories.

---

## PR Description as Primary Review Artifact

> "Authors must write a description clear enough to understand the change without reading the code. If it's not clear, it gets sent back."

This is the single most important norm. The description must answer:

1. **What** changed (summary of the diff in plain language)
2. **Why** it changed (motivation, ticket reference, user impact)
3. **How to verify** (test plan, ephemeral env links, screenshots)
4. **What's risky** (areas that need careful review, known trade-offs)

**Pipeline implication:** The execution stage should generate PR descriptions that meet this bar. A description that requires reading the diff to understand the change is insufficient. When the pipeline creates PRs, the description quality is the primary gate — not code quality (which Claude Code Review handles).

---

## Review Readiness Gate

A PR is **not ready for review** until all of the following are true:

- [ ] Self-review completed (author has read their own diff)
- [ ] AI review completed (Claude Code Review or equivalent)
- [ ] CI checks passing
- [ ] Out of draft mode
- [ ] PR description written and clear

> "The 'push, get 10 comments, fix, push again' cycle kills velocity for everyone involved."

**Pipeline implication:** The execution stage should enforce this as a pre-review checklist. Do not request review until all gates pass. If CI is failing, fix it first. If the description is thin, expand it first.

---

## PR Labels

### Status Labels (mutually exclusive — one per PR)

| Label | Meaning | Action |
|-------|---------|--------|
| `ready to merge` | Approved, checks passing, no blockers | Merge immediately. Do not let these sit. |
| `needs review` | Out of draft, checks passing, description written | This is the queue reviewers scan daily. |
| `needs author action` | Changes requested, checks failing, or missing description | Reviewers skip these. Ball is in the author's court. |
| `blocked` | Waiting on another PR or external input | Don't review until the blocker clears. |

### Category Labels (stackable)

| Label | Purpose |
|-------|---------|
| `experimental` | POCs, spikes, "DO NOT MERGE." Filtered from default view. |
| `chain` | Part of a stacked PR sequence. Use suffix: `chain:react-19` |
| `infrastructure` | CI, build tooling, infra. Different reviewer pool. |
| `high risk` | DB migrations, auth, data model changes. Requires deep human review. |

### Lifecycle Labels (auto-applied)

| Label | Trigger |
|-------|---------|
| `stale` | No activity for 21+ days (GitHub Action) |
| `close candidate` | Flagged during triage. Author has 1 week to respond. |

**Default reviewer view:** `is:open is:pr label:"needs review"`

**Pipeline implication:** The execution stage should apply `needs review` when opening PRs and `high risk` when the diff touches dangerous areas. After approval, apply `ready to merge`. EMs monitor `ready to merge` and `stale`.

---

## Velocity Norms

### Merge & Fix Forward

Unless the change is high-risk (DB migrations, data model, auth, billing), **bias toward merging and addressing follow-ups in a subsequent PR**. A fast follow-up PR beats a week-long review cycle.

### 2-PR Chain Maximum

If stacking PRs, only the one in review and the next one up should be open. The rest stay as local branches.

### Daily Queue Clearing

Reviewers must do a first pass with comments on their entire review queue every day, at minimum.

### Review Duration Signal

> "If a PR is taking you hours to review, something's off — the description isn't good enough, or it needs to be escalated."

Long reviews indicate a process failure, not a complex change. Escalate to EM.

---

## Shift Left to Planning

> "If we're discovering foundational or architecture issues at PR time, something went wrong much earlier."

Architecture and design debates belong in the PRD and Architecture stages of the pipeline — not in PR comments. Agent execution is fast; use the time savings to plan better, not to start more agent cycles.

**Pipeline implication:** This strongly validates the pipeline's sequential PRD -> Architecture -> Implementation -> Execution flow. The Architecture stage's dependency DAGs and interface contracts exist precisely to prevent "discovery at PR time." If a devil's advocate review at the Architecture stage catches a design issue, that's a success. The same issue caught during code review is a pipeline failure.

---

## Use AI Before Asking a Human

> "Generally speaking, you will be judged more harshly for not checking with AI first than for relying on it."

Before opening a PR: ask Claude to review your own code.
Before leaving a review comment: sanity-check your concern with Claude.
Before flagging an architecture issue: ask Claude if there's a simpler path.

**Pipeline implication:** This is the cultural foundation that makes the agentic pipeline viable at OpenSpace. The organization expects AI to be the first pass on everything. Pipeline-generated PRs that have been through the devil's advocate skill are already operating above this bar.

---

## Recommendations for Pipeline Integration

### Execution Stage Updates

1. **PR description generation** should be the execution stage's highest-priority output — more important than the code itself for review velocity.
2. **Label automation** — apply `needs review` on PR creation, `high risk` when touching dangerous areas, `ready to merge` after approval.
3. **Review readiness gate** — enforce the 5-point checklist before requesting review.
4. **Dangerous-area detection** — scan diffs for auth, migration, data model, and billing paths. Flag for human review and apply `high risk` label.

### Devil's Advocate Updates

1. Add the dangerous-area categories as a **mandatory checklist item** during execution-stage review.
2. When reviewing PRs, evaluate description quality as a first-class concern — not just code quality.
3. Flag PRs that would require "hours to review" as candidates for splitting.

### Architecture Stage Reinforcement

1. The shift-left-to-planning norm means architecture-stage thoroughness is even more critical. Design issues caught here save 10x the time vs. catching them in PR review.
2. Interface contracts between modules should be explicit enough that PR reviewers can verify "does this match the contract?" without needing deep domain knowledge.
