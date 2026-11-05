# Basic Memory adapter contract and capability discovery

**Status:** proposed integration boundary. Basic Memory is a separate product, with its own Markdown vault, search index, versioning and deployment. memDB does **not** vendor or modify it and must not depend directly on its private SQLite schema.

## Official interface references (verify against installed version)

- Basic Memory description: https://docs.basicmemory.com/start-here/what-is-basic-memory
- MCP tools: https://docs.basicmemory.com/reference/mcp-tools-reference
- Semantic search: https://docs.basicmemory.com/concepts/semantic-search

At planning time, the documentation lists `search_notes`, `read_note`, `write_note`, `edit_note` and `build_context`. Actual argument types and returned IDs must be validated against the deployed version. The adapter translates memDB's generic tool contract into supported Basic Memory calls rather than mirroring every Basic Memory function.

## Adapter capabilities

| Capability | Purpose | Fallback |
| --- | --- | --- |
| `search(query, filters)` | Find relevant notes or specs | Degraded local snapshot with explicit stale flag |
| `resolve(external_id)` | Read canonical note and provenance | `UPSTREAM_UNAVAILABLE` or `NOT_FOUND` |
| `build_related_context(note)` | Fetch source-linked relevant notes | Model-driven explicit next reads |
| `create_note` / `update_note` | Optional authorized write-through | Disabled if unsupported or policy forbids |
| `metadata` | Fetch title/tags/status/updated time | Cached descriptor only; marked stale |
| `health` | Connection diagnostics | Avoid failing local dataset operations |

`memory_read` should return full source text when requested, with stable provenance and available section selection. Unsupported section operations must fail clearly or return bounded full content, not fabricated support.

## Stable note mapping

Local `external_refs` rows use `provider=basic_memory` and a unique external identifier. During Phase 0 prove whether the Basic Memory ID survives rename, path move and content edits. If only path/permalink is available, treat it as a weaker identifier, store aliases, and explicitly define collision/reconciliation behavior. Do not silently link to a different note with a similar title.

## Index and cache

memDB may index **descriptors**: note ID, title, description, tags, topical triggers, source freshness. Caching full note text or private user data by default creates an additional erasure and consistency burden; avoid it in v1. Local FTS provides metadata discovery; Basic Memory provides note search and meaning-based retrieval when configured.

Cache invalidation mechanisms are version-dependent. Prefer explicit short TTL and a supported change feed if one exists. Background periodic synchronization is acceptable only for metadata and with resource limits; it does **not** require another LLM.

## Auth boundary

Use private service-to-service authentication even inside the VM when supported. The gateway must enforce user scope and per-entity permissions before asking Basic Memory to fetch private content. Service credentials to Basic Memory should not be exposed to the client or embedded in tool output. If Basic Memory has no compatible per-user ACL, enforce tenant separation in memDB and strongly consider separate Basic Memory projects or instances; do not assume upstream read permissions magically exist.

## Partial failure/reconciliation

- Basic Memory down: local dataset operations unaffected; read attempts return `UPSTREAM_UNAVAILABLE`.
- Note moved: reconcile by stable ID when supported; do not bind using just a matched title.
- Note deleted: mark local reference `unavailable`, remove sensitive cached text/metadata from search as retention rules require.
- Note updated while model reads: include timestamps and source version when exposed.
- Write-through succeeds upstream but local link fails: audit and retry local reference/link without duplicating upstream note.

## Phase-0 proofs

1. Search a note by title, tag, technical type and semantically related phrase.
2. Read all note text and an arbitrary section if available.
3. Resolve after rename, move and edit.
4. Test missing/deleted note.
5. Confirm whether upstream write_note/edit_note returns an immutable ID.
6. Check supported private/authenticated transport and service credential behavior.
7. Confirm whether a local Basic Memory instance supports the chosen search mode without a hosted inference dependency.

Record actual versions, arguments and tool transcripts in a versioned compatibility report before coding the adapter.
