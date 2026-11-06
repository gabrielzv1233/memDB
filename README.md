# memDB

A self-hosted, agent-accessible **structured dataset engine and MCP gateway**, linking flexible SQLite datasets with knowledge stored in a separate Basic Memory instance.

> **Status:** planning / architecture only. No runtime implementation is committed.

## Start here

**For autonomous operations:** [Autonomy policy suite](planning/autonomy/README.md) · [Risk matrix](planning/autonomy/policy.md) · [Trusted approvals](planning/autonomy/delegation-and-approvals.md)

**For model/context behavior:** [Context discovery](planning/context-discovery.md) · [Client instructions](planning/model-instructions.md) · [Memory lifecycle](planning/memory-lifecycle.md) · [Client compatibility](planning/client-compatibility.md)


- [Planning index](planning/README.md)
- [Architecture](planning/architecture.md)
- [Data model](planning/data-model.md)
- [MCP tools](planning/mcp-contract.md)
- [Pocket ID and authorization](planning/security-pocketid.md)
- [Proxmox operations](planning/operations.md)
- [Roadmap](planning/roadmap.md)
- [Agent instructions](AGENTS.md)
- [Detailed handoff for coding agents](planning/agent-handoff.md)
- [Acceptance scenarios](planning/acceptance-scenarios.md)
- [Editable diagrams](planning/diagrams/)

## Separation of concerns

| Path | Purpose |
| --- | --- |
| `planning/` | Architecture, technical sheets, editable diagrams, decisions, examples |
| `src/memdb/` | Future Python gateway and SQLite dataset engine |
| `tests/` | Future automated tests |
| `deploy/` | Future Proxmox VM/container manifests and operations |

Basic Memory is **not** included or forked. memDB will link to its separately deployed instance through an adapter.

## Main idea

An AI creates logical namespaces and datasets inside one SQLite database, chooses versioned schemas and metadata, stores validated records, and links datasets or records to one or more notes in Basic Memory. Everything has discoverable names, descriptions, tags, relationships and IDs.

The server does not run a second generative LLM; the existing MCP client model decides when to call tools. A short user-controlled ChatGPT memory pointer can remind that richer external knowledge exists, but cannot guarantee tool invocation. The gateway enforces authentication, permissions, safe queries and transactional changes.
