---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Break implementation work into atomic, trackable features. Each feature describes WHAT to build, not HOW — implementation details are left to the executing agent.

Output is a feature list JSON file that serves as the execution contract: agents pick up features by status, know what's done, and can resume seamlessly across sessions.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Save plans to:** `docs/zeropowers/plans/YYYY-MM-DD-<feature-name>.json`
- (User preferences for plan location override this default)

## Spec Documents Location

**REQUIRED:** Before writing any plan, read ALL spec documents in:
```
project-root/zeropowers/specs/
├── API.yaml             # API Specifications (Swagger 2.0)
└── DATABASE.md          # Database Schema
```

All planning decisions must reference specific sections/sentences from these specs. If no specs exist yet, use **REQUIRED BACKGROUND:** zeropowers:pre-dev-docs to generate them first.

## Spec Document Reference

Each spec type serves a specific purpose in the planning pipeline:

| Document | Purpose | Key Content |
|----------|---------|--------------|
| API | Interface contract | Endpoints, auth, request/response formats |
| Database | Data model | Schema, relationships, migrations |

**Dependencies:** API → Database. API data models inform database schema design.

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining features, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the feature decomposition. Each feature should produce self-contained changes that make sense independently.

## Feature Granularity

Each feature is one independently testable unit of work — small enough for an agent to complete in a single focused session, large enough to deliver meaningful value.

**Good feature:** "User login with email and password, returns JWT token"
**Bad feature:** "Authentication system" (too large)
**Bad feature:** "Add `import jwt` to auth module" (too small, no independent value)

## Dependency Ordering

Features must be ordered in topological sort order — a feature appears before any feature that depends on it. When writing the JSON array:

1. Start with features that have no dependencies
2. Follow with features whose dependencies are already listed
3. Continue until all features are placed
4. Verify: no feature references a dependency that appears later in the array

This ordering ensures consumers can process features sequentially without re-sorting.

## JSON Schema

```json
[
  {
    "id": "auth-001",
    "category": "authentication",
    "function": "user-login",
    "description": "Implement email/password login. Validate credentials against database, return JWT token on success.",
    "acceptance_criteria": [
      "Valid email+password returns 200 with JWT token",
      "Wrong password returns 401 Unauthorized",
      "Non-existent email returns 401 (must not reveal whether user exists)"
    ],
    "files": ["src/auth/login.ts", "tests/auth/login.test.ts"],
    "dependencies": [],
    "spec_refs": ["API.yaml#POST /auth/login"],
    "status": "pending"
  }
]
```

### Field Definitions

| Field | Required | Description |
|-------|----------|-------------|
| `id` | yes | Unique identifier, used in dependencies references. Format: `{category-prefix}-{number}` |
| `category` | yes | Module/subsystem this feature belongs to |
| `function` | yes | Feature name, concise and specific |
| `description` | yes | What to build — enough context for an agent to make good implementation decisions, but NO code |
| `acceptance_criteria` | yes | List of verifiable conditions. Spec reviewers check against these. |
| `files` | yes | File paths this feature will create or modify. Must be specific paths, not patterns. |
| `dependencies` | yes | List of feature IDs that must be completed first. Empty array if none. |
| `spec_refs` | no | References to upstream spec documents (e.g. `API.yaml#POST /auth/login`, `DATABASE.md#users-table`) |
| `status` | yes | `pending` / `in_progress` / `done` / `skipped` / `blocked`. Default: `pending` |

## Self-Review

After writing the complete feature list, check against the spec. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a feature that implements it? List any gaps.

**2. Dependency validity:** Every ID in a `dependencies` array must exist as another feature's `id`. No circular dependencies.

**3. File path consistency:** Do the files referenced across features align? If feature A creates a module that feature B uses, do the paths match?

**4. Acceptance criteria quality:** Each criterion must be objectively verifiable — not vague ("works correctly"), but specific ("returns 401 for invalid password").

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no feature, add the feature.

## Execution Handoff

After saving the plan, offer execution choice:

**"Plan complete and saved to `docs/zeropowers/plans/<filename>.json`. Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per feature, review between features, fast iteration

**2. Inline Execution** - Execute features in this session using executing-plans, sequential execution with checkpoints

**Which approach?"**

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use zeropowers:subagent-driven-development
- Fresh subagent per feature + two-stage review

**If Inline Execution chosen:**
- **REQUIRED SUB-SKILL:** Use zeropowers:executing-plans
- Sequential execution with checkpoints for review
