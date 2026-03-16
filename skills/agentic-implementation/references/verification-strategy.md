# Verification Strategy for Multi-Agent Architecture

## Why Verification is Different for Agent Swarms

When a single developer writes code, they hold the full system model in their head.
When 5 agents write code in parallel, each agent holds only its module's model.
Verification must catch the gaps between these local models -- interface mismatches,
missing integrations, and architectural drift that no single agent can detect.

## The Verification Lattice

Verification operates at five levels, forming a lattice from fast/cheap to slow/thorough:

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

## Per-Module Verification

Every module must be independently testable before integration. This means:

### 1. Unit Tests with Mock Dependencies

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

### 2. Contract Compliance Tests

Verify that the module's exports match the interface contracts:

```typescript
// src/modules/auth/tests/contract.test.ts
import { AuthServiceContract } from '../../../shared/contracts/auth.contract';
import { authService } from '../index';

describe('Auth module contract compliance', () => {
  it('implements AuthServiceContract', () => {
    // Verify all required methods exist
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

### 3. Export Verification

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

## Integration Checkpoint Verification

At each tier boundary, run verification that crosses module boundaries:

### Tier 0 -> Tier 1 Gate

```bash
#!/bin/bash
# scripts/verify-tier-0.sh

echo "=== Tier 0 Verification ==="

echo "1. Type checking..."
npx tsc --noEmit || exit 1

echo "2. Foundation tests..."
npm test -- --grep "shared" || exit 1

echo "3. Linting..."
npm run lint || exit 1

echo "4. Export verification..."
node -e "
  const types = require('./src/shared/types');
  const db = require('./src/shared/db/client');
  const config = require('./src/shared/config/env');
  console.log('All foundation exports verified');
" || exit 1

echo "=== Tier 0 PASSED ==="
```

### Tier 1 -> Tier 2 Gate (Post-Merge)

```bash
#!/bin/bash
# scripts/verify-tier-1.sh

echo "=== Tier 1 Verification ==="

echo "1. Merge feature branches..."
for branch in feature/auth feature/products feature/notifications feature/ui-shell; do
  git merge $branch --no-edit || {
    echo "MERGE CONFLICT in $branch"
    git merge --abort
    exit 1
  }
  echo "Merged $branch, running tests..."
  npm test || {
    echo "TESTS FAILED after merging $branch"
    git revert HEAD --no-edit
    exit 1
  }
done

echo "2. Full type check..."
npx tsc --noEmit || exit 1

echo "3. Full test suite..."
npm test || exit 1

echo "4. Full lint..."
npm run lint || exit 1

echo "5. Cross-module import verification..."
node -e "
  // Verify all modules are importable
  const auth = require('./src/modules/auth');
  const products = require('./src/modules/products');
  // ... etc
  console.log('All module exports verified');
" || exit 1

echo "=== Tier 1 PASSED ==="
```

### Final Integration Gate

```bash
#!/bin/bash
# scripts/verify-final.sh

echo "=== Final Integration Verification ==="

echo "1. Full test suite..."
npm test || exit 1

echo "2. Build..."
npm run build || exit 1

echo "3. Type check..."
npx tsc --noEmit || exit 1

echo "4. Lint..."
npm run lint || exit 1

echo "5. Integration tests..."
npm test -- --grep "integration" || exit 1

echo "6. E2E tests..."
npm test -- --grep "e2e" || exit 1

echo "=== FINAL INTEGRATION PASSED ==="
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

## Common Verification Failures

| Failure | Cause | Fix |
|---------|-------|-----|
| Type error on import | Contract type mismatch | Check contract definition vs implementation |
| Mock data shape wrong | Mock doesn't match updated type | Update mock in foundation |
| Test timeout | Missing async/await or unresolved promise | Add proper async handling |
| Lint error in generated code | Agent didn't follow style conventions | Update CLAUDE.md with specific style rules |
| Integration test fails | Module assumes internal details of another | Replace with contract-based interaction |
| E2E test fails | Missing route registration | Check integration module route composition |
