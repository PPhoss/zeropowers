# Spec Compliance Reviewer Prompt Template

Use this template when dispatching a spec compliance reviewer subagent.

**Purpose:** Verify implementer built what was specified — nothing more, nothing less. Check against acceptance criteria.

```
Task tool (general-purpose):
  description: "Review spec compliance for [feature-id]"
  prompt: |
    You are reviewing whether an implementation matches its specification.

    ## Feature

    **ID:** [feature.id]
    **Function:** [feature.function]
    **Description:** [feature.description]

    ## Acceptance Criteria

    [feature.acceptance_criteria — one per line, bulleted]

    ## Spec References

    [feature.spec_refs — one per line, if present. These define the upstream
    requirements this feature must satisfy. Cross-check implementation against
    these if acceptance criteria are ambiguous.]

    ## Files Changed

    [feature.files — one per line, bulleted]

    ## What Implementer Claims They Built

    [From implementer's report]

    ## CRITICAL: Do Not Trust the Report

    The implementer finished suspiciously quickly. Their report may be incomplete,
    inaccurate, or optimistic. You MUST verify everything independently.

    **DO NOT:**
    - Take their word for what they implemented
    - Trust their claims about completeness
    - Accept their interpretation of requirements

    **DO:**
    - Read the actual code they wrote
    - Compare actual implementation to acceptance criteria line by line
    - Check for missing pieces they claimed to implement
    - Look for extra features they didn't mention

    ## TDD Verification

    **Critical:** This feature must be test-driven. Verify:

    **Test coverage mapping:**
    - Does each acceptance criterion have at least one test?
    - Read test files — can you map each test to a specific criterion?
    - Are edge cases and error paths tested?

    **Test quality check:**
    - Do tests verify behavior (what it does) or implementation (how it works)?
    - Are tests using real code (minimal mocks)?
    - Are test names clear and descriptive?

    If you can't identify tests for an acceptance criterion → flag it immediately.

    ## Your Job

    Read the implementation AND test code. Verify against each acceptance criterion:

    **Missing requirements:**
    - Is each acceptance criterion fully satisfied?
    - Are there criteria they skipped or partially implemented?
    - Did they claim something works but didn't actually implement it?
    - **Is each criterion covered by a test?**

    **Extra/unneeded work:**
    - Did they build things not covered by any acceptance criterion?
    - Did they over-engineer or add unnecessary features?
    - Did they add "nice to haves" that weren't specified?

    **Misunderstandings:**
    - Did they interpret criteria differently than intended?
    - Did they solve the wrong problem?
    - Did they implement the right feature but wrong way?

    **Test gaps:**
    - Any acceptance criterion without test coverage?
    - Tests that don't verify actual behavior?
    - Missing edge cases or error handling tests?

    **Verify by reading code and tests, not by trusting report.**

    Report:
    - ✅ Spec compliant + Test covered (if every acceptance criterion is met, tested, and nothing extra was built)
    - ❌ Issues found: [list which criteria are unmet, untested, or what extra work was done, with file:line references]
```
