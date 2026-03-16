# Task Decomposition Methodology

How to break architecture modules into agent-executable tasks. Task granularity
is the single largest determinant of multi-agent implementation success.

## Why Granularity Matters

Research findings that directly impact task design:

- **Structured decomposition = 58% faster** completion on complex tasks (MGX.dev research)
- **Tasks >20 min** risk context window degradation and output quality loss
- **Tasks <5 min** waste more time on coordination overhead than they save
- **The 4-agent threshold**: performance saturates or degrades beyond 4-5 parallel
  coding agents (Towards Data Science, CodeCRDT research)
- **Highly coupled tasks show 39.4% slowdown** when parallelized vs. sequential
  (CodeCRDT, 600-trial study)
- **Coordination failures = 36.94%** of all multi-agent system failures (MAST taxonomy)
- **Overly detailed context files reduce success rates** and increase cost by 20%+
  (ETH Zurich study) -- keep task specs focused, not exhaustive

## The Decomposition Process

### Step 1: Map Architecture Modules to Task Groups

For each module in the architecture plan, create a task group:

```
Module: auth-service
  DAG Tier: 1
  Files: src/modules/auth/login.ts, src/modules/auth/session.ts, ...
  Contracts consumed: UserType, SessionType (from Tier 0)
  Contracts produced: AuthServiceContract
  Estimated complexity: Medium (8-12 files)
```

### Step 2: Identify the Natural Task Boundaries Within Each Module

Each module decomposes into tasks along these seams:

1. **Contract implementation tasks** -- implement the interfaces this module exposes
2. **Core logic tasks** -- the business logic that makes the module work
3. **Data access tasks** -- database queries, API calls, file I/O
4. **Validation tasks** -- input validation, error handling, edge cases
5. **Test tasks** -- unit tests, contract compliance tests (often combined with above via TDD)

### Step 3: Apply the TDD Task Pattern

Every implementation task follows this internal structure:

```
1. Write failing tests (Red)
2. Write minimum implementation to pass (Green)
3. Verify all tests pass
4. Commit
```

For the implementation plan, this means each task's "files to create" always
includes the test file first.

### Step 4: Size Each Task

| Complexity | Agent Time | Files Created | Tests | Use When |
|:-----------|:-----------|:-------------|:------|:---------|
| **Small (S)** | ~5 min | 1-2 files | 3-5 tests | Type definitions, simple utilities, config |
| **Medium (M)** | ~10 min | 2-4 files | 5-10 tests | Service implementations, API endpoints |
| **Large (L)** | ~15 min | 3-6 files | 8-15 tests | Complex business logic, multi-step workflows |

**If a task exceeds Large, split it.** Common split patterns:
- Separate the happy path from error handling
- Separate CRUD operations into individual tasks
- Separate the public API from internal helpers
- Separate the implementation from the integration wiring

### Step 5: Add Refactoring Tasks

After every 3-4 implementation tasks within a module, insert a refactoring task.
AI-generated code accumulates technical debt faster than human-written code due to:
- Inconsistent patterns across task boundaries
- Duplicated logic that emerged independently in different tasks
- Suboptimal abstractions chosen for speed

Refactoring tasks should:
- Review all code written in the preceding 3-4 tasks
- Remove duplication
- Ensure consistent naming and patterns per CLAUDE.md
- Verify all exports are intentional
- Run the full module test suite + lint + type check

### Step 6: Define Dependencies

Map task dependencies as a mini-DAG within each module, then connect across modules
via the architecture DAG:

```
T0.1 (shared types) --> T0.2 (shared utils) --> T0.3 (mock data) --> T0.R1 (refactor)
                                                       |
                    -----------------------------------|
                    |               |                  |
                    v               v                  v
              T1.1 (auth)    T2.1 (users)       T3.1 (content)
                    |               |                  |
                    v               v                  v
              T1.2 (auth)    T2.2 (users)       T3.2 (content)
                    |               |                  |
                    v               v                  v
              T1.R1 (refactor) T2.R1 (refactor) T3.R1 (refactor)
```

## Decomposition by Module Type

### Foundation Module (Tier 0)

Foundation should decompose into many small tasks because it gates everything:

| Task Pattern | Example | Complexity |
|:------------|:--------|:-----------|
| Database schema | Define all tables, indexes, constraints | S-M |
| Shared type definitions | TypeScript interfaces for all domain entities | S |
| Shared utilities | Validation helpers, formatters, constants | S |
| Mock data | Test fixtures for every shared type | S |
| Configuration | Environment config, app config | S |
| Error types | Standardized error classes and codes | S |
| Export verification | Test that all shared exports are correct | S |

### Feature Module (Tier 1+)

Feature modules decompose by capability:

| Task Pattern | Example | Complexity |
|:------------|:--------|:-----------|
| Service skeleton + contract | Implement the interface, stub methods | S |
| Core CRUD operations | Create, Read, Update, Delete for primary entity | M |
| Business logic | Validation rules, computed fields, workflows | M-L |
| Error handling | Edge cases, graceful degradation, error responses | M |
| API endpoint(s) | Route handler(s) consuming the service | M |
| Integration wiring | Connect to other modules via contracts | S-M |

### Integration Module (Final Tier)

Integration tasks wire everything together:

| Task Pattern | Example | Complexity |
|:------------|:--------|:-----------|
| Router composition | Compose all module routes into app router | S-M |
| Middleware chain | Auth, logging, error handling middleware | M |
| E2E test suite | End-to-end tests covering main user flows | M-L |
| Build verification | Production build, bundle analysis | S |
| Deployment config | CI/CD pipeline, environment configs | M |

## Common Decomposition Mistakes

| Mistake | Why It Fails | Fix |
|:--------|:-------------|:----|
| One task per module | Tasks too large; agent output degrades | Split by capability within module |
| Tasks that cross module boundaries | Creates file ownership conflicts | Each task touches only its module's files |
| Tests as separate tasks from implementation | Breaks TDD flow; tests written after code are weaker | Combine test + impl as one TDD task |
| No refactoring tasks | Technical debt accumulates, later tasks struggle | Insert refactoring every 3-4 tasks |
| Implicit dependencies | Agent starts before its inputs are ready | Explicit `Depends on` field for every task |
| Foundation too few tasks | Single massive foundation task becomes bottleneck | Split foundation into 4-8 focused tasks |
| Overly detailed task descriptions | Context overhead; agents perform worse | Focus on: objective, files, contracts, tests, constraints |
