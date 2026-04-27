---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Break implementation work into atomic, trackable features. Each feature describes WHAT to build, not HOW — implementation details are left to the executing agent.

Output is a feature list JSON file that serves as the execution contract: agents pick up features by status, know what's done, and can resume seamlessly across sessions.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Locate output dir:** Scan `openspec/changes/` to find the target change directory (created externally). If multiple exist, ask user which one. All output goes to `openspec/changes/<dir>/`.

**Save plan to:** `openspec/changes/<dir>/plan.json`

## OpenSpec Directory Structure

Before writing any plan, you MUST understand the three zones of the openspec directory. Each zone has a different role and different authority:

```
openspec/
├── specs/                          # Zone 1: Main Specs (ALREADY IMPLEMENTED)
│   ├── feature-a/
│   │   └── spec.md
│   ├── feature-b/
│   │   └── spec.md
│   └── ...
├── changes/
│   ├── <active-change>/            # Zone 2: Active Change (THIS IS WHAT WE'RE PLANNING)
│   │   ├── .openspec.yaml
│   │   ├── design.md
│   │   ├── proposal.md
│   │   ├── tasks.md
│   │   ├── API.yaml
│   │   ├── DATABASE.md
│   │   ├── specs/
│   │   │   └── ...
│   │   └── plan.json               # ← Your output goes here
│   └── archive/                    # Zone 3: Archived Changes (ALREADY IMPLEMENTED)
│       ├── YYYY-MM-DD-<change-name>/
│       │   ├── API.yaml
│       │   ├── DATABASE.md
│       │   ├── design.md
│       │   ├── plan.json
│       │   ├── proposal.md
│       │   ├── specs/
│       │   │   └── ...
│       │   └── tasks.md
│       └── ...
└── config.yaml
```

### Zone Authority

| Zone | Path | Status | How to Use |
|------|------|--------|------------|
| **Main Specs** | `openspec/specs/` | **Already implemented** | Source of truth for existing system behavior. These specs are synchronized from completed changes. Your plan must be compatible with these. Do NOT re-plan anything already defined here. |
| **Active Change** | `openspec/changes/<dir>/` | **To be implemented** | This is the PRIMARY input. Everything in this directory is new work that needs a plan. This is what you are planning right now. |
| **Archived Changes** | `openspec/changes/archive/` | **Already implemented** | Historical record of completed work. Reference for context on past decisions, but these features are DONE. Do NOT include archived work in your plan. |

### Reading Strategy

**Step 1 — Read the active change (PRIMARY):**
```
openspec/changes/<dir>/
├── .openspec.yaml           # Change metadata
├── design.md                # Overall design (IMPORTANT)
├── proposal.md              # Change proposal (IMPORTANT)
├── tasks.md                 # Task breakdown from external tool (read to inform plan)
├── API.yaml                 # API Specifications (Swagger 2.0)
├── DATABASE.md              # Database Schema
└── specs/
    ├── feature-a/
    │   └── spec.md          # Per-feature spec (IMPORTANT)
    └── ...
```

Read priority for the active change:
1. `design.md`, `proposal.md` — understand the full change scope
2. `specs/*/spec.md` — each feature's specific requirements
3. `tasks.md` — understand existing task decomposition, use to inform your plan
4. `API.yaml`, `DATABASE.md` — technical interface contracts

All planning decisions must reference specific sections/sentences from these specs. If no specs exist yet, use **REQUIRED BACKGROUND:** zeropowers:pre-dev-docs to generate them first.

**Step 2 — Read main specs (CONTEXT):**

Scan `openspec/specs/` to understand what already exists in the system. These represent implemented requirements. Your plan must:
- Be compatible with existing specs
- Not duplicate or conflict with already-implemented behavior
- Reference existing specs where the new change extends or modifies them

**Step 3 — Skim archived changes (OPTIONAL, only if needed):**

Look at `openspec/changes/archive/` when you need:
- Historical context on why past decisions were made
- Understanding of implementation patterns already established
- Clarity on how previous changes structured their plans

Do NOT plan features for anything found in archive — it's already built.

## Spec Document Reference

Each document type serves a specific purpose in the planning pipeline:

| Document | Purpose | Key Content |
|----------|---------|--------------|
| design.md | Overall architecture | System design, key decisions |
| proposal.md | Change scope | What and why of this change |
| specs/*/spec.md | Per-feature requirements | Detailed feature specs |
| tasks.md | Task decomposition | Existing task breakdown |
| API.yaml | Interface contract | Endpoints, auth, request/response formats |
| DATABASE.md | Data model | Schema, relationships, migrations |

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
    "spec_refs": ["openspec/changes/<dir>/specs/user-auth/spec.md#login-flow", "openspec/changes/<dir>/API.yaml#POST /auth/login"],
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
| `spec_refs` | no | References to upstream spec documents with full paths (e.g. `openspec/changes/<dir>/specs/user-auth/spec.md#login-flow`, `openspec/changes/<dir>/API.yaml#POST /auth/login`) |
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

**"Plan complete and saved to `openspec/changes/<dir>/plan.json`. Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per feature, review between features, fast iteration

**2. Inline Execution** - Execute features in this session using executing-plans, sequential execution with checkpoints

**Which approach?"**

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use zeropowers:subagent-driven-development
- Fresh subagent per feature + two-stage review

**If Inline Execution chosen:**
- **REQUIRED SUB-SKILL:** Use zeropowers:executing-plans
- Sequential execution with checkpoints for review
