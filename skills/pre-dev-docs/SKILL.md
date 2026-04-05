---
name: pre-dev-docs
description: Use when users present a product idea or requirement before starting development. Trigger phrases: "I want to build...", "help me plan a project for...", "need documentation for...", "let's design a system for..."
---

# Pre-Development Documentation

Generate pre-dev docs iteratively: PRD → Architecture → API → Database → Dev Plan (not all required every project).

## Workflow

1. **Understand** - Ask about vision, features, constraints. Skip if entering from brainstorming.
2. **Classify** - Confirm project type, select docs from table below.
3. **Check Existing** - Scan `docs/zeropowers/specs/` for existing docs. Ask: reuse/revise/regenerate?

## Document Generation Loop

```dot
digraph doc_loop {
    rankdir=TB;
    "A: Generate" -> "B: Cross-Check" -> "C: Self-Review" -> "D: User Review";
    "D: User Review" -> "Changes?" [shape=diamond];
    "Changes?" -> "C: Self-Review" [label="yes"];
    "Changes?" -> "Next Doc" [label="no"];
    "Next Doc" -> "More?" [shape=diamond];
    "More?" -> "A: Generate" [label="yes"];
    "More?" -> "Done" [label="no"];
}
```

**CRITICAL: Complete A→B→C→D for EACH document. No skipping.**

### A: Generate
Read template from `references/template-{doc}.md`, ask gap questions (max 3-5), generate.

### B: Cross-Check
Compare with previous docs. Flag conflicts to user.

### C: Self-Review

**Step 1: Read and scan the document you just generated. Check each item:**

| Check | What to scan for |
|-------|------------------|
| Placeholders | Search for "TBD", "TODO", "[", incomplete sections |
| Consistency | Read through — do any sections contradict each other? |
| Scope | Is this focused enough, or trying to do too much? |
| Ambiguity | Could any requirement be interpreted two ways? |

**Step 2: Fix any issues found. Edit the document inline.**

**Step 3: Output this table to confirm you completed the review:**

```
## 📋 Self-Review: {Doc Name}

| Check | Status | Notes |
|-------|--------|-------|
| Placeholders | ✅/⚠️ | [what you found] |
| Consistency | ✅/⚠️ | [what you found] |
| Scope | ✅/⚠️ | [what you found] |
| Ambiguity | ✅/⚠️ | [what you found] |

[If ⚠️: What you fixed]
```

**The table is proof of review, not the review itself. You must actually scan the document first.**

### D: User Review
Present to user:

> "Spec written to `<path>`. Please review and let me know if you want any changes."

Wait for approval. If changes requested, redo C→D.

## Doc Selection by Project Type

| Type | PRD | Arch | API | DB | DevPlan |
|------|-----|------|-----|-----|---------|
| Web/Full-Stack/API/Mobile | ✓ | ✓ | ✓ | ✓ | ✓ |
| CLI | ✓ | ✓ | ✗ | ✗ | ✓ |
| Desktop | ✓ | ✓ | opt | opt | ✓ |

## Red Flags — STOP

- **⚠️ No Self-Review table** → CRITICAL violation. Output table NOW.
- **Generating all docs at once** → Must iterate with user feedback.
- **Skipping PRD** → Architecture needs requirements foundation.

## Rationalizations

| Excuse | Reality |
|--------|---------|
| "I did review internally" | Internal ≠ visible. Output the table. |
| "It's straightforward" | Simple docs still have issues. Check anyway. |
| "User will catch issues" | Your job is quality, not pushing work to user. |

## Templates

| Document | Template | Key Content |
|----------|----------|-------------|
| PRD | `references/template-prd.md` | Vision, users, features, acceptance criteria |
| Architecture | `references/template-architecture.md` | Components, tech stack, security, scaling |
| API | `references/template-api.md` | Endpoints, auth, request/response formats |
| Database | `references/template-database.md` | Schema, relationships, migrations |
| Dev Plan | `references/template-dev-plan.md` | Phases, tasks, dependencies, risks |

## Output
```
docs/zeropowers/specs/{PRD,ARCHITECTURE,API,DATABASE,DEV_PLAN}.md
```
