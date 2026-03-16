# Agentic Architecture Plan Template

Use this template to generate architecture plans optimized for parallel
decomposition and conflict-free implementation. Adapt section depth based on
project complexity.

This template is agent-agnostic -- it defines the structural decomposition.
Agent assignment, orchestration model selection, and specialist role definitions
belong to the downstream implementation and execution skills.

---

```markdown
# Architecture Plan: [Project Name]

## 1. Executive Summary

### System Overview
[2-3 sentences: What is being built, the primary architectural style, and
the key technology choices. Reference the PRD if one exists.]

### Key Architectural Decisions
| Decision | Choice | Rationale |
|----------|--------|-----------|
| Decomposition strategy | Vertical (feature) slicing | Maximizes parallel throughput |
| Database | [Database] via [ORM] | [Reason] |
| State management | [Approach] | [Reason] |
| API style | [REST/GraphQL/tRPC] | [Reason] |
| Testing strategy | [Framework] | [Reason] |

### Parallelization Summary
| Metric | Value |
|--------|-------|
| Total modules | [N] |
| DAG tiers | [N] |
| Max parallel width | [N] |
| Critical path length | [N modules] |

---

## 2. Module Boundary Map

### 2.1 Decomposition Strategy

[Explain the chosen decomposition: vertical, horizontal, or hybrid.
Justify why this strategy maximizes parallelism for this specific project.]

### 2.2 Module Definitions

#### Module: foundation
**Purpose**: Shared types, database schema, utilities, configuration
**Tier**: 0

**Files created**:
- `src/shared/db/schema.ts`
- `src/shared/db/client.ts`
- `src/shared/db/migrations/001_initial.sql`
- `src/shared/types/index.ts`
- `src/shared/types/user.types.ts`
- `src/shared/types/[domain].types.ts`
- `src/shared/utils/validation.ts`
- `src/shared/utils/formatting.ts`
- `src/shared/contracts/[module].contract.ts`
- `src/shared/mocks/[module].mocks.ts`
- `src/shared/config/env.ts`
- `tsconfig.json`
- `package.json`
- `.env.example`

**Files read**: None (first module)
**Contracts produced**: All shared types and interfaces
**Contracts consumed**: None

---

#### Module: [feature-name]
**Purpose**: [One sentence]
**Tier**: [N]

**Files created**:
- `src/modules/[feature]/api/[endpoint].ts`
- `src/modules/[feature]/services/[service].ts`
- `src/modules/[feature]/types/[feature].types.ts`
- `src/modules/[feature]/tests/[feature].test.ts`
- `src/modules/[feature]/index.ts`

**Files read (not modified)**:
- `src/shared/types/index.ts`
- `src/shared/contracts/[feature].contract.ts`
- `src/shared/db/schema.ts`

**Contracts produced**:
- `[FeatureName]Service` interface at `src/modules/[feature]/index.ts`
- Route handlers at `src/modules/[feature]/api/`

**Contracts consumed**:
- `[DependencyType]` from `src/shared/types/`
- `db` client from `src/shared/db/client.ts`

---

#### Module: integration
**Purpose**: Route composition, cross-cutting middleware, final wiring
**Tier**: [Final]

**Files created**:
- `src/app.ts` (main application entry point)
- `src/routes/index.ts` (route composition)
- `src/middleware/index.ts` (middleware stack)
- `tests/integration/[feature].integration.test.ts`
- `tests/e2e/[flow].e2e.test.ts`

**Files read (not modified)**:
- All `src/modules/*/index.ts` exports
- All `src/shared/` files

---

### 2.3 File Ownership Matrix

| File/Directory | Owner Module | Access | Notes |
|---------------|-------------|--------|-------|
| `src/shared/` | foundation | READ for all others | Frozen after Tier 0 |
| `src/modules/auth/` | auth | EXCLUSIVE | No other module touches |
| `src/modules/products/` | products | EXCLUSIVE | No other module touches |
| `src/app.ts` | integration | EXCLUSIVE | Created in final tier |
| `package.json` | foundation | READ for all others | Dependencies frozen |

---

## 3. Dependency DAG

### 3.1 DAG Visualization

```
                    ┌─────────────┐
                    │ Foundation   │  Tier 0
                    │ schemas,     │
                    │ types, utils │
                    └──────┬──────┘
           ┌───────┬───────┼───────┬──────────┐
           v       v       v       v          v
      ┌────────┐ ┌──────┐ ┌─────┐ ┌────────┐ ┌────────┐
      │ Auth   │ │Prods │ │Notif│ │UI Shell│ │Search  │  Tier 1
      └────┬───┘ └──┬───┘ └──┬──┘ └───┬────┘ └───┬────┘  (width: 5)
           │        │        │        │           │
           └────────┴────────┴────────┴───────────┘
                              │
                    ┌─────────v─────────┐
                    │   Integration     │  Tier 2
                    │   routes, wiring, │
                    │   e2e tests       │
                    └───────────────────┘
```

### 3.2 DAG Table

| Tier | Module | Dependencies | Critical Path? |
|------|--------|-------------|---------------|
| 0 | foundation | none | Yes |
| 1 | auth | Tier 0 | No |
| 1 | products | Tier 0 | Yes |
| 1 | notifications | Tier 0 | No |
| 1 | ui-shell | Tier 0 | No |
| 2 | integration | Tiers 0-1 | Yes |

### 3.3 Critical Path Analysis

**Critical path**: foundation -> [longest Tier 1 module] -> integration
**Optimization opportunities**: [List any dependency restructuring possible]

---

## 4. Interface Contracts

### 4.1 Shared Type Definitions

[Include all shared type definitions that go into the foundation module.
These are the contracts that all feature modules code against.]

### 4.2 API Contracts

[For each cross-module API endpoint, define request/response schemas,
error formats, and mock data. Use the patterns from interface-contracts.md.]

### 4.3 Service Contracts

[For each internal service interface, define the contract with method
signatures, error types, and mock implementations.]

### 4.4 Contract Registry

| Contract | Type | Owner | Consumers | Location |
|----------|------|-------|-----------|----------|
| [ContractName] | [type/api/service/event] | [module] | [modules] | [file path] |

---

## 5. Integration Checkpoints

### Checkpoint: Tier 0 Complete

**Trigger**: Foundation module is complete
**Verification**:
```bash
npx tsc --noEmit                    # Types compile
npm test -- --grep "shared"          # Foundation tests pass
npm run lint                         # Linting passes
```
**Gate**: ALL must pass before Tier 1 modules begin
**Artifacts released**: All `src/shared/` files become available (read-only)

### Checkpoint: Tier 1 Complete

**Trigger**: All Tier 1 modules report completion
**Verification (per module)**:
```bash
npm test -- --grep "[module-name]"   # Module tests pass
npx tsc --noEmit                     # Types still compile
```
**Verification (integration)**:
```bash
npm test                             # Full test suite
npm run lint                         # Full lint
```
**Gate**: ALL must pass before integration tier starts

### Checkpoint: Final Integration

**Trigger**: Integration module is complete
**Verification**:
```bash
npm test                             # All tests pass
npm run build                        # Build succeeds
npm run lint                         # No lint errors
npx tsc --noEmit                     # Type check passes
```
**Manual verification**:
- [ ] Application starts without errors
- [ ] All user flows work end-to-end
- [ ] No console errors or warnings

---

## 6. Risk Mitigation

### 6.1 Merge Conflict Prevention
- File ownership matrix prevents overlap by design
- Foundation module is frozen after Tier 0
- Sequential merge strategy catches conflicts early

### 6.2 Architectural Drift Prevention
- CLAUDE.md constrains all worker behavior
- Interface contracts define exact boundaries
- DO NOT CHANGE lists grow with each tier

### 6.3 Contract Mismatch Recovery
- If a worker discovers a contract needs changing:
  1. Worker STOPS and creates CONTRACT_CHANGE_REQUEST.md
  2. Lead reviews and updates foundation if approved
  3. All affected workers restart from updated foundation
  4. Unchanged workers continue unaffected

### 6.4 Failure Recovery
- If a worker fails or produces poor output:
  1. Discard the worker's branch (trivial with git)
  2. Respawn with additional context
  3. Other workers' work is unaffected (isolation)

### 6.5 Integration Failure Recovery
- If integration tests fail after merge:
  1. Identify which module's code caused the failure
  2. Revert that module's merge
  3. Fix the issue (contract mismatch, missing export, etc.)
  4. Re-merge and re-test

---

## 7. CLAUDE.md Recommendations

[Include the recommended CLAUDE.md content for this project.
See references/claude-md-config.md for patterns.]

---

## 8. Output Contract

The following outputs are consumed by the downstream implementation skill.
If any are missing, the implementation plan cannot be built.

| Output | Description | Used By |
|--------|-------------|---------|
| Module Boundary Map (Section 2) | Every module, its files, its single owner | Task decomposition, file manifests |
| Dependency DAG (Section 3) | Tiers, critical path, parallelization width | Execution scheduling, wave planning |
| Interface Contracts (Section 4) | Type signatures, API schemas, mock data | Spawn prompts, contract tests |
| Integration Checkpoints (Section 5) | Where parallel work must sync | Verification gates, merge ordering |
| CLAUDE.md Recommendations (Section 7) | Conventions, constraints, frozen foundations | Agent spawn context |

---

## Appendix A: Project Structure

```
[Full directory tree showing every file, organized by module.
Mark which module owns each file.]
```

## Appendix B: Technology Stack

| Component | Technology | Version | Notes |
|-----------|-----------|---------|-------|
| Language | [lang] | [ver] | |
| Framework | [framework] | [ver] | |
| Database | [db] | [ver] | |
| ORM | [orm] | [ver] | |
| Testing | [runner] | [ver] | |
| Linting | [linter] | [ver] | |

## Appendix C: PRD Cross-Reference

[Map each PRD phase to its corresponding architecture module(s) and DAG tier.
This ensures nothing from the PRD is missed in the architecture.]

| PRD Phase | Architecture Module(s) | DAG Tier |
|-----------|----------------------|----------|
| Phase 1: Foundation | foundation | 0 |
| Phase 2: [Feature] | [module] | 1 |
| Phase N: Polish | integration | Final |
```

---

## Template Sizing Guide

| Project Size | Modules | Tiers | Max Parallel Width | Total Plan Length |
|-------------|---------|-------|-------------------|-----------------|
| Small (1-3 features) | 3-5 | 2-3 | 2-3 | 3-5 pages |
| Medium (4-8 features) | 5-10 | 3-4 | 4-6 | 5-10 pages |
| Large (8+ features) | 8-15 | 3-5 | 5-8 | 10-15 pages |

**Rules of thumb:**
- Never exceed 15 modules -- split into sub-projects instead
- Keep DAG depth to 3-5 tiers maximum
- Each module should have 5-50 files
