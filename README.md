# soul.md

A portable, model-independent personal context ledger.

`soul.md` is an experiment in keeping useful AI context outside any single model provider. The repository is intended to become the canonical, inspectable source for durable context, current state, decisions and model observations. LLMs should retrieve only the context they need and contribute structured observations back through a controlled read/write interface.

> **Status:** architecture seed / V0.1

## Why

LLM memory is useful, but it is provider-specific, implicit and difficult to inspect or move. `soul.md` treats personal context as user-controlled infrastructure instead:

```text
ChatGPT work ─┐
ChatGPT personal ─┤
Claude / Codex ───┼── MCP ──> soul.md ledger
Local models ─────┘             │
                                ├─ canonical context
                                ├─ current state
                                ├─ observations
                                └─ decisions
```

The core rule is:

> **Models may append freely; canonical truth changes conservatively.**

## Repository model

```text
canonical/          stable or curated context
current/            volatile state that may change often
observations/       append-oriented model/session observations
decisions/          explicit decisions with provenance
proposed-updates/   suggestions awaiting promotion
schemas/            machine-readable contracts
docs/               architecture and operating rules
```

The repository is currently **public**, so the initial seed deliberately contains no private biography, contacts, credentials, University data or other sensitive personal context. A private deployment can later ingest the richer portable context pack.

## Intended MCP surface

Read:

- `get_context(query)`
- `get_current_state()`
- `get_recent_observations(days)`
- `get_project(id)`

Write:

- `append_observation(observation)`
- `record_decision(decision)`
- `propose_update(update)`
- `apply_volatile_update(update)`

Canonical files should not be directly writable by ordinary model sessions.

See [`docs/architecture.md`](docs/architecture.md) and [`docs/mcp-contract.md`](docs/mcp-contract.md).

## Daily context maintenance

A scheduled model can later:

1. verify MCP health;
2. inspect changes since its previous run;
3. append a short observation;
4. update volatile state where justified;
5. flag stale or contradictory context;
6. avoid rewriting durable identity without explicit promotion.

This gives multiple AI identities a shared context bus without attempting to synchronize their proprietary memories directly.

## Safety

Never commit:

- passwords, API keys, tokens or secrets;
- sensitive employer/client information;
- personal data that should not be public;
- raw conversation dumps by default.

The long-term architecture should support a **private canonical ledger** even if this public repository remains the reference implementation.