---
name: pre-dev-docs
description: Use when users present a product idea or requirement before starting development. Trigger phrases: "I want to build...", "help me plan a project for...", "need documentation for...", "let's design a system for..."
---

# Pre-Development Documentation

Generate complete pre-dev documentation iteratively, one document at a time with user feedback.

## When to Use

**Trigger symptoms:**
- User describes a product idea or feature request
- User asks to "plan", "design", or "document" a project
- User mentions specific doc types: PRD, architecture, API, database, dev plan

**Skip if:** User just wants code implementation (no pre-dev docs needed)

**Context:** May be invoked directly by user, or by zeropowers:brainstorming after design approval. If entering from brainstorming, design decisions are already made — skip to Phase 2.

## Document Order (Each Builds on Previous)

1. **PRD** → 2. **Architecture** → 3. **API** → 4. **Database** → 5. **Dev Plan**

Not all projects need all 5 docs. Document selection is determined in Phase 2.

## Workflow

### Phase 0: Check Existing Specs

Scan `zeropowers/specs/` for existing documents before starting. For each found:

1. Review completeness — search for "TBD", "TODO", "[", placeholder sections
2. Ask user: "Found existing `{doc}.md` — reuse, revise, or regenerate?"

Only generate docs the user hasn't approved. Resume from the next missing doc in order.

### Phase 1: Understand

Ask targeted questions about vision, core features, tech constraints, and project scope.

Skip if entering from brainstorming (design already approved).

### Phase 2: Classify Project Type & Select Docs

Confirm project type with user:

Web App / Mobile App / API Service / Full-Stack / Microservices / Desktop / CLI

Then select required documents based on project type:

| Project Type | PRD | Architecture | API | Database | Dev Plan |
|-------------|-----|-------------|-----|----------|----------|
| Web App / Full-Stack | required | required | required | required | required |
| API Service / Microservices | required | required | required | required | required |
| Mobile App | required | required | required | required | required |
| CLI Tool | required | required | skip | skip | required |
| Desktop App | required | required | optional | optional | required |

Confirm the doc list with user before proceeding. "This project needs: PRD → Architecture → Dev Plan. Sound right?"

### Phase 3: Generate Iteratively

For each document in the agreed list:
1. Read `references/template-{doc}.md`
2. Tell user what you're generating
3. Ask gap questions (see guidelines below)
4. Generate using template structure
5. **Cross-document consistency check:** Does this doc conflict with any previously generated spec? If yes, flag to user before proceeding.
6. Get user feedback before next

**Critical:** Before Architecture doc, discuss technology stack with user.

**Gap questions should be:**
- Specific to sections you can't fill from previous docs or conversation
- Limited to 3-5 questions max per document
- NOT questions already answered in previous docs or brainstorming

### Phase 4: Spec Self-Review

After generating the spec document, review with fresh eyes:

1. **Placeholder scan:** Any "TBD", "TODO", incomplete sections, or vague requirements? Fix them.
2. **Internal consistency:** Do any sections contradict each other? Does the architecture match the feature descriptions?
3. **Scope check:** Is this focused enough for a single implementation plan, or does it need decomposition?
4. **Ambiguity check:** Could any requirement be interpreted two different ways? If so, pick one and make it explicit.

Fix any issues inline. No need to re-review — just fix and move on.

### Phase 5: User Review Gate

After the spec review loop passes, ask the user to review the written spec before proceeding:

> "Spec written and committed to `<path>`. Please review it and let me know if you want to make any changes before we start writing out the implementation plan."

Wait for the user's response. If they request changes, make them and re-run the spec review loop. Only proceed once the user approves.

After user approves the final spec, suggest next step:

> "Specs approved. Ready to create an implementation plan? I can invoke the writing-plans skill."

## Quick Reference

| Document | Template | Key Content |
|----------|----------|--------------|
| PRD | `references/template-prd.md` | Vision, users, features, acceptance criteria |
| Architecture | `references/template-architecture.md` | Components, tech stack, security, scaling |
| API | `references/template-api.md` | Endpoints, auth, request/response formats |
| Database | `references/template-database.md` | Schema, relationships, migrations |
| Dev Plan | `references/template-dev-plan.md` | Phases, tasks, dependencies, risks |

## Output Location
```
project-root/zeropowers/specs/
├── PRD.md / ARCHITECTURE.md / API.md / DATABASE.md / DEV_PLAN.md
```

## Red Flags — STOP

- **Generating all docs at once** → Must be iterative with user feedback
- **Skipping PRD** → Architecture needs requirements foundation
- **No tech stack discussion before architecture** → Technology choices required first
- **Using placeholder text instead of asking** → If missing info, ask the user
- **Skipping user review** → Spec needs user approval before implementation
- **Skipping self-review before user review** → You must pass Phase 4 before Phase 5

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Generating all docs at once | One doc, get feedback, repeat |
| Wrong order | PRD → Architecture → API → DB → Dev Plan |
| Over-specifying | Leave room for implementation decisions |
| Ignoring dependencies | Dev plan must reflect actual task dependencies |
| Asking too many gap questions | Max 3-5 per document, only for real gaps |
| Skipping consistency check | New doc must not contradict previous docs |
