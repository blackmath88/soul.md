# Architecture

## Goal

`soul.md` is a provider-independent personal context substrate: a small, inspectable ledger that can be read and updated by multiple LLM environments without making any one provider's memory authoritative.

## Design principles

1. **User-controlled source of truth** — Git history is the durable audit trail.
2. **Read broadly, write narrowly** — retrieval can span context; writes are constrained by class.
3. **Append before mutate** — observations and decisions are safer than silent rewrites.
4. **Canonical context is curated** — stable identity/doctrine changes require promotion.
5. **Volatile state may move quickly** — current state can be updated by trusted automation.
6. **Every write has provenance** — provider, workspace/client, timestamp and confidence.
7. **No raw memory synchronization** — models exchange structured context, not proprietary hidden memory.
8. **Retrieve minimally** — return only context relevant to the current task.

## Layers

### canonical/
Durable, curated context such as working preferences, public project registry, technical doctrine and stable identity statements.

Ordinary model sessions should not directly mutate these files.

### current/
Fast-changing state: current focus, active projects, near-term commitments, temporary priorities.

This layer may be updated by trusted write tools when evidence is explicit.

### observations/
Append-oriented records contributed by LLM sessions or automations. These answer: *what did this model learn or notice that may matter later?*

An observation is not automatically truth.

### decisions/
Explicit decisions made by the user or clearly established in a conversation. These should be append-only and can supersede earlier decisions without deleting history.

### proposed-updates/
Suggested changes to canonical context. These exist so an LLM can say “this seems durable” without being allowed to rewrite identity/doctrine itself.

## Trust model

| Operation | Default policy |
| --- | --- |
| Read canonical/current | allowed |
| Append observation | allowed |
| Record explicit decision | allowed when evidence is clear |
| Update volatile current state | allowed for explicit facts |
| Propose canonical update | allowed |
| Apply canonical update | user/admin approval |
| Delete history | admin only |

## Provenance

Every model-originated write should identify at minimum:

- `timestamp`
- `source.provider`
- `source.client` or workspace
- `type`
- `summary`
- `scope`
- `confidence`
- optional `evidence`
- optional `supersedes`

This makes it possible to distinguish, for example, a work ChatGPT observation from a personal ChatGPT or Codex observation.

## Conflict handling

Newer does not always mean truer.

When records conflict:

1. explicit user statements outrank model inference;
2. explicit decisions outrank observations;
3. canonical context outranks unpromoted observations;
4. current state may supersede older current state;
5. unresolved contradictions should be surfaced, not silently reconciled.

## Daily maintenance loop

A daily scheduled client can perform a conservative maintenance cycle:

```text
health check
   ↓
read changes / recent observations
   ↓
identify meaningful deltas
   ↓
append daily observation
   ↓
update volatile state if explicit
   ↓
propose durable changes if needed
   ↓
report only breakage / meaningful change / ambiguity
```

The daily process should not manufacture updates simply to create activity.

## Future deployment

A likely lightweight implementation:

```text
LLM clients
   │
   ▼
MCP endpoint
   │
   ├─ retrieval / ranking
   ├─ validation / write policy
   └─ GitHub adapter
          │
          ▼
      soul.md repo
```

A Cloudflare Worker is a good candidate for the MCP edge, with secrets stored outside the repository. The repository itself remains the durable ledger.