# MCP Contract

This document defines the intended V0 tool surface for `soul.md`.

## Read tools

### `get_context(query, scopes?, limit?)`
Return only context relevant to a task.

Expected behavior:
- search canonical, current, recent decisions and relevant observations;
- prefer explicit/canonical information over inference;
- include provenance;
- do not return secrets or excluded scopes;
- keep output compact.

### `get_current_state(scopes?)`
Return the current volatile state for the requested scopes.

### `get_recent_observations(days = 7, scopes?, sources?)`
Return recent structured observations, newest first.

### `get_project(id)`
Return canonical project context plus current state, recent decisions and selected observations.

## Write tools

### `append_observation(observation)`
Append one observation to the ledger.

Allowed by default.

Required fields:
- `timestamp`
- `source`
- `summary`
- `scope`
- `importance`
- `confidence`

### `record_decision(decision)`
Record an explicit user decision.

Use only when the user actually made or confirmed the decision. Do not convert model recommendations into decisions.

### `propose_update(update)`
Propose a change to canonical context.

The tool writes a proposal, not the canonical file itself.

### `apply_volatile_update(update)`
Update state under `current/` when the fact is explicit, current and suitable for automatic maintenance.

Examples:
- project status changed from planned to deployed;
- an event was completed;
- a workspace was created;
- onboarding is now pending.

Not suitable:
- personality changes;
- inferred preferences;
- reinterpretation of career identity;
- sensitive personal conclusions.

## Example observation

```json
{
  "timestamp": "2026-09-14T18:00:00+02:00",
  "source": {
    "provider": "chatgpt",
    "client": "unibas-business"
  },
  "type": "observation",
  "scope": ["unibas", "ld", "ai-pilot"],
  "summary": "ChatGPT Business workspace created and all six L&D members invited.",
  "importance": "high",
  "confidence": "high",
  "evidence": "Explicitly stated by user",
  "suggested_updates": [
    {
      "target": "current/work.json",
      "field": "chatgpt_business.members_invited",
      "value": true
    }
  ]
}
```

## Write policy

The MCP server should validate writes against schemas and reject arbitrary file paths. Clients should never receive a generic `write_file(path, content)` capability.

Recommended mapping:

| MCP tool | Repository area |
| --- | --- |
| `append_observation` | `observations/` |
| `record_decision` | `decisions/` |
| `propose_update` | `proposed-updates/` |
| `apply_volatile_update` | `current/` |

Canonical promotion should be a separate privileged operation, ideally requiring explicit user approval.

## Idempotency

Write tools should accept or derive a stable event ID. Repeated calls with the same ID should not create duplicate observations.

## Concurrency

Prefer append-only daily JSONL files or one-record-per-file writes to reduce merge conflicts between multiple clients.

Suggested path:

```text
observations/2026/09/14/<timestamp>-<source>-<id>.json
```

rather than having every client edit one large shared JSON document.