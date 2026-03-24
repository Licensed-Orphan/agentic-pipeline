# Critique Module: PRD Stage

## Pre-Mortem Prompt

"The feature shipped exactly as specified in this PRD and was a disaster. Users
hated it, it caused production incidents, and the team spent 3 months rebuilding.
What did the PRD get wrong?"

## Dimension Checklist

### 1. Requirements Completeness (IEEE 830)
- Score against 8 quality attributes: correct, unambiguous, complete, consistent,
  ranked, verifiable, modifiable, traceable
- Build a traceability matrix: every research finding maps to a requirement;
  every requirement traces to research or stakeholder need
- Search for contradictory requirement pairs (e.g., "respond within 100ms" +
  "encrypt all data with RSA-4096 at rest and in transit")
- Verify presence of: functional, non-functional (performance, security,
  accessibility, reliability), interface, data, constraints, and assumptions

### 2. Acceptance Criteria Verifiability
- For each criterion, attempt a Gherkin conversion (Given-When-Then) -- if it
  fails, the criterion is too vague
- Classify each: Fully automatable, Semi-automatable, Manual-only, Untestable
- Flag any "untestable" criteria as S1
- Check for weasel words: "fast," "user-friendly," "reasonable," "appropriate,"
  "as needed" -- each must be replaced with specific thresholds

### 3. Scope Creep / Non-Goal Sufficiency
- For every stated goal, check for at least one explicit non-goal at the boundary
- Look for scope inflation signals: "and more," "including but not limited to,"
  "future-proof," "extensible"
- Verify MoSCoW classification exists (Must/Should/Could/Won't) with populated Won't
- For each feature, identify the simplest adjacent extension -- if the PRD doesn't
  explicitly exclude it, flag as scope creep vector

### 4. Edge Case Coverage (EARS Patterns)
- Check coverage of all 5 EARS patterns:
  - Ubiquitous: Always-active requirements (no trigger)
  - Event-driven (When): System response to triggering events
  - State-driven (While): Requirements active during specific states
  - Unwanted behavior (If/Then): Response to undesired situations
  - Optional feature (Where): Product-specific requirements
- For each input, identify: minimum, maximum, zero, null/empty, negative, overflow,
  unicode/special characters, concurrent access
- Check error taxonomy: network failure, auth failure, data corruption, rate limiting,
  partial success, idempotency violations

### 5. Security, Performance, Accessibility
- **Security:** Authentication, authorization, encryption (rest + transit), input
  validation, audit logging, rate limiting, CSRF/XSS. Apply OWASP Top 10 as checklist.
- **Performance:** Quantified targets for latency (p50, p95, p99), throughput,
  concurrent users, data volume, degradation behavior under load
- **Accessibility:** WCAG 2.1 AA reference, keyboard navigation, screen reader,
  color contrast, focus management
- **Reliability:** SLA/SLO targets, RTO/RPO, graceful degradation

### 6. User Story Quality (INVEST)
- Score each story: Independent, Negotiable, Valuable, Estimable, Small, Testable
- Verify every identified persona has at least one story
- Map stories against user journey -- look for gaps (creation but not deletion,
  happy path but not error recovery)
- Check "so that" clause quality -- generic benefits indicate weak stories

## Deterministic Pre-Checks

- Verify all file paths in the PRD are absolute (not bare filenames)
- Check that every phase has a DO NOT CHANGE section (after Phase 1)
- Count requirements per phase -- flag any phase with >50 requirements
- Verify each phase ends with verification commands the agent can run
