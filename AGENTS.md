# Agent instructions

Read `planning/README.md`, `planning/architecture.md`, `planning/data-model.md`, `planning/mcp-contract.md`, and `planning/security-pocketid.md` **before implementing**.

## Scope and invariants
- Repository: memDB, a standalone dataset engine + public MCP gateway; Basic Memory stays an **external dependency**.
- `planning/` is design only. Runtime Python belongs in `src/memdb/`, tests in `tests/`, deployment in `deploy/`.
- One SQLite DB; namespaces and datasets are logical entities. Never expose arbitrary SQL as an MCP tool.
- Stable UUID identifiers, normalized tags, many-to-many typed links, optimistic revision checks, idempotent writes, soft deletion, explicit schema versioning, audit history.
- Do not assume Basic Memory internals or directly modify its SQLite index. Use its supported adapter interface. External notes remain owned by Basic Memory.
- Identity must be verified at the **gateway**; private networking or proxy headers alone do not establish identity. Reject unauthenticated writes.
- Do not implement automatic background LLM interpretation, hidden autonomous ingestion, or auto-mutations.
- Avoid inventing OAuth claims, endpoints, group IDs or Pocket ID configuration; verify integration against currently deployed versions.
- Design documents describe **targets, not implemented behavior**. Flag changes in an ADR.

## Working method
1. Choose the earliest incomplete phase in `planning/roadmap.md`.
2. Write tests for failure modes (invalid schemas, access control, duplicate retries, stale revisions, migrations) before considering the phase complete.
3. Keep the MCP response envelope, pagination and error codes in `planning/mcp-contract.md` consistent with code.
4. Use additive, versioned SQLite migrations; WAL mode, foreign keys, busy timeout and transaction boundaries.
5. Never commit secrets, tokens, vault contents, food logs or production database files.
6. Update planning docs and `planning/decisions/` whenever changing a contract; include migration impact.
7. Preserve existing files unless specifically instructed to replace them.

## Diagram conventions
Diagrams belong in `planning/diagrams/` and must remain native, editable `.drawio` files. Use meaningful labels, orthogonal arrows, consistent symbols and a simple monochrome presentation. Diagrams are supplemental; normative behavior is defined in Markdown specs and tests.

## Done means
- Tests pass, existing tests remain green, README reflects implemented features, docs and examples are updated, security paths have negative tests, and the change is scoped to an explicit roadmap milestone.
