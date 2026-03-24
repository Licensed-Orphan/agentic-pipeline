# Verification Framework

Verification gates are the checkpoints that prevent bad code from propagating
through the implementation. Without gates, errors compound: a bad type definition
in Tier 0 cascades into every module, and a broken contract in Tier 1 cascades
into integration. When 5 agents write code in parallel, each agent holds only its
module's model -- verification must catch the gaps between these local models.

## The Research Case for Aggressive Verification

- Google's 2025 DORA Report: 90% AI adoption increase correlates with **9% climb
  in bug rates**, **91% increase in code review time**, **154% increase in PR size**
- CodeScene: "Agents lack a reliable understanding of maintainability and change
  risk inside a real codebase"
- CodeRabbit 2025: AI generates **1.5-2x more security vulnerabilities** and
  **3x more readability issues** than human code
- Automated verification yields consistent gains of **3.7-6.5%**, primarily
  correcting typos, missing dependencies, and wrong arguments

**Conclusion:** More autonomous coding requires MORE verification, not less.

## TDD as the Primary Gate

Test-Driven Development is the single most important verification strategy for
agentic coding. Research consensus (Simon Willison, Tweag, CodeScene, MIT):

### Why TDD Works for Agents

1. **Tests are specifications** -- a test file is a precise, machine-readable
   spec that guides the agent toward exactly the behavior you expect
2. **Red/Green cycle is natural for agents** -- write failing test (Red), iterate
   on implementation until test passes (Green) -- agents excel at this loop
3. **Tests catch drift** -- agents make locally sensible but globally inconsistent
   decisions; tests catch the inconsistencies immediately
4. **Tests enable isolation** -- with proper mocks, each module can be tested
   independently, enabling true parallel development

### TDD Task Pattern

Every implementation task in the plan should follow this structure:

```
1. READ: Interface contract for what this task implements
2. WRITE: Test file with tests for expected behavior
   - Happy path (the expected inputs produce expected outputs)
   - Validation errors (invalid inputs are rejected correctly)
   - Edge cases (empty input, maximum values, null/undefined)
   - Error cases (dependency failures, network errors)
3. RUN: Verify tests fail (Red) -- confirms tests are actually testing something
4. WRITE: Implementation to make tests pass
5. RUN: Verify tests pass (Green)
6. RUN: Verify lint + type check pass
7. COMMIT: Implementation + tests as an atomic unit
```

### Test Requirements Per Task Type

| Task Type | Min Tests | Coverage Focus |
|:----------|:---------|:--------------|
| Type definitions | 3-5 | Exports exist, required fields, constraints |
| Utility functions | 5-8 | Each function: happy path, edge cases, errors |
| Service implementation | 8-12 | Each method: CRUD + validation + errors |
| API endpoint | 5-10 | Status codes, response shapes, auth, validation |
| Integration wiring | 3-5 | Cross-module calls work, errors propagate correctly |
| E2E flow | 3-5 per flow | Full user journey: setup -> action -> verification |

## The Five-Level Verification Lattice

Verification operates at five levels, from fast/cheap to slow/thorough:

```
Level 1: DETERMINISTIC (every commit, every agent)
  ├── Compilation / type checking
  ├── Unit tests (module-scoped)
  ├── Linting
  └── Formatting

Level 2: CONTRACT (per-module completion)
  ├── Interface compliance tests
  ├── Mock data validation (mocks match real types)
  ├── Export verification (all public APIs importable)
  └── Schema validation (DB types match ORM types)

Level 3: INTEGRATION (per-tier completion)
  ├── Cross-module import resolution
  ├── API endpoint response validation
  ├── Event handler processing verification
  ├── Database query execution
  └── Middleware chain testing

Level 4: SYSTEM (final integration)
  ├── End-to-end user flow tests
  ├── Full regression suite
  ├── Performance benchmarks
  └── Security scanning (SAST, dependency audit)

Level 5: ACCEPTANCE (human sign-off)
  ├── PRD acceptance criteria verification
  ├── Manual exploratory testing
  ├── Cross-browser/device validation
  └── Stakeholder review
```

### Level 1: Task-Level Gates (After Every Task)

**Triggered by:** Agent completes a task
**Run by:** The agent itself, before marking the task as Done

```bash
# Every agent runs after every task:
npx tsc --noEmit                                      # Type check
npm test -- --testPathPattern="[task-specific-path]"   # Task tests
npx eslint [task-specific-files] --max-warnings=0     # Lint

# All three must exit with code 0
```

**If any check fails:** The agent must fix the issue before moving to the next
task. Do NOT proceed with a failing gate.

### Level 2: Module-Level Gates (After All Tasks in a Module)

**Triggered by:** All tasks in a module are marked Done
**Run by:** The agent that completed the module

```bash
# Contract compliance verification
npm test -- --testPathPattern="modules/[name]"    # All module tests
npx tsc --noEmit                                  # Full type check
```

**Gate criteria:**
- [ ] All module tests pass (0 failures)
- [ ] Contract compliance test passes (exports match interface)
- [ ] No lint warnings in module directory
- [ ] Type check clean
- [ ] No TODO comments or placeholder implementations
- [ ] No files created outside the module's file manifest

### Level 3: Wave-Level Gates (At Synchronization Points)

**Triggered by:** All agents in a wave report Done
**Run by:** The lead agent or integration agent

```bash
# Sequential merge with test-after-each:
git merge agent-a1-branch
npm test                        # Full test suite after first merge
# If pass: continue
# If fail: revert, fix, retry

git merge agent-a2-branch
npm test                        # Full test suite after second merge
# Repeat for each agent branch

# After all merges:
npx tsc --noEmit               # Full type check
npm run build                   # Build verification (catches import issues)
npx eslint . --max-warnings=0  # Full lint
```

**Gate criteria:**
- [ ] All agent branches merged successfully
- [ ] Full test suite passes after all merges
- [ ] Build succeeds
- [ ] No type errors
- [ ] No lint warnings

**If a merge introduces failures:**
1. Revert the problematic merge
2. Identify the failing tests -- they indicate the conflict boundary
3. Determine if it's a contract mismatch, file ownership violation, or logic error
4. Fix in the feature branch and re-attempt merge

### Level 4: System-Level Gates (Final Verification)

**Triggered by:** All waves complete
**Run by:** Integration agent or lead

```bash
# Full system verification
npx tsc --noEmit                          # Type check
npm test                                   # Full unit + integration tests
npm run test:e2e                           # End-to-end tests
npm run build                              # Production build
npx eslint . --max-warnings=0             # Full lint
```

**Gate criteria:**
- [ ] All tests pass (unit + integration + E2E)
- [ ] Production build succeeds
- [ ] No lint warnings, no type errors
- [ ] All PRD acceptance criteria verified (map each criterion to a test or manual check)
- [ ] Manual exploratory testing completed for user-facing features
- [ ] No TODO comments, no placeholder implementations, no commented-out code

### Level 5: Acceptance (Human Sign-Off)

- PRD acceptance criteria mapped to passing tests or manual verification
- Exploratory testing of user-facing features
- Cross-browser/device validation where applicable
- Stakeholder review and approval

## Per-Module Verification

Every module must be independently testable before integration.

### Unit Tests with Mock Dependencies

Each module tests against mock implementations of its dependencies, not real
implementations (which may not exist yet when working in parallel):

```typescript
// src/modules/auth/tests/login.test.ts
import { mockDb } from '../../../shared/mocks/db.mocks';
import { mockUser } from '../../../shared/mocks/user.mocks';
import { loginHandler } from '../api/login';

describe('loginHandler', () => {
  it('returns JWT for valid credentials', async () => {
    mockDb.users.findByEmail.mockResolvedValue(mockUser);
    const result = await loginHandler({
      email: 'test@example.com',
      password: 'ValidPass123!'
    });
    expect(result.token).toBeDefined();
    expect(result.user.id).toBe(mockUser.id);
  });

  it('returns 401 for invalid password', async () => {
    mockDb.users.findByEmail.mockResolvedValue(mockUser);
    await expect(
      loginHandler({ email: 'test@example.com', password: 'wrong' })
    ).rejects.toMatchObject({ status: 401, code: 'INVALID_CREDENTIALS' });
  });
});
```

### Contract Compliance Tests

Verify that the module's exports match the interface contracts:

```typescript
// src/modules/auth/tests/contract.test.ts
import { AuthServiceContract } from '../../../shared/contracts/auth.contract';
import { authService } from '../index';

describe('Auth module contract compliance', () => {
  it('implements AuthServiceContract', () => {
    const contract: AuthServiceContract = authService;
    expect(typeof contract.verifyToken).toBe('function');
    expect(typeof contract.hasRole).toBe('function');
  });

  it('verifyToken returns User shape', async () => {
    const user = await authService.verifyToken('valid-test-token');
    expect(user).toHaveProperty('id');
    expect(user).toHaveProperty('email');
    expect(user).toHaveProperty('displayName');
    expect(user).toHaveProperty('role');
  });
});
```

### Export Verification

Verify all public APIs are importable from the module's index:

```typescript
// src/modules/auth/tests/exports.test.ts
import * as authModule from '../index';

describe('Auth module exports', () => {
  it('exports authService', () => {
    expect(authModule.authService).toBeDefined();
  });

  it('exports authMiddleware', () => {
    expect(authModule.authMiddleware).toBeDefined();
  });

  it('exports auth router', () => {
    expect(authModule.authRouter).toBeDefined();
  });
});
```

## Verification Script Templates

Include these as runnable scripts in the implementation plan:

### tier0-gate.sh
```bash
#!/bin/bash
set -e
echo "=== Tier 0 Verification Gate ==="
echo "Type checking..."
npx tsc --noEmit
echo "Running foundation tests..."
npm test -- --testPathPattern="shared" --ci
echo "Linting foundation..."
npx eslint src/shared/ src/db/ --max-warnings=0
echo "Export verification..."
node -e "
  const types = require('./src/shared/types');
  const db = require('./src/shared/db/client');
  const config = require('./src/shared/config/env');
  console.log('All foundation exports verified');
" || exit 1
echo "=== Tier 0 Gate: PASSED ==="
```

### module-gate.sh
```bash
#!/bin/bash
set -e
MODULE=$1
echo "=== Module Gate: $MODULE ==="
echo "Type checking..."
npx tsc --noEmit
echo "Running module tests..."
npm test -- --testPathPattern="modules/$MODULE" --ci
echo "Linting module..."
npx eslint "src/modules/$MODULE/" --max-warnings=0
echo "=== Module Gate: $MODULE PASSED ==="
```

### merge-gate.sh
```bash
#!/bin/bash
set -e
echo "=== Tier Merge Gate ==="
for branch in "$@"; do
  echo "Merging $branch..."
  git merge "$branch" --no-edit || {
    echo "MERGE CONFLICT in $branch"
    git merge --abort
    exit 1
  }
  echo "Running tests after merging $branch..."
  npm test --ci || {
    echo "TESTS FAILED after merging $branch"
    git revert HEAD --no-edit
    exit 1
  }
done
echo "Full type check..."
npx tsc --noEmit
echo "Building..."
npm run build
echo "Cross-module import verification..."
node -e "
  // Verify all modules are importable
  const auth = require('./src/modules/auth');
  const products = require('./src/modules/products');
  console.log('All module exports verified');
" || exit 1
echo "=== Merge Gate: PASSED ==="
```

### final-gate.sh
```bash
#!/bin/bash
set -e
echo "=== Final Verification Gate ==="
echo "Type checking..."
npx tsc --noEmit
echo "Running all tests..."
npm test --ci
echo "Running E2E tests..."
npm run test:e2e --ci
echo "Building for production..."
npm run build
echo "Linting..."
npx eslint . --max-warnings=0
echo "=== Final Gate: PASSED ==="
```

## Merge Strategy

### Sequential Merge with Test-After-Each

The safest merge strategy for agent swarm output:

1. Merge Tier 0 (foundation) into main
2. Run full test suite
3. For each Tier 1 branch:
   a. Merge into main
   b. Run full test suite
   c. If tests fail, revert and investigate
4. Merge Tier 2 (integration) into main
5. Run final verification

This catches conflicts early and makes them easy to attribute.

### If Merge Conflicts Occur

1. **Same file, different sections**: Usually auto-resolvable by git
2. **Same file, same section**: Indicates a boundary violation -- review
   the file ownership matrix and fix the boundary design
3. **Import conflicts**: Usually means a module imported from another module
   instead of from the foundation -- fix by moving the shared code
4. **Package.json conflicts**: Foundation owns package.json; feature modules
   should not add dependencies directly

## Agent-Specific Verification Commands

Include these in each agent's spawn prompt:

```markdown
## Verification Commands (run before reporting completion)

1. Type check: `npx tsc --noEmit`
2. Module tests: `npm test -- --grep "[your-module-name]"`
3. Lint: `npm run lint`
4. Build check: `npm run build` (if applicable)

ALL must pass. Do not report completion until all pass.
If any fail, fix the issues and re-run.
```

## Common Verification Failures and Fixes

| Failure | Likely Cause | Fix |
|:--------|:------------|:----|
| Type errors after merge | Contract mismatch between modules | Compare module's usage against contract; fix the non-compliant side |
| Type error on import | Contract type mismatch | Check contract definition vs implementation |
| Mock data shape wrong | Mock doesn't match updated type | Update mock in foundation |
| Test timeout | Mock not properly configured or missing async/await | Replace real deps with mocks; add proper async handling |
| Import not found | Module not exporting expected symbol | Check contract compliance test; add missing export |
| Lint errors | Agent used different style conventions | Ensure CLAUDE.md style rules are in spawn prompt |
| Integration test fails | Module assumes internal details of another | Replace with contract-based interaction |
| E2E test fails | Integration wiring incorrect or missing route registration | Trace full call chain; check middleware order and route composition |
| Build fails | Circular imports between modules | Module imported from another feature module; refactor through shared |
| Test passes locally, fails in CI | Environment-specific dependency | Ensure all tests use mocks for external services |
