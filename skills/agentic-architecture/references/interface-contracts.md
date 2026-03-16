# Interface Contract Specification Patterns

## Why Contracts Are Non-Negotiable

When agents build modules in parallel, they cannot read each other's code. Agent A
building the auth module and Agent B building the products module must agree on
exactly how their modules will interact -- before either starts coding.

Without explicit contracts:
- Agent A exports `getUser(id: string)` returning `{ name, email }`
- Agent B expects `fetchUser(userId: number)` returning `{ username, emailAddress }`
- Integration fails on function name, parameter type, AND return shape

Anthropic's multi-agent research found that detailed task boundaries are essential:
each subagent needs "an objective, an output format, guidance on tools and sources
to use, and clear task boundaries." Contracts are those boundaries for code.

## Contract Types

### 1. Type Contracts (Data Shapes)

Define every shared type in the foundation module before parallel work begins:

```typescript
// src/shared/types/user.types.ts -- FOUNDATION MODULE (frozen after Tier 0)

export interface User {
  id: string;           // UUID v4
  email: string;        // Lowercase, max 255 chars
  displayName: string;  // 1-100 chars, trimmed
  role: UserRole;
  createdAt: Date;
  updatedAt: Date;
}

export type UserRole = 'admin' | 'member' | 'viewer';

export interface CreateUserInput {
  email: string;
  displayName: string;
  password: string;     // Min 8 chars, must contain upper + lower + number
  role?: UserRole;      // Defaults to 'member'
}

export interface UserListResponse {
  users: User[];
  total: number;
  page: number;
  pageSize: number;     // Default 20, max 100
}
```

### 2. API Contracts (Endpoint Specifications)

Define every cross-module API endpoint:

```markdown
## POST /api/auth/login

**Owner**: auth module
**Consumers**: ui-shell, products (for authenticated requests)

### Request
```json
{
  "email": "user@example.com",    // Required, valid email format
  "password": "SecurePass123"      // Required, string
}
```

### Response (200 OK)
```json
{
  "token": "eyJhbGci...",         // JWT, expires in 24h
  "user": {
    "id": "550e8400-e29b-...",
    "email": "user@example.com",
    "displayName": "John Doe",
    "role": "member"
  }
}
```

### Error Responses
| Status | Code | When |
|--------|------|------|
| 400 | `INVALID_INPUT` | Missing or malformed email/password |
| 401 | `INVALID_CREDENTIALS` | Email not found or password mismatch |
| 429 | `RATE_LIMITED` | More than 5 attempts per IP in 15 min |

### Error Format
```json
{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Invalid email or password"
  }
}
```
```

### 3. Function/Service Contracts

For internal module-to-module calls (not HTTP APIs):

```typescript
// src/shared/contracts/auth.contract.ts -- FOUNDATION MODULE

/**
 * Contract for the authentication service.
 * Implemented by: auth module
 * Consumed by: any module needing authentication checks
 */
export interface AuthServiceContract {
  /**
   * Verify a JWT token and return the authenticated user.
   * @throws AuthError with code 'TOKEN_EXPIRED' if token has expired
   * @throws AuthError with code 'TOKEN_INVALID' if token is malformed
   */
  verifyToken(token: string): Promise<User>;

  /**
   * Check if a user has the required role.
   * @returns true if user has the role, false otherwise (never throws)
   */
  hasRole(userId: string, role: UserRole): Promise<boolean>;
}
```

### 4. Database Contracts (Schema Agreements)

Define the exact schema that all modules will read from:

```sql
-- Foundation Module: database schema contract
-- ALL table definitions live here. Feature modules READ ONLY.

CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) NOT NULL UNIQUE,
  display_name VARCHAR(100) NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  role VARCHAR(20) NOT NULL DEFAULT 'member'
    CHECK (role IN ('admin', 'member', 'viewer')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Index contract: these indexes WILL exist, feature modules can rely on them
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```

### 5. Event/Message Contracts

For modules that communicate through events:

```typescript
// src/shared/contracts/events.contract.ts

export interface DomainEvent<T = unknown> {
  type: string;
  payload: T;
  timestamp: Date;
  sourceModule: string;
}

// Auth module PRODUCES these events:
export interface UserRegisteredEvent extends DomainEvent<{
  userId: string;
  email: string;
}> {
  type: 'user.registered';
  sourceModule: 'auth';
}

// Notifications module CONSUMES these events:
// It must handle UserRegisteredEvent by sending a welcome email.
```

## Mock/Stub Data for Isolated Testing

Every contract must include mock data so modules can test without their
dependencies being implemented:

```typescript
// src/shared/mocks/auth.mocks.ts -- FOUNDATION MODULE

import { User, AuthServiceContract } from '../contracts/auth.contract';

export const mockUser: User = {
  id: '550e8400-e29b-41d4-a716-446655440000',
  email: 'test@example.com',
  displayName: 'Test User',
  role: 'member',
  createdAt: new Date('2024-01-01'),
  updatedAt: new Date('2024-01-01'),
};

export const mockAuthService: AuthServiceContract = {
  verifyToken: async (token: string) => {
    if (token === 'valid-token') return mockUser;
    throw new AuthError('TOKEN_INVALID', 'Invalid token');
  },
  hasRole: async (userId: string, role: UserRole) => {
    return mockUser.id === userId && mockUser.role === role;
  },
};
```

## Contract Registry

Maintain a central registry mapping every contract to its owner and consumers:

```markdown
## Contract Registry

| Contract | Owner Module | Consumer Modules | Location |
|----------|-------------|-----------------|----------|
| User type | foundation | auth, products, orders | `src/shared/types/user.types.ts` |
| AuthService | auth | products, orders, ui-shell | `src/shared/contracts/auth.contract.ts` |
| POST /api/auth/login | auth | ui-shell | `src/shared/contracts/api/auth.api.ts` |
| Product type | foundation | products, orders, search | `src/shared/types/product.types.ts` |
| UserRegistered event | auth | notifications | `src/shared/contracts/events.contract.ts` |
```

## Contract Validation Rules

1. **Contracts live in the foundation module** -- never in feature modules
2. **Contracts are frozen after Tier 0** -- changes require an integration checkpoint
3. **Every cross-module import must go through a contract** -- no direct imports
   between feature modules
4. **Contracts include error cases** -- not just happy paths
5. **Contracts include mock data** -- agents must be able to test in isolation
6. **Contracts use concrete types** -- no `any`, no `unknown` at boundaries
7. **Contracts specify constraints** -- min/max values, required fields, formats

## Contract Change Process

When a Tier 1+ agent discovers a contract needs modification:

1. **STOP implementation** -- do not work around the contract
2. **Document the needed change** in a `CONTRACT_CHANGE_REQUEST.md` file
3. **Notify the lead/orchestrator** via Agent Teams mailbox or task update
4. **Wait for approval** -- the lead updates the foundation and notifies all agents
5. **Resume with updated contract** -- all agents pull the updated foundation

This is expensive but necessary. The alternative (agents working with incompatible
contracts) is more expensive at integration time.
