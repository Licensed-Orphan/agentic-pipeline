# Research Phase Checklist

Research is mandatory before writing any agentic PRD. Skipping this step is the
single most common cause of AI agent implementation failures. AI agents execute
specifications as written -- incomplete research compounds into cascading errors.

## Existing Codebase Research

If the user has an existing project, explore it before specifying anything:

### Architecture & Patterns
- [ ] Read entry points (`index.ts`, `main.py`, `app.ts`, etc.)
- [ ] Identify the framework and its version
- [ ] Map the directory structure and file organization conventions
- [ ] Identify state management patterns (Redux, Context, Zustand, etc.)
- [ ] Note the component/module organization pattern
- [ ] Check for monorepo structure (workspaces, packages)

### Data Layer
- [ ] Read database schema files or migration history
- [ ] Identify the ORM/query builder in use (Prisma, Drizzle, SQLAlchemy, etc.)
- [ ] Map existing models and their relationships
- [ ] Check for seed data or fixtures
- [ ] Note any caching layers (Redis, in-memory, etc.)

### API Layer
- [ ] Map existing API routes and their patterns (REST, GraphQL, tRPC)
- [ ] Identify authentication/authorization mechanisms
- [ ] Note request validation patterns (Zod, Joi, class-validator)
- [ ] Check for API versioning conventions
- [ ] Read middleware chain and order

### Testing Infrastructure
- [ ] Identify test runner and framework (Jest, Vitest, Pytest, etc.)
- [ ] Note test file naming conventions (`*.test.ts`, `*.spec.ts`, `__tests__/`)
- [ ] Check for test utilities, factories, or fixtures
- [ ] Read CI/CD pipeline configuration
- [ ] Note coverage thresholds if configured

### Code Style
- [ ] Check for linter configuration (ESLint, Prettier, Ruff, etc.)
- [ ] Note naming conventions (camelCase, snake_case, kebab-case)
- [ ] Identify import ordering conventions
- [ ] Check for TypeScript strictness settings
- [ ] Note error handling patterns (try/catch style, Result types, error boundaries)

## Platform & Technology Research

Search the web for current information. Always include the current date in queries
to avoid stale results from the LLM's training data.

### Core Platform
- [ ] "[Platform] capabilities and limitations [current date]"
- [ ] "[Platform] supported languages and frameworks [current date]"
- [ ] "[Platform] database and storage options [current date]"
- [ ] "[Platform] deployment options and constraints [current date]"
- [ ] "[Platform] environment variables and secrets management"

### Specific Technologies
- [ ] "[Framework] [platform] compatibility [current date]"
- [ ] "[Library] latest version breaking changes [current date]"
- [ ] "[Service] API documentation [current date]"
- [ ] "Known issues [technology stack] [current date]"

### Platform-Native vs External Services
Always prefer platform-native services over external integrations:
- Native services have simpler configuration and fewer failure modes
- External services introduce authentication complexity agents handle poorly
- Document which services are native vs external in the PRD research summary

## Competitive & Domain Research (If Applicable)

For new products or features:
- [ ] Identify 2-3 comparable products and their approaches
- [ ] Note UX patterns that are standard in the domain
- [ ] Check for regulatory or compliance requirements
- [ ] Identify accessibility standards that apply (WCAG level)

## Research Summary Format

Compile findings into a concise summary for the PRD:

```markdown
## Research Summary

### Verified Platform Capabilities
- [Capability]: [How it works, any constraints]
- [Capability]: [How it works, any constraints]

### Existing Codebase Patterns
- Framework: [name] v[version]
- Database: [type] via [ORM]
- Testing: [runner] with [pattern] conventions
- Style: [key conventions]

### Constraints Discovered
- [Constraint with source/evidence]
- [Constraint with source/evidence]

### Risks & Unknowns
- [Risk]: [Mitigation approach]
- [Unknown]: [How to resolve during implementation]
```
