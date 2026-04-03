---
name: pre-dev-docs
description: Use when users present a product idea or requirement before starting development. Trigger phrases: "I want to build...", "help me plan a project for...", "need documentation for...", "let's design a system for...", or when users mention PRD, system design, API specs, database schema, development plan, or UI guidelines.
---

# Pre-Development Documentation

Generate complete pre-dev documentation iteratively, one document at a time with user feedback.

## When to Use

**Trigger symptoms:**
- User describes a product idea or feature request
- User asks to "plan", "design", or "document" a project
- User mentions specific doc types: PRD, architecture, API, database, dev plan, UI specs

**Skip if:** User just wants code implementation (no pre-dev docs needed)

## Document Order (Each Builds on Previous)

1. **PRD** → 2. **Architecture** → 3. **API** → 4. **Database** → 5. **Dev Plan**

## Workflow

### Phase 1: Understand
Ask targeted questions about vision, core features, tech constraints, and project scope.

### Phase 2: Classify Project Type
Web App / Mobile App / API Service / Full-Stack / Microservices / Desktop / CLI

Confirm type with user before proceeding.

### Phase 3: Generate Iteratively
For each document:
1. Read `references/template-{doc}.md`
2. Tell user what you're generating
3. Ask any gap questions
4. Generate using template structure
5. Get user feedback before next

**Critical:** Before Architecture doc, discuss technology stack with user.

### Phase 4: Final Review
Ensure consistency across all docs; verify dev plan matches architecture.

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
project-root/docs/
├── PRD.md / ARCHITECTURE.md / API.md / DATABASE.md / DEV_PLAN.md
```

## Red Flags — STOP

- **Generating all docs at once** → Must be iterative with user feedback
- **Skipping PRD** → Architecture needs requirements foundation
- **No tech stack discussion before architecture** → Technology choices required first
- **Using placeholder text instead of asking** → If missing info, ask the user
- **Skipping user review** → Each doc needs approval before moving on

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Generating all docs at once | One doc, get feedback, repeat |
| Wrong order | PRD → Architecture → API → DB → Dev Plan → UI |
| Over-specifying | Leave room for implementation decisions |
| Ignoring dependencies | Dev plan must reflect actual task dependencies |
