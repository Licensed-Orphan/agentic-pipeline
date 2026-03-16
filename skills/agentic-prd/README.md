---
name: agentic-prd
description: >
  Generate PRDs (Product Requirements Documents) optimized for AI agentic coding
  tools. Produces phased, dependency-ordered specifications with machine-verifiable
  acceptance criteria, explicit constraints, and protection patterns that AI coding
  agents can reliably implement. Use when: writing a PRD, creating a product spec,
  planning a feature for AI agent implementation, specifying requirements for Claude
  Code or other agentic tools, drafting technical specifications, or decomposing a
  project into implementable phases. Triggers on phrases like "write a PRD",
  "product requirements", "spec for this feature", "plan this project",
  "agentic PRD", "specification for AI coding".
license: MIT
metadata:
  version: "1.0"
  author: wesleydubose
---

# Agentic PRD Generator

Generate PRDs structured as sequential, dependency-ordered phase specifications
that AI coding agents can reliably implement. Traditional PRDs optimize for human
comprehension; agentic PRDs function as programming interfaces -- precise enough to
execute, structured enough to sequence, constrained enough to prevent scope drift.

## Workflow

### 1. Research (Mandatory -- Do NOT Skip)

Before writing any specification, gather current ground truth. LLMs have knowledge
cutoffs and AI coding platforms evolve rapidly. Outdated assumptions compound into
implementation failures.

Run these searches (adapt to the user's target platform):
- "[Platform/framework] capabilities [current date]"
- "[Platform] database and storage options"
- "[Specific technology] [platform] compatibility [current date]"
- "[Platform] deployment constraints [current date]"
- "Known issues [framework] [platform] [current date]"

If the user has an existing codebase, explore it first:
- Read key files to understand patterns, conventions, and architecture
- Identify the tech stack, test infrastructure, and deployment setup
- Map existing data models and API structures
- Note naming conventions and code style

See [references/research-phase.md](references/research-phase.md) for the full checklist.

### 2. Interview & Clarify

Use the AskUserQuestion tool to gather essential context. Do NOT ask obvious
questions. Dig into the hard parts the user may not have considered:

- Core functionality and feature priorities
- Target users and key user journeys
- Success metrics and how they will be measured
- Known constraints (performance, compliance, existing systems)
- What explicitly should NOT be built (non-goals)
- Deployment target and platform preferences
- Existing code, design systems, or patterns to follow

Keep interviewing until you have enough to write a complete spec. If the user has
already provided detailed context, skip redundant questions.

### 3. Structure Into Phases

Map feature dependencies and determine phase order. Each phase must:
- Represent a complete, runnable unit (no half-implemented features)
- Build on foundations from prior phases (database before queries, utilities before consumers)
- Contain 30-50 requirements maximum (frontier LLMs degrade beyond ~150-200 instructions)
- End with functionality the user can manually verify
- Take roughly 5-15 minutes of agent work at current capability levels
- Group requirements so that parallel workers within a phase can own separate
  files (two agents editing the same file causes overwrites; the architecture
  skill handles this in detail, but phasing with parallelism in mind makes
  downstream decomposition much cleaner)

See [references/phase-structure.md](references/phase-structure.md) for dependency mapping guidance.

### 4. Write the PRD

Generate the PRD using the template and patterns in the reference files.
Every PRD must include:

- **Research summary** grounding the spec in verified facts
- **Non-goals** stated explicitly (AI cannot infer from omission)
- **Sequential phases** with dependency ordering
- **Machine-verifiable acceptance criteria** for each phase
- **DO NOT CHANGE sections** protecting stable functionality
- **Full file paths** (never bare filenames -- agents create duplicates)
- **Executable verification commands** the agent can run to self-check
- **Always / Ask First / Never** boundary definitions

See [references/prd-template.md](references/prd-template.md) for the complete template.
See [references/acceptance-criteria.md](references/acceptance-criteria.md) for criteria patterns.
See [references/protection-patterns.md](references/protection-patterns.md) for constraint patterns.

### 5. Deliver

Write the PRD to a file (default: `PRD.md` in the project root, or as the user
specifies). Summarize the phase structure and ask the user to review before any
implementation begins.

### 6. Adapt (Mid-Project Changes)

When the user returns with changes after implementation has started:
- **Data model or layout changes** --> Restructure affected phases immediately
- **Isolated new feature** --> Add as an additional phase at the end
- **Visual polish or minor tweaks** --> Scoped phase with DO NOT CHANGE section
- **Scope reduction** --> Remove phases and update dependencies

Always update the PRD file to reflect ground truth. The spec is the source of truth.

## Key Principles

1. **Spec first, code second** -- the specification determines what gets built
2. **Research before specifying** -- verify assumptions against current reality
3. **Phase everything** -- dependency-ordered phases, not monolithic documents
4. **Make it verifiable** -- every criterion must be testable by the agent
5. **State constraints explicitly** -- AI cannot infer boundaries from omission
6. **Compensate for AI weaknesses** -- explicitly require error handling, null checks, security patterns, and performance considerations (AI systematically omits these)
7. **Keep context lean** -- stay under 50 requirements per phase
8. **Use full paths** -- always specify complete file paths
9. **Each phase must be runnable** -- no dead ends, commented-out code, or placeholders
10. **Protect what works** -- DO NOT CHANGE sections for every phase after the first

## Reference Files

- [references/research-phase.md](references/research-phase.md) -- Pre-specification research checklist
- [references/prd-template.md](references/prd-template.md) -- Complete PRD template with examples
- [references/phase-structure.md](references/phase-structure.md) -- Phase sizing, dependency mapping, and sequencing
- [references/acceptance-criteria.md](references/acceptance-criteria.md) -- Machine-verifiable criteria patterns
- [references/protection-patterns.md](references/protection-patterns.md) -- Constraint and boundary patterns
