# Agentic Pipeline

A complete pipeline for AI-powered multi-agent software development with Claude Code. Four sequential skills transform a product idea into a fully executed codebase built by coordinated agent swarms.

## Pipeline Overview

```
PRD --> Architecture --> Implementation --> Execution
```

| Stage | Skill | Purpose | Output |
|-------|-------|---------|--------|
| 1. PRD | `agentic-prd` | Define what to build and why | `PRD.md` |
| 2. Architecture | `agentic-architecture` | Design structural decomposition for parallel work | `ARCHITECTURE.md` |
| 3. Implementation | `agentic-implementation` | Break architecture into agent-executable tasks | `IMPLEMENTATION.md` |
| 4. Execution | `agentic-execution` | Live dispatch, monitoring, and completion | `EXECUTION.md` |

## What's Inside

Each skill contains:
- **README.md** -- The main skill definition with workflow, principles, and methodology
- **references/** -- Templates, patterns, and detailed guidance documents

### Skill: Agentic PRD (`skills/agentic-prd/`)

Generates PRDs structured as sequential, dependency-ordered phase specifications that AI coding agents can reliably implement.

**References:**
- `prd-template.md` -- Complete PRD template with examples
- `phase-structure.md` -- Phase sizing, dependency mapping, and sequencing
- `research-phase.md` -- Pre-specification research checklist
- `acceptance-criteria.md` -- Machine-verifiable criteria patterns
- `protection-patterns.md` -- Constraint and boundary patterns

### Skill: Agentic Architecture (`skills/agentic-architecture/`)

Generates architecture plans optimized for parallel decomposition and conflict-free multi-agent implementation.

**References:**
- `architecture-template.md` -- Complete architecture plan template
- `module-boundaries.md` -- Module boundary design methodology
- `dependency-dag.md` -- DAG construction and parallelization strategy
- `interface-contracts.md` -- Contract specification patterns
- `claude-md-config.md` -- CLAUDE.md configuration for multi-agent projects
- `anti-patterns.md` -- Known failure modes and how to avoid them

### Skill: Agentic Implementation (`skills/agentic-implementation/`)

Generates implementation plans with agent-executable task breakdowns, spawn prompts, TDD verification gates, and merge strategies.

**References:**
- `implementation-template.md` -- Complete implementation plan template
- `task-decomposition.md` -- Task decomposition methodology and sizing
- `execution-scheduling.md` -- Wave scheduling and parallel lane assignment
- `spawn-prompts.md` -- Spawn prompt templates and best practices
- `verification-gates.md` -- TDD approach and quality gate definitions
- `verification-strategy.md` -- Testing and integration verification patterns
- `merge-integration.md` -- Merge strategy and conflict resolution
- `progress-tracking.md` -- Progress monitoring and failure recovery
- `anti-patterns.md` -- Known failure modes in implementation planning

### Skill: Agentic Execution (`skills/agentic-execution/`)

Generates execution plans for live dispatch and runtime orchestration of specialist multi-agent swarms.

**References:**
- `execution-template.md` -- Complete execution plan template
- `specialist-agents.md` -- Specialist agent definitions and role templates
- `orchestration-runtimes.md` -- Runtime selection and configuration
- `agent-assignment.md` -- Agent assignment strategies and orchestration patterns
- `execution-runbook.md` -- Step-by-step execution runbook template
- `wave-dispatch.md` -- Wave dispatch commands and patterns
- `adaptive-replanning.md` -- Adaptive re-planning triggers and recovery
- `budget-controls.md` -- Budget projections and cost controls
- `anti-patterns.md` -- Known execution failure modes and how to avoid them

## Installation as Claude Code Skills

To use these as Claude Code slash commands, copy each skill directory into your Claude Code skills folder:

```bash
# Copy all skills
cp -r skills/agentic-prd ~/.claude/skills/agentic-prd
cp -r skills/agentic-architecture ~/.claude/skills/agentic-architecture
cp -r skills/agentic-implementation ~/.claude/skills/agentic-implementation
cp -r skills/agentic-execution ~/.claude/skills/agentic-execution
```

Then rename `README.md` back to `SKILL.md` in each:

```bash
for dir in ~/.claude/skills/agentic-*/; do
  mv "$dir/README.md" "$dir/SKILL.md"
done
```

After installation, use the skills as slash commands in Claude Code:
- `/agentic-prd` -- Generate a PRD
- `/agentic-architecture` -- Generate an architecture plan
- `/agentic-implementation` -- Generate an implementation plan
- `/agentic-execution` -- Generate an execution plan

## Research-Backed Thresholds

Key empirical findings that inform the pipeline:

- **45% single-agent threshold** (Google/MIT, 2025): Multi-agent only helps for parallelizable, decomposable work
- **35-minute degradation cliff** (Zylos Research, 2026): Agent performance degrades sharply after 35 min
- **79% specification failure rate** (MAST/UC Berkeley, 2025): Most multi-agent failures come from spec issues, not model limitations
- **90.2% improvement** (Anthropic): Opus lead + Sonnet workers outperformed single-agent Opus
- **15x token cost** (Anthropic): Multi-agent systems use ~15x more tokens -- budget controls are essential

## License

MIT
