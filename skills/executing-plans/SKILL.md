---
name: executing-plans
description: Use when you have a feature list JSON file to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load the feature list JSON file, review critically, execute all features sequentially, report when complete.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** Tell your human partner that Superpowers works much better with access to subagents. The quality of its work will be significantly higher if run on a platform with subagent support (such as Claude Code or Codex). If subagents are available, use zeropowers:subagent-driven-development instead of this skill.

## The Process

### Step 1: Load and Review Plan
1. Read the feature list JSON file from `openspec/changes/<dir>/plan.json`
2. Review critically — identify any questions or concerns about the features, dependencies, or acceptance criteria
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Check status with feature script and proceed

   ```bash
   python3 skills/subagent-driven-development/scripts/feature-manager.py status openspec/changes/<dir>/plan.json
   ```

### Step 2: Execute Features

Use the feature manager script to get and track features:

```bash
# Get next feature (respects dependencies)
python3 skills/subagent-driven-development/scripts/feature-manager.py next openspec/changes/<dir>/plan.json

# Mark feature as started
python3 skills/subagent-driven-development/scripts/feature-manager.py start openspec/changes/<dir>/plan.json <feature-id>
```

Then for each feature:
1. Invoke `zeropowers:test-driven-development` via the Skill tool before writing any code — do NOT attempt TDD from memory
2. Follow the loaded TDD skill's Red-Green-Refactor workflow exactly
3. Verify all acceptance criteria are met
4. Commit
5. Mark feature as completed:

   ```bash
   python3 skills/subagent-driven-development/scripts/feature-manager.py complete openspec/changes/<dir>/plan.json <feature-id>
   ```

### Step 3: Complete Development

After all features done and verified:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use zeropowers:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Feature has critical gaps preventing implementation
- You don't understand an acceptance criterion
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** - stop and ask.

## Remember
- Review plan critically first
- Follow acceptance criteria — they define done
- Don't skip verifications
- Update feature status in JSON after each completion (enables cross-session resume)
- Stop when blocked, don't guess
- Never start implementation on main/master branch without explicit user consent

## Integration

**Required workflow skills:**
- **zeropowers:writing-plans** - Creates the feature list JSON file this skill executes
- **zeropowers:finishing-a-development-branch** - Complete development after all features

**Shared tools:**
- **Feature manager script** (`skills/subagent-driven-development/scripts/feature-manager.py`) - Persistent feature tracking across sessions
