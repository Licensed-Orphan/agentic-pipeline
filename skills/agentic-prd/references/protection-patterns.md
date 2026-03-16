# Protection Patterns & Constraint Specification

AI coding agents, trained on vast codebases and optimized for helpfulness, will
attempt to fulfill unstated requirements their training suggests. Unchecked, they
will "improve" working code into broken code, refactor stable patterns into novel
architectures, and expand scope beyond specification intent. Explicit constraints
are essential.

## The "DO NOT CHANGE" Pattern

Include a DO NOT CHANGE section in every phase after Phase 1. This section grows
with each phase as more functionality becomes stable.

```markdown
### DO NOT CHANGE
- `/src/db/schema.ts` -- database schema is frozen after Phase 1
- `/src/lib/auth.ts` -- authentication logic must not be modified
- `/src/routes/api/users.ts` -- user API endpoints are complete
- All test files from prior phases -- do not modify existing tests
```

### Why This Is Necessary

Human engineers make risk assessments through institutional knowledge and code
review norms. They know intuitively that "this code could be better" and "this
code must remain stable" are different situations. AI agents at current capability
levels lack this contextual judgment. They will refactor working authentication
code because it does not match the latest patterns in their training data.

### Rules for DO NOT CHANGE

1. **Use full file paths** -- never "the auth file" or "database code"
2. **Be specific about what aspect is protected** -- "schema is frozen" vs
   "the entire db directory is off-limits"
3. **Cumulative growth** -- each phase adds to the list, never removes from it
4. **Include both files and behaviors** -- "do not change API response format"
   alongside "do not change `/src/routes/api/v1/`"

## Non-Goals

AI cannot infer scope from omission. If you do not explicitly say "do not implement
user authentication in this phase," an agent might add authentication because most
web applications require it. Every boundary must be stated positively.

```markdown
## Non-Goals (DO NOT Implement)

- DO NOT implement user roles or permissions beyond basic auth
- DO NOT add real-time/WebSocket functionality
- DO NOT build admin panel or back-office tools
- DO NOT implement email notifications or messaging
- DO NOT add internationalization (i18n) support
- DO NOT optimize for mobile native (web responsive only)
- DO NOT implement analytics or tracking
- DO NOT add social login (OAuth with Google/GitHub/etc.)
```

### How to Identify Non-Goals

Ask: "What would a helpful AI agent probably try to add that we do NOT want?"

Common scope creep from AI agents:
- Adding authentication when the spec does not mention it
- Implementing dark mode / theme switching
- Adding analytics or logging infrastructure
- Building admin interfaces
- Adding export/import functionality
- Implementing caching layers
- Adding WebSocket connections for "real-time" features
- Creating API documentation endpoints (Swagger/OpenAPI)
- Adding rate limiting or throttling
- Building comprehensive error tracking (Sentry integration)

If any of these are NOT wanted, state it explicitly.

## The Always / Ask First / Never Framework

Organize all constraints into three tiers:

```markdown
## Boundaries

### Always (required in every phase, every commit)
- Run the test suite before considering a phase complete
- Use TypeScript strict mode -- no `any` types
- Handle all Promise rejections explicitly
- Validate user input at the API boundary
- Follow existing naming conventions in the codebase
- Write tests for new functionality (minimum: happy path + one error case)
- Use environment variables for all configuration values

### Ask First (pause and get user approval before proceeding)
- Adding any new npm/pip/cargo dependency
- Changing database schema beyond current phase scope
- Modifying public API response formats
- Altering authentication or authorization logic
- Creating new database tables not specified in the phase
- Changing build or deployment configuration

### Never (categorically prohibited)
- Committing secrets, API keys, or credentials to source files
- Using `eval()`, `exec()`, or dynamic code execution
- Disabling TypeScript strict checks or ESLint rules
- Modifying files in node_modules/, vendor/, or .git/
- Removing or commenting out failing tests
- Adding TODO/FIXME/HACK comments as implementation substitutes
- Using `!important` in CSS (unless overriding third-party styles)
- Storing passwords in plain text or reversible encryption
- Using `any` type as a workaround for type errors
```

## Compensating for Known AI Weaknesses

CodeRabbit's 2025 report found AI-generated code has systematic weaknesses.
The PRD must explicitly require what AI tends to omit:

### Error Handling (AI omits null checks, guard clauses, exception handling)

```markdown
### Error Handling Requirements
- All API endpoints must return structured error responses: `{error: {code, message}}`
- All async operations must have try/catch with specific error handling
- All user-facing errors must display a helpful message (not raw error text)
- All network requests must handle timeout, connection refused, and 5xx errors
- Form validation errors must appear inline next to the relevant field
- Null/undefined checks required before accessing nested object properties
```

### Security (AI generates 1.5-2x more security vulnerabilities)

```markdown
### Security Requirements
- Sanitize all user input before database queries (parameterized queries only)
- Validate and sanitize all user input before rendering (XSS prevention)
- Use CSRF tokens for all state-changing operations
- Set secure, httpOnly, sameSite flags on all cookies
- Implement rate limiting on authentication endpoints
- Never log sensitive data (passwords, tokens, PII)
- Use HTTPS for all external API calls
```

### Performance (AI favors simplicity over efficiency)

```markdown
### Performance Requirements
- Database queries must use indexes for all WHERE and JOIN conditions
- No N+1 query patterns -- use eager loading or batch queries
- Paginate all list endpoints (default: 20 items, max: 100)
- Lazy-load images and heavy components below the fold
- Bundle size must not exceed [X]KB gzipped
```

### Code Quality (AI generates 3x more readability issues)

```markdown
### Code Quality Requirements
- Functions must not exceed 50 lines (extract helpers for complex logic)
- No deeply nested conditionals (max 3 levels -- use early returns)
- No magic numbers -- use named constants
- No duplicate code -- extract shared logic into utility functions
- All exported functions must have JSDoc comments describing parameters and return
```

## Mid-Project Adaptation Patterns

When the user needs changes after implementation has started:

### Data Model Change
**Impact**: High -- cascades through all dependent phases.
**Action**: Update the affected phase AND all subsequent phases that touch the
changed data. Re-run verification for all completed phases.

### New Feature Addition
**Impact**: Low if isolated, high if it touches existing functionality.
**Action**: Add as a new phase at the end. Include DO NOT CHANGE for all existing
code. If it requires changes to existing code, create a focused "modification phase"
that specifies exactly what changes in existing files.

### Scope Reduction
**Impact**: Simplifies the remaining work.
**Action**: Remove the phase(s) covering the cut feature. Update dependency chains.
Verify no orphaned code or incomplete integrations remain.

### Technology Change
**Impact**: High -- may require rebuilding phases.
**Action**: Research the new technology first. Rewrite affected phases from scratch
rather than trying to patch existing specifications. Protect all unaffected phases
with DO NOT CHANGE.
