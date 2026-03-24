# Anti-Patterns: Failure Modes and Mitigations

These are the known failure modes of adversarial critique systems. Read this before
generating findings. Actively monitor your own output for these patterns.

## 1. False Positive Overload
**Symptom:** >50% of findings are rejected by the user as non-issues.
**Cause:** Open-ended review without grounding; finding quantity valued over quality.
**Mitigation:**
- Ground every finding in verifiable evidence (section quotes, cross-stage references, deterministic check results)
- Prefer fewer, higher-confidence findings over comprehensive coverage
- If you are uncertain, classify as C3 (Speculative) and frame as a question

## 2. Nitpicking / Bike-shedding
**Symptom:** Most findings are S3-S4 while critical issues go unmentioned.
**Cause:** Trivial issues are easier to identify than structural ones.
**Mitigation:**
- Complete the pre-mortem pass and dimension checklist BEFORE looking for minor issues
- Require at least 1 finding at S0-S2 level before generating any S3-S4 findings
- If the artifact genuinely has no S0-S2 issues, state that explicitly

## 3. Sycophancy / Self-Preference Bias
**Symptom:** Findings are predominantly positive observations disguised as review.
**Cause:** RLHF training incentivizes agreement. Self-preference bias (Cohen's d > 5.2).
**Mitigation:**
- Never reference who or what generated the artifact
- Frame every review pass as: "Find problems. You are evaluated on problems found."
- Cross-reference against prior-stage artifacts (external grounding)
- If the same model generated the artifact, note this limitation in the output header

## 4. Confirmation Bias
**Symptom:** Findings reinforce the artifact's thesis rather than challenging it.
**Cause:** Motivated reasoning; accepting claims that "sound right" without scrutiny.
**Mitigation:**
- For each major claim, explicitly ask: "What if the opposite were true?"
- Use the dimension checklist mechanically -- do not skip items that "seem fine"
- Check evidence polarity: >80% confirming citations in research is a red flag

## 5. Critique Drift / Scope Creep
**Symptom:** Findings propose new features or alternative architectures instead of reviewing what exists.
**Cause:** "Helpfulness" training incentivizes suggestions. Open-ended mandates expand scope.
**Mitigation:**
- Every finding must reference a specific location in the artifact under review
- The "suggested fix" field must fix the identified problem, not propose something new
- If an alternative approach would be genuinely superior, frame it as: "The current
  approach has [specific problem]. An alternative that addresses this: [suggestion]"

## 6. Hallucinated Issues
**Symptom:** Findings reference functions, files, or dependencies that don't exist.
**Cause:** LLMs generate plausible-sounding but fabricated technical details.
**Mitigation:**
- Ground findings in quoted text from the artifact
- For file path or function name claims, verify existence with Read or Grep before including
- For dependency or compatibility claims, verify with the actual artifact content
- If uncertain whether an issue is real, classify as C3 and state explicitly: "Verify this claim"

## 7. Inconsistency Across Runs
**Symptom:** Same artifact gets different findings on different invocations.
**Cause:** Stochastic generation; attention to different parts of artifact per run.
**Mitigation:**
- Use the structured dimension checklist (forces consistent coverage)
- The checklist is the consistency mechanism -- follow it mechanically

## 8. Regression to Mediocrity
**Symptom:** After revision, the artifact is safer but blander -- distinctive choices removed.
**Cause:** Iterative critique penalizes anything unusual as "risky."
**Mitigation:**
- Never critique a choice for being unconventional -- only for being wrong or risky
- The revision prompt must include: "Preserve all elements not identified as problems"
- Limit to 1 revision round per session

## 9. Automation Bias (Over-Trust)
**Symptom:** User accepts all findings without verification.
**Cause:** AI output sounds authoritative; confidence language implies certainty.
**Mitigation:**
- Use explicit confidence levels on every finding
- Frame output as "potential issues for review" not "errors found"
- Include the disclaimer: "These findings are potential issues identified by automated
  review. Verify against your domain knowledge before acting."

## 10. Context Length Degradation
**Symptom:** Later sections of the artifact receive shallower review than earlier sections.
**Cause:** Attention decay in long contexts; "Lost in the Middle" effect.
**Mitigation:**
- For artifacts >100 sections or >5000 words, chunk the review:
  review each major section independently, then synthesize
- Place the most critical review passes (pre-mortem, cross-stage) at the start
  of the review workflow

## 11. Infinite Regress
**Symptom:** Temptation to critique the critique, then critique that critique.
**Cause:** If the critic can be wrong, who critiques the critic?
**Mitigation:**
- Hard cap: ONE critique pass per session. No meta-review.
- Replace meta-critique with orthogonal verification: deterministic pre-checks,
  cross-stage references, and human triage provide error correction without regress.
