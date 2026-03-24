---
name: agentic-architecture
description: >
  Generate software architecture plans optimized for parallel decomposition and
  conflict-free implementation. Produces module boundary maps with explicit file
  ownership, dependency DAGs, interface contracts, and CLAUDE.md recommendations
  that maximize throughput when implemented by coordinated coding agents or teams.
  This skill is agent-agnostic -- it defines HOW the system decomposes structurally,
  not how agents will be assigned or orchestrated (that belongs to the downstream
  implementation and execution skills). Use when: creating a software architecture
  plan, designing system architecture for parallel implementation, translating a
  PRD into an architecture plan, planning a codebase structure for parallel coding,
  or architecting a system that will be built by multiple workers. Triggers on
  phrases like "architecture plan", "software architecture", "system design for
  agents", "architect this PRD", "design the architecture", "agentic architecture".
---

# Agentic Architecture Planner

Generate architecture plans optimized for parallel decomposition and
conflict-free implementation by multiple coding agents. The plan bridges
a PRD (what to build) and an implementation plan (how to decompose into tasks).

## Constraints

- Do NOT assign the same file to multiple modules -- overlapping ownership guarantees merge conflicts
- Do NOT skip Tier 0 (foundation) -- shared types, schemas, and contracts must exist before feature work
- Do NOT create circular dependencies in the DAG -- if A depends on B and B depends on A, extract the shared dependency into a lower tier
- Do NOT define interface contracts inside feature modules -- all contracts belong in the foundation module
- Do NOT make agent orchestration decisions (model selection, specialist roles, spawn strategies) -- those belong to the downstream implementation and execution skills
- Do NOT leave any cross-module boundary without an explicit contract
- Do NOT exceed 15 modules -- split into sub-projects instead
- Do NOT produce a plan missing any item from the Output Contract table

## Position in the Pipeline

```
PRD (agentic-prd)          -- what to build and why
Architecture (this skill)  -- how it decomposes structurally
Implementation             -- tasks, prompts, verification gates
Execution                  -- live dispatch, monitoring, completion
```

This skill consumes the PRD and produces an agent-agnostic architecture plan.
It does NOT make decisions about agent orchestration models, specialist roles,
or model selection -- those are downstream concerns handled by the implementation
and execution skills.

## Workflow

### 1. Gather Inputs (Mandatory)

Before designing architecture, collect ground truth.

**If a PRD exists**, read it fully:
- Extract all phases, requirements, and constraints
- Identify the dependency graph between phases
- Note the tech stack, conventions, and boundary rules
- Map which requirements can be parallelized
- Identify non-goals and protection patterns
- Extract the Always / Ask First / Never boundaries

**If a codebase exists**, explore it for architecture-specific concerns:
- Map the current directory structure and module boundaries
- Identify existing interface contracts (API routes, type definitions, schemas)
- Understand the build system and dependency graph
- Note existing test infrastructure patterns

(The PRD's research summary covers platform capabilities and technology choices.
Do not re-research what the PRD already verified. Focus on structural concerns
the PRD does not cover: directory layout, module organization, build tooling.)

**If neither PRD nor codebase exists**, interview the user:
- Use AskUserQuestion to gather requirements, stack preferences, and constraints
- Determine deployment target and scaling requirements
- Advise generating a PRD first for best results

### 2. Design the Module Boundary Map

Design module boundaries first -- they determine parallelization potential.
Read [references/module-boundaries.md](references/module-boundaries.md) for the
complete boundary design methodology.

**Core principles:**
- Each module must be ownable by a single worker with no file overlap
- Modules communicate through explicit interface contracts (types, APIs, schemas)
- Shared code lives in a dedicated utilities/shared module built first
- Database schemas and type definitions are foundational -- built before all else
- No module should require another module's internal implementation details
- Design for vertical (feature) slicing when possible -- it maximizes parallelism

### 3. Build the Dependency DAG

Map every module and its dependencies as a directed acyclic graph.
Read [references/dependency-dag.md](references/dependency-dag.md) for DAG
construction methodology.

**The DAG must answer:**
- What can be built in parallel at each tier?
- What is the critical path (longest sequential chain)?
- Where are the integration points that require synchronization?
- What is the minimum number of sequential tiers needed?
- How wide is each tier (how many parallel workers can operate simultaneously)?

### 4. Define Interface Contracts

Define an explicit contract for every cross-module boundary. Read
[references/interface-contracts.md](references/interface-contracts.md) for
contract specification patterns.

**Every contract must include:**
- Type signatures / API schemas with full field definitions
- Input validation rules and constraints
- Error response formats with specific error codes
- Mock/stub data for testing in isolation
- Expected behavior under edge cases (empty inputs, max values)

### 5. Generate CLAUDE.md Recommendations

The architecture plan must include recommended CLAUDE.md content that workers
will load automatically. CLAUDE.md is the **only guaranteed shared context**
across all agents -- subagents, Agent Team teammates, and manual worktree
sessions all load it, but none of them inherit the lead's conversation history.
This makes CLAUDE.md the primary mechanism for both constraining behavior and
delivering essential architectural context that every agent needs regardless
of their specific role.

Beyond conventions and constraints, CLAUDE.md should include:
- A summary of key architectural decisions and their rationale
- Module boundaries and file ownership rules
- Interface contract locations and frozen/mutable status
- Verification commands every agent must run before reporting completion

See [references/claude-md-config.md](references/claude-md-config.md) for
CLAUDE.md configuration patterns for multi-worker projects.

### 6. Write the Architecture Plan

Generate the architecture plan using the template and patterns in the reference
files. Every plan must include:

- **Executive Summary** with system overview and key decisions
- **Module Boundary Map** with complete file ownership per module
- **Dependency DAG** showing parallelization tiers and critical path
- **Interface Contracts** for every cross-module boundary
- **Integration Checkpoints** defining where parallel work must sync
- **Risk Mitigation** for merge conflicts, architectural drift, and failure recovery
- **CLAUDE.md Recommendations** for project-level instructions

See [references/architecture-template.md](references/architecture-template.md)
for the complete template.

### 7. Deliver

Write the architecture plan to a file (default: `ARCHITECTURE.md` in the project
root, or as the user specifies). Summarize the module structure, parallelization
tiers, and critical path. Ask the user to review before implementation planning
begins.

If an agentic PRD exists, update it to reference the architecture plan.

#### Output Contract

The architecture plan provides the following to the downstream implementation
skill. If any of these are missing, the implementation plan cannot be built:

| Output | Description | Used By |
|--------|-------------|---------|
| Module Boundary Map | Every module, its files, its single owner | Task decomposition, file manifests |
| Dependency DAG | Tiers, critical path, parallelization width | Execution scheduling, wave planning |
| Interface Contracts | Type signatures, API schemas, mock data | Spawn prompts, contract tests |
| Integration Checkpoints | Where parallel work must sync | Verification gates, merge ordering |
| CLAUDE.md Recommendations | Conventions, constraints, frozen foundations | Agent spawn context |

## Key Principles

1. **File ownership is sacred** -- every file belongs to exactly one module;
   overlapping ownership guarantees merge conflicts
2. **Interfaces before implementations** -- define contracts first, build
   implementations against them; parallel workers can proceed when contracts exist
3. **Maximize the DAG width** -- wider tiers mean more parallel work; flatten
   dependencies by extracting shared foundations
4. **Build foundations first** -- schemas, types, shared utilities, and interface
   contracts must exist before feature modules begin
5. **Design for isolation** -- each module must be testable independently using
   mocks/stubs of its dependencies
6. **Minimize the critical path** -- the longest sequential chain determines
   total build time; restructure to shorten it
7. **Plan for merge** -- define integration checkpoints where parallel work
   converges and is verified together
8. **Make architecture machine-readable** -- use consistent structure, explicit
   file paths, and parseable formats (DAGs, tables, type definitions)
9. **Ship incrementally** -- each DAG tier should produce a runnable system;
   no tier should leave the codebase in a broken state

## Reference Files

- [references/module-boundaries.md](references/module-boundaries.md) -- Module boundary design methodology
- [references/dependency-dag.md](references/dependency-dag.md) -- DAG construction and parallelization strategy
- [references/interface-contracts.md](references/interface-contracts.md) -- Contract specification patterns
- [references/architecture-template.md](references/architecture-template.md) -- Complete architecture plan template
- [references/claude-md-config.md](references/claude-md-config.md) -- CLAUDE.md configuration for multi-worker projects
- [references/anti-patterns.md](references/anti-patterns.md) -- Known failure modes and how to avoid them

## Evaluation Scenarios

1. **Happy path**: Given a PRD for a 4-feature web app (auth, products, orders, notifications), generate an architecture plan with vertical slicing, a 3-tier DAG (foundation -> features -> integration), full interface contracts, and CLAUDE.md recommendations. Verify every Output Contract item is present.

2. **Edge case -- existing codebase, no PRD**: Given an existing monorepo with no PRD, explore the codebase and produce a retroactive architecture plan documenting current module boundaries, identifying file ownership conflicts, and recommending a DAG for future parallel work.

3. **Failure case -- monolith trap**: Given a PRD where all features share a single database table and heavy cross-feature dependencies, verify the skill detects the monolith trap, recommends extracting shared types into foundation, and produces a DAG with width > 1 at Tier 1 (not a sequential chain).
