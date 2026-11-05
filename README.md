# memDB

A self-hosted, agent-accessible **structured dataset engine and MCP gateway**, designed to link flexible SQLite datasets with knowledge stored in a separate Basic Memory instance.

> **Status:** planning / architecture. No runtime implementation has been committed yet.

## Start here

- [Planning index](planning/README.md)
- [Architecture](planning/architecture.md)
- [Data model](planning/data-model.md)
- [MCP tool contract](planning/mcp-contract.md)
- [Pocket ID and authorization](planning/security-pocketid.md)
- [Implementation roadmap](planning/roadmap.md)
- [Agent handoff](AGENTS.md)

## Repository boundaries

| Location | Purpose |
| --- | --- |
| `planning/` | Requirements, technical sheets, diagrams, examples, decisions and acceptance tests |
| `src/memdb/` | Future Python gateway and SQLite engine implementation |
| `tests/` | Future automated tests |
| `deploy/` | Future VM/container manifests, migrations and operational scripts |

**Basic Memory is not vendored or forked here.** memDB will interact with it through a versioned adapter using supported MCP/API methods.

Nothing in `planning/` should be mistaken for a deployed or implemented feature. Technical specifications are proposals until verified by tests and approved decisions.

## Design principles

1. One SQLite file can host many logical namespaces and datasets.
2. Agents can define datasets, fields, metadata, tags, and relationships without adding tools or writing arbitrary SQL.
3. Notes in Basic Memory and datasets/records in memDB share linkable identities but retain separate ownership.
4. All dataset writes are validated, auditable, transactional, versioned, and permission-checked.
5. One remotely accessible MCP endpoint sits behind authenticated HTTPS; internal services are not directly published.
6. The service works without running a generative LLM. Client models invoke tools; the server indexes, validates and persists data.

See [planning/README.md](planning/README.md) for the complete design map.
