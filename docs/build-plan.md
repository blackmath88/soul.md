# Build Plan

## V0.1 — ledger contract ✅

- repository purpose and safety boundary
- architecture and trust model
- read/write MCP contract
- observation schema
- public-safe current state

## V0.2 — working MCP edge

Implement a small authenticated MCP service, preferably as a Cloudflare Worker.

### Required tools

Read:
- `get_context`
- `get_current_state`
- `get_recent_observations`
- `get_project`

Write:
- `append_observation`
- `record_decision`
- `propose_update`
- `apply_volatile_update`

### Constraints

- no generic filesystem write tool
- server-side allowlist of writable repository prefixes
- JSON Schema validation before write
- idempotent observation IDs
- GitHub token stored only as deployment secret
- authenticated MCP endpoint
- dry-run/test mode for writes
- provenance injected/validated server-side where possible
- useful errors rather than silent fallback

### Tests

At minimum:

1. health endpoint works;
2. MCP tool discovery works;
3. read current state;
4. append valid observation;
5. duplicate ID does not duplicate;
6. malformed observation rejected;
7. canonical direct write rejected;
8. volatile update limited to `current/`;
9. unauthorized request rejected;
10. GitHub/API failure does not corrupt ledger.

## V0.3 — context retrieval

Start simple before adding embeddings.

1. scope/tag filtering;
2. recency weighting for volatile state;
3. precedence: decisions/canonical > observations;
4. keyword/text matching;
5. compact response pack with provenance.

Only add vector search if the corpus becomes large enough that deterministic retrieval is insufficient.

## V0.4 — multi-client provenance

Register clients such as:

```json
{
  "chatgpt-work": {
    "provider": "chatgpt",
    "class": "work"
  },
  "chatgpt-personal": {
    "provider": "chatgpt",
    "class": "personal"
  },
  "codex": {
    "provider": "openai",
    "class": "builder"
  }
}
```

The ledger should preserve where an observation came from without assuming one client is inherently correct.

## V0.5 — daily maintenance

Create a scheduled ChatGPT task/client routine:

1. call MCP health/read tools;
2. inspect changes since last check;
3. append a concise daily observation only when meaningful;
4. apply explicit volatile updates;
5. propose durable context changes rather than applying them;
6. report breakage, contradictions, stale state or high-value changes.

A quiet day should be allowed to produce no write.

## V1 — private personal context

The current repository is public. Do not add the detailed portable personal profile here until the storage/security decision is explicit.

Likely options:

- make `soul.md` private and use it as the canonical ledger;
- keep this public repo as the reference implementation and point the MCP deployment at a second private data repo;
- separate public doctrine/schema from private context entirely.

The second option gives the cleanest open-source/public-demo boundary while keeping actual personal context private.