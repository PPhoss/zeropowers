# Feature Manager Script

Persistent, cross-session feature tracking for implementation plans.

**Location:** `skills/subagent-driven-development/scripts/feature-manager.py`

**Also used by:** `skills/executing-plans`

## Why This Script?

**Problem:** TodoWrite is session-scoped and loses state across sessions.

**Solution:** Use the feature list JSON as the single source of truth with this management script.

| Feature | TodoWrite | feature-manager.py |
|---------|-----------|-------------------|
| **Persistence** | ❌ Session only | ✅ File system |
| **Cross-session** | ❌ Lost | ✅ Preserved |
| **Dependency tracking** | ❌ Manual | ✅ Automatic |
| **Topological ordering** | ❌ Manual | ✅ Built-in |

## Installation

```bash
# No dependencies required - uses only Python standard library
chmod +x skills/subagent-driven-development/scripts/feature-manager.py
```

## Commands

### Check Status

```bash
python3 skills/subagent-driven-development/scripts/feature-manager.py status openspec/changes/<dir>/plan.json
```

Output:
```
Feature List Status: openspec/changes/<dir>/plan.json
  Total: 5
  ✅ Done: 1
  🔄 In Progress: 1
  ⏳ Pending: 3
  🚫 Blocked: 2
  ⏭️  Skipped: 0

Currently in progress:
  - auth-002: token-refresh

Progress: 20.0%
```

### Get Next Feature

```bash
python3 skills/subagent-driven-development/scripts/feature-manager.py next openspec/changes/<dir>/plan.json
```

Returns the next feature to work on, respecting:
- Status must be 'pending'
- All dependencies must be 'done'
- Topological order (array order in JSON)

Output:
```
Next feature: auth-002
  Category: authentication
  Function: token-refresh
  Description: Implement token refresh endpoint

Acceptance Criteria:
  1. Valid refresh token returns new access token
  2. Invalid refresh token returns 401
  3. Old refresh token is invalidated

Dependencies: auth-001

Files:
  - src/auth/refresh.ts
  - tests/auth/refresh.test.ts
```

### Start Feature

```bash
python3 skills/subagent-driven-development/scripts/feature-manager.py start openspec/changes/<dir>/plan.json auth-002
```

Updates JSON:
```json
{
  "id": "auth-002",
  "status": "in_progress"  // was: "pending"
}
```

### Complete Feature

```bash
python3 skills/subagent-driven-development/scripts/feature-manager.py complete openspec/changes/<dir>/plan.json auth-002
```

Updates JSON:
```json
{
  "id": "auth-002",
  "status": "done"  // was: "in_progress"
}
```

### List Blocked Features

```bash
python3 skills/subagent-driven-development/scripts/feature-manager.py blocked openspec/changes/<dir>/plan.json
```

Output:
```
Blocked Features: 2

  auth-003: logout
    Waiting on: auth-002

  user-002: update-profile
    Waiting on: user-001
```

## Feature List JSON Format

```json
[
  {
    "id": "auth-001",
    "category": "authentication",
    "function": "user-login",
    "description": "Implement email/password login",
    "acceptance_criteria": [
      "Valid credentials return JWT token",
      "Invalid password returns 401"
    ],
    "files": ["src/auth/login.ts", "tests/auth/login.test.ts"],
    "dependencies": [],
    "status": "pending"
  }
]
```

**Required fields:**
- `id`: Unique identifier
- `status`: `pending` | `in_progress` | `done` | `skipped`
- `dependencies`: List of feature IDs (can be empty)

## Error Handling

### Circular Dependencies

```bash
❌ Circular dependencies detected:
  auth-001 → auth-002 → auth-003 → auth-001
```

### Invalid Feature ID

```bash
❌ Feature 'auth-999' not found
```

### Invalid Status Transition

```bash
❌ Feature 'auth-001' is not pending (current: done)
```

## Integration with Skills

This script is used by:
- `subagent-driven-development` - Track features across subagents
- `executing-plans` - Track features during sequential execution

## Advanced Usage

### Check Progress in Scripts

```bash
# Get progress percentage
python3 skills/subagent-driven-development/scripts/feature-manager.py status plan.json | grep "Progress:" | awk '{print $2}'
# Output: 40.0%
```

### List All Done Features

```bash
# Parse JSON directly
jq '.[] | select(.status == "done") | .id' plan.json
```

### Find Features by Category

```bash
jq '.[] | select(.category == "authentication") | .id' plan.json
```
