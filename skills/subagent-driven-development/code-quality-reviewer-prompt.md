# Code Quality Reviewer Prompt Template

Use this template when dispatching a code quality reviewer subagent.

**Purpose:** Verify implementation is well-built (clean, tested, maintainable)

**Only dispatch after spec compliance review passes.**

```
Task tool (zeropowers:code-reviewer):
  Use template at requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [from implementer's report]
  PLAN_OR_REQUIREMENTS: Feature [feature.id] "[feature.function]" — [feature.description]
  ACCEPTANCE_CRITERIA: [feature.acceptance_criteria]
  BASE_SHA: [commit before feature]
  HEAD_SHA: [current commit]
  DESCRIPTION: [feature summary]
```

## Coverage Verification

**The reviewer MUST run coverage analysis:**

```bash
# Detect and run appropriate coverage command
if [ -f "package.json" ]; then
  npm test -- --coverage
  # or: npm run test:coverage
elif [ -f "Cargo.toml" ]; then
  cargo tarpaulin --out Stdout
elif [ -f "requirements.txt" ]; then
  pytest --cov --cov-report=term-missing
elif [ -f "go.mod" ]; then
  go test -coverprofile=coverage.out ./...
  go tool cover -func=coverage.out
fi
```

**Coverage requirements:**
- Minimum 80% line coverage for new code
- Identify uncovered lines in changed files
- Flag if coverage tool not configured

**In addition to standard code quality concerns, the reviewer should check:**

**Test Quality (TDD Verification):**
- Can you identify the failing test that drove each piece of implementation?
- Do tests verify behavior (not just mock interactions)?
- Are edge cases and error paths tested?
- Is test coverage actually meaningful (not just hitting lines)?

**Code Organization:**
- Does each file have one clear responsibility with a well-defined interface?
- Are units decomposed so they can be understood and tested independently?
- Does the implementation follow the file structure specified in the feature?
- Did this implementation create new files that are already large, or significantly grow existing files? (Don't flag pre-existing file sizes — focus on what this change contributed.)

**Code reviewer returns:** 
- Coverage report (percentage + uncovered lines)
- Strengths
- Issues (Critical/Important/Minor)
- Assessment (including coverage verdict)
