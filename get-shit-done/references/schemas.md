# File Schemas Reference

<purpose>
Canonical schemas for all GSD-generated files.
Use this when creating or validating STATE.md, config.json, SUMMARY.md, or PLAN.md files.
</purpose>

<schemas>

## STATE.md

The living memory file that persists across sessions.

```markdown
# State

## Current Position
- **Milestone:** v1.0 (MVP)
- **Phase:** 03 - API Endpoints
- **Plan:** 03-02
- **Status:** executing | planning | paused | blocked

## Decisions
<!-- Append-only log. Never delete entries. -->
| # | Decision | Rationale | Phase |
|---|----------|-----------|-------|
| 1 | Use jose over jsonwebtoken | CommonJS compatibility in Next.js | 02 |
| 2 | PostgreSQL over SQLite | Multi-user concurrency required | 01 |

## Deferred Issues
<!-- Track non-blocking issues for later. -->
| # | Issue | Severity | Discovered | Phase |
|---|-------|----------|------------|-------|
| 1 | Rate limiting not implemented | medium | Phase 03 | 03 |

## Blockers
<!-- Active blockers preventing progress. Remove when resolved. -->
_None_

## Session Notes
<!-- Updated each session. Overwrite freely. -->
- Last worked: 2026-03-15
- Next step: Execute 03-02-PLAN.md
- Context: Auth middleware complete, starting API routes

## Metrics
- Plans completed: 8
- Plans remaining: 4
- Avg plan time: 3.2 min
```

### Rules
- **Current Position**: Always reflects actual state. Updated by every GSD command.
- **Decisions**: Append-only. Never edit or remove past decisions.
- **Deferred Issues**: Move to ISSUES.md at milestone completion.
- **Session Notes**: Overwritten each session — ephemeral context only.
- **Metrics**: Auto-updated by `/gsd:progress`.

---

## config.json

Project configuration. Created by `/gsd:new-project`, editable via `/gsd:settings`.

```json
{
  "version": "1.24.0",
  "mode": "interactive",
  "granularity": "standard",
  "model_profile": "balanced",
  "model_routing": {
    "enabled": true,
    "default_model": "sonnet",
    "routing": {
      "validation": "haiku",
      "file_ops": "haiku",
      "execution": "sonnet",
      "planning": "sonnet",
      "complex_planning": "opus",
      "decisions": "opus",
      "research": "opus"
    },
    "escalation": {
      "haiku_retry_limit": 2,
      "auto_escalate": true
    },
    "overrides": {
      "always_opus": ["new-project", "discuss-milestone"],
      "always_sonnet": ["execute-plan"],
      "prefer_haiku": ["progress", "help"]
    }
  },
  "adaptive_context": {
    "enabled": true
  },
  "checkpoints": {
    "enabled": true
  },
  "workflow": {
    "research": true,
    "plan_check": true,
    "verifier": true,
    "auto_advance": false,
    "node_repair": true,
    "node_repair_budget": 2
  },
  "parallelization": {
    "enabled": true
  },
  "planning": {
    "commit_docs": true
  },
  "git": {
    "branching_strategy": "none",
    "phase_branch_template": "gsd/phase-{phase}-{slug}",
    "milestone_branch_template": "gsd/{milestone}-{slug}"
  }
}
```

### Field Reference

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `version` | string | — | GSD version that created this config |
| `mode` | enum | `"interactive"` | `"yolo"` auto-approves, `"interactive"` confirms |
| `granularity` | enum | `"standard"` | `"coarse"` / `"standard"` / `"fine"` — controls phase splitting |
| `model_profile` | enum | `"balanced"` | `"quality"` / `"balanced"` / `"budget"` / `"inherit"` |
| `model_routing.enabled` | bool | `true` | Enable multi-model routing |
| `adaptive_context.enabled` | bool | `true` | Enable tiered context loading |
| `checkpoints.enabled` | bool | `true` | Enable git checkpoint tags |
| `workflow.research` | bool | `true` | Spawn research agent before planning |
| `workflow.plan_check` | bool | `true` | Verify plans before execution |
| `workflow.verifier` | bool | `true` | Verify output after execution |
| `workflow.auto_advance` | bool | `false` | Chain discuss → plan → execute |
| `workflow.node_repair` | bool | `true` | Auto-fix failed task verification |
| `workflow.node_repair_budget` | int | `2` | Max repair attempts per task |

### Defaults
All fields are optional. Missing fields use defaults shown above.
The file itself is optional — GSD works without it using all defaults.

---

## SUMMARY.md Frontmatter

Created after each plan execution. Frontmatter enables the dependency graph.

```yaml
---
phase: 03
plan: 03-02
title: API Endpoints - User Routes
status: complete
subsystem: api, auth
requires:
  - phase: 02
    provides: [JWT middleware, User model]
  - phase: 01
    provides: [Database schema, Prisma client]
provides: [User CRUD endpoints, Auth routes]
affects: [frontend, tests]
tech-stack: [Next.js, Prisma, jose]
key-files:
  - src/app/api/users/route.ts
  - src/app/api/auth/login/route.ts
---
```

### Frontmatter Fields

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `phase` | yes | int | Phase number |
| `plan` | yes | string | Plan ID (e.g., "03-02") |
| `title` | yes | string | Human-readable plan title |
| `status` | yes | enum | `complete` / `partial` / `failed` |
| `subsystem` | no | string | Comma-separated subsystem tags |
| `requires` | no | list | Phases this plan depends on |
| `requires[].phase` | — | int | Required phase number |
| `requires[].provides` | — | list | What that phase provides |
| `provides` | no | list | What this plan makes available |
| `affects` | no | list | Subsystems impacted |
| `tech-stack` | no | list | Technologies used |
| `key-files` | no | list | Important files created/modified |

### How It's Used
1. **Context loader** reads frontmatter to build dependency graph
2. **Plan-phase** uses `requires` to load only relevant prior summaries
3. **Progress** uses `status` to track completion
4. **Subsystem detection** uses `subsystem` and `affects` for smart loading

### Body Format

```markdown
## What Was Done
- Bullet list of completed work

## Key Decisions
- Decision and rationale (also logged in STATE.md)

## Files Changed
- `path/to/file.ts` — what changed and why

## Deviations
- Any deviations from the plan and why

## Issues Discovered
- Non-blocking issues found (tracked in STATE.md deferred issues)
```

---

## PLAN.md Task Format

```xml
<plan>
  <meta>
    <phase>03</phase>
    <plan_id>03-02</plan_id>
    <title>User API Routes</title>
    <files_to_read>
      src/lib/db.ts
      src/middleware/auth.ts
    </files_to_read>
  </meta>

  <tasks>
    <task type="auto">
      <name>Create user CRUD endpoints</name>
      <files>src/app/api/users/route.ts, src/app/api/users/[id]/route.ts</files>
      <action>
        Implement GET (list/single), POST (create), PUT (update), DELETE.
        Use Prisma client from src/lib/db.ts.
        Apply auth middleware to all routes.
      </action>
      <verify>
        curl localhost:3000/api/users returns 200 with user list.
        curl -X POST with valid body returns 201.
      </verify>
      <done>All CRUD operations work with auth required</done>
    </task>

    <task type="checkpoint:human-verify">
      <name>Verify API responses match frontend expectations</name>
      <criteria>
        Response shape matches TypeScript interfaces in src/types/
      </criteria>
    </task>
  </tasks>

  <acceptance_criteria>
  All user endpoints operational with authentication.
  Error responses follow consistent format.
  </acceptance_criteria>
</plan>
```

### Task Types

| Type | Frequency | Description |
|------|-----------|-------------|
| `auto` | ~90% | Fully automated — execute + verify |
| `checkpoint:human-verify` | ~9% | Pause for user confirmation |
| `checkpoint:decision` | ~0.9% | Present options, user chooses |
| `checkpoint:human-action` | ~0.1% | User must do something external |

</schemas>
