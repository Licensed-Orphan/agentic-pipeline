# Machine-Verifiable Acceptance Criteria

AI coding agents cannot interpret subjective criteria. "Fast," "intuitive," and
"user-friendly" are meaningful to human engineers who apply professional judgment.
AI agents require quantified thresholds, specific behaviors, and conditions testable
within their environment.

## The Translation Problem

| Human Criterion | Why It Fails for AI | Machine-Verifiable Version |
|----------------|--------------------|-----------------------------|
| "Page loads quickly" | "Quickly" is undefined | "Page renders in < 200ms (LCP metric)" |
| "Intuitive navigation" | "Intuitive" requires user testing | "All primary actions reachable within 2 clicks from home" |
| "Handles errors gracefully" | "Gracefully" is subjective | "All API errors return `{error: {code, message}}` format, UI displays error component with retry button" |
| "Secure authentication" | "Secure" is unbounded | "Passwords hashed with bcrypt (cost 12+), sessions expire after 24h, rate limit 5 failed attempts per IP per 15min" |
| "Responsive design" | "Responsive" lacks breakpoints | "Layout adapts at 640px, 768px, 1024px breakpoints. No horizontal scroll at any width >= 320px" |
| "Easy to use" | Completely subjective | "New user can complete core flow (signup -> first action) in < 3 steps" |

## Criteria Formats

### Gherkin / Given-When-Then (Preferred)

Best for behavior-driven criteria that map directly to automated tests:

```gherkin
Scenario: Successful user registration
  Given the registration form is displayed at /register
  When the user submits email "test@example.com" and password "ValidPass123!"
  Then a new user record exists in the users table with email "test@example.com"
  And the response status is 201
  And the response body contains a valid JWT token
  And the user is redirected to /dashboard

Scenario: Registration with duplicate email
  Given a user with email "existing@example.com" already exists
  When a new user submits registration with email "existing@example.com"
  Then the response status is 409
  And the response body contains {"error": {"code": "EMAIL_EXISTS"}}
  And no new user record is created

Scenario: Registration with weak password
  Given the registration form is displayed
  When the user submits email "test@example.com" and password "123"
  Then the response status is 400
  And the response body contains {"error": {"code": "WEAK_PASSWORD"}}
  And the password field displays an inline error message
```

### Checklist Format

Best for verification steps the user performs manually:

```markdown
### Manual Verification
- [ ] Navigating to /dashboard shows the user's data
- [ ] Clicking "Delete" opens a confirmation dialog before deleting
- [ ] Submitting an empty form shows validation errors on all required fields
- [ ] The page renders correctly at 320px viewport width
- [ ] All images have alt text attributes
```

### Command-Based Format

Best for automated verification the agent can run:

```markdown
### Automated Verification
Run after implementation:

```bash
# Unit tests pass
npm test -- --grep "UserRegistration"

# API endpoint responds correctly
curl -X POST http://localhost:3000/api/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"ValidPass123!"}' \
  | jq '.token' # should return a non-null string

# Type checking passes
npx tsc --noEmit

# Linting passes
npm run lint
```
```

### Threshold Format

Best for performance and quality metrics:

```markdown
### Performance Criteria
| Metric | Threshold | Tool |
|--------|-----------|------|
| Largest Contentful Paint | < 2.5s | Lighthouse |
| First Input Delay | < 100ms | Lighthouse |
| Cumulative Layout Shift | < 0.1 | Lighthouse |
| Bundle size (gzipped) | < 200KB | `npm run build` output |
| API response time (p95) | < 500ms | Load test with k6 |
| Test coverage | > 80% lines | `npm test -- --coverage` |
```

## Writing Effective Criteria

### Rules

1. **One behavior per criterion** -- do not combine multiple assertions
2. **Include specific values** -- "returns 201" not "returns success"
3. **Specify the negative case** -- what happens when input is invalid
4. **Include the verification method** -- how can the agent or user check this
5. **Use full paths** -- `/src/routes/auth.ts` not "the auth file"

### Anti-Patterns

Avoid these -- they are unverifiable by AI agents:

- "The system should be performant" (what threshold?)
- "Handle edge cases appropriately" (which edge cases? what handling?)
- "Follow best practices" (whose best practices? which ones?)
- "Maintain backward compatibility" (with what? how to verify?)
- "The code should be clean and well-organized" (by what measure?)

### AI-Specific Criteria

If the feature incorporates AI/ML capabilities, add:

```markdown
### AI-Specific Acceptance Criteria
- Model accuracy: >= [X]% on [labeled dataset of N samples]
- Hallucination rate: < [X]% measured against [ground truth source]
- Response latency: < [X]ms at p95 under [concurrent users] load
- Graceful degradation: When AI service is unavailable, [fallback behavior]
- Bias audit: [Criteria for fairness across user demographics]
```

## Criteria Coverage Checklist

For each phase, ensure acceptance criteria cover:

- [ ] **Happy path**: The primary success scenario
- [ ] **Validation errors**: Invalid inputs are rejected with clear messages
- [ ] **Authorization**: Unauthorized access is blocked appropriately
- [ ] **Empty states**: What renders when there is no data
- [ ] **Error states**: What happens when external services fail
- [ ] **Boundary conditions**: Maximum lengths, minimum values, edge cases
- [ ] **Regression**: All prior phase functionality still works
