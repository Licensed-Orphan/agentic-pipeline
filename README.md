# Agentic Pipeline

A complete pipeline for AI-powered multi-agent software development with Claude Code. Eight skills -- five sequential pipeline stages plus three support skills -- transform a research question into a fully executed codebase built by coordinated agent swarms.

## Pipeline Overview

```
Research --> PRD --> Architecture --> Implementation --> Execution
   ↑                                                       |
   └──── Devil's Advocate (review any stage) ──────────────┘
```

| Stage | Skill | Purpose | Output |
|-------|-------|---------|--------|
| 0. Research | `research` | Standalone deep research on any topic | `./research/RESEARCH-[topic].md` |
| 0a. Agentic Research | `agentic-research` | Pipeline-aware research feeding downstream stages | `./research/RESEARCH-[topic].md` |
| 1. PRD | `agentic-prd` | Define what to build and why | `PRD.md` |
| 2. Architecture | `agentic-architecture` | Design structural decomposition for parallel work | `ARCHITECTURE.md` |
| 3. Implementation | `agentic-implementation` | Break architecture into agent-executable tasks | `IMPLEMENTATION.md` |
| 4. Execution | `agentic-execution` | Live dispatch, monitoring, and completion | `EXECUTION.md` |

### Support Skills

| Skill | Purpose |
|-------|---------|
| `devils-advocate` | Adversarial review of any pipeline artifact -- surfaces risks, gaps, and blockers |
| `skill-builder` | Meta-skill for building and optimizing new Claude Code skills |

## What's Inside

Each skill contains:
- **README.md** -- The main skill definition with workflow, principles, and methodology
- **references/** -- Templates, patterns, and detailed guidance documents

### Skill: Research (`skills/research/`)

Standalone deep research engine for investigating any topic -- technical, product/market, or domain. Produces flexible, standalone research artifacts with evidence-based findings and actionable recommendations. Uses adaptive depth calibration with intelligent source routing across web, Slack, Confluence, Jira, Gmail, Gong, Snowflake, and Figma.

**Methodology:**
- MECE decomposition for exhaustive, non-overlapping research coverage
- Pyramid Principle output structure (conclusions first, evidence on demand)
- SIFT source evaluation with 3-tier credibility scoring
- Analysis of Competing Hypotheses (ACH) for conflict resolution
- Triangulation across source types for confidence scoring
- Information foraging stopping criteria

**References:**
- `output-guide.md` -- Research output formatting and structure guide
- `output-template.md` -- Research artifact template
- `scoping-guide.md` -- Scoping question bank by research type
- `source-evaluation.md` -- Source evaluation criteria (SIFT method) and citation guide
- `methodology.md` -- Research methodology patterns (MECE, Pyramid Principle, ACH, funnel technique)

### Skill: Agentic Research (`skills/agentic-research/`)

Pipeline-aware research engine optimized for feeding downstream agentic pipeline stages (PRD, Architecture, Implementation, Execution). Same core methodology as the standalone research skill but produces structured artifacts with explicit pipeline hooks -- dependency maps, risk registers, and constraint matrices that downstream skills consume directly.

**References:**
- `output-template.md` -- Pipeline-aware research artifact template
- `scoping-guide.md` -- Scoping question bank by research type
- `source-evaluation.md` -- Source evaluation criteria (SIFT method) and citation guide
- `methodology.md` -- Research methodology patterns (MECE, Pyramid Principle, ACH, funnel technique)

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
- `verification.md` -- Verification methodology and patterns
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
- `evaluation-scenarios.md` -- Execution evaluation scenarios and examples
- `anti-patterns.md` -- Known execution failure modes and how to avoid them

### Skill: Devil's Advocate (`skills/devils-advocate/`)

Structured adversarial review of pipeline artifacts at any stage -- Research, PRD, Architecture, Implementation, Execution, or post-wave. Combines pre-mortem analysis, dimension-specific checklists, cross-stage consistency verification, and stress testing to surface blockers, risks, and gaps. Produces severity-rated findings with actionable fixes.

**References:**
- `severity-framework.md` -- Severity rating framework for findings
- `output-schema.md` -- Structured output format for critique reports
- `cross-stage-consistency.md` -- Cross-stage consistency verification checks
- `feedback-loop.md` -- Feedback integration and iteration patterns
- `critique-research.md` -- Research-stage critique checklist
- `critique-prd.md` -- PRD-stage critique checklist
- `critique-architecture.md` -- Architecture-stage critique checklist
- `critique-implementation.md` -- Implementation-stage critique checklist
- `critique-execution.md` -- Execution-stage critique checklist
- `critique-wave.md` -- Post-wave critique checklist
- `critique-generic.md` -- Generic critique checklist for any artifact
- `anti-patterns.md` -- Common critique anti-patterns to avoid

### Skill: Skill Builder (`skills/skill-builder/`)

Meta-skill for building and optimizing Claude Code skills using research-backed principles from 80+ academic, industry, and practitioner sources. Helps create, audit, and refine skills with structured scaffolding, quality checklists, and evaluation scenarios.

**References:**
- `principles.md` -- Core principles for effective skill design
- `patterns.md` -- Proven skill patterns and structures
- `quality-checklist.md` -- Quality audit checklist for skills

**Assets:**
- `skill-template.md` -- Starter template for new skills

**Examples:**
- `evaluation-scenarios.md` -- Evaluation scenarios for skill testing

## Installation as Claude Code Skills

To use these as Claude Code slash commands, copy each skill directory into your Claude Code skills folder:

```bash
# Copy all skills
for skill in research agentic-research agentic-prd agentic-architecture agentic-implementation agentic-execution devils-advocate skill-builder; do
  cp -r skills/$skill ~/.claude/skills/$skill
done
```

Then rename `README.md` back to `SKILL.md` in each:

```bash
for dir in ~/.claude/skills/research ~/.claude/skills/agentic-*/ ~/.claude/skills/devils-advocate ~/.claude/skills/skill-builder; do
  mv "$dir/README.md" "$dir/SKILL.md"
done
```

After installation, use the skills as slash commands in Claude Code:

**Pipeline stages:**
- `/research` -- Standalone deep research on any topic
- `/agentic-research` -- Pipeline-aware research feeding downstream stages
- `/agentic-prd` -- Generate a PRD
- `/agentic-architecture` -- Generate an architecture plan
- `/agentic-implementation` -- Generate an implementation plan
- `/agentic-execution` -- Generate an execution plan

**Support skills:**
- `/devils-advocate` -- Adversarial review of any pipeline artifact
- `/skill-builder` -- Build or optimize a Claude Code skill

## Research-Backed Thresholds

Key empirical findings that inform the pipeline:

- **45% single-agent threshold** (Google/MIT, 2025): Multi-agent only helps for parallelizable, decomposable work
- **35-minute degradation cliff** (Zylos Research, 2026): Agent performance degrades sharply after 35 min
- **79% specification failure rate** (MAST/UC Berkeley, 2025): Most multi-agent failures come from spec issues, not model limitations
- **90.2% improvement** (Anthropic): Opus lead + Sonnet workers outperformed single-agent Opus
- **15x token cost** (Anthropic): Multi-agent systems use ~15x more tokens -- budget controls are essential
- **57% post-rationalized citations** (arXiv 2412.18004): LLMs often generate claims first then find citations -- the research skill combats this with retrieval-first reasoning
- **14 failure modes** (DEFT taxonomy, 2025): Categorized across reasoning, retrieval, and generation -- the research skill explicitly mitigates each

## License

MIT
