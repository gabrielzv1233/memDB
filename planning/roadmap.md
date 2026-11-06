# Implementation roadmap

Each phase requires docs, automated tests and a runnable acceptance demonstration.

## Phase 0: Integration spike

- Verify the live Basic Memory version's supported MCP/API calls for search, read and external note identity.
- Verify remote MCP transport, ChatGPT chat-mode connection, Pocket ID OAuth compatibility, token audience and PKCE requirements.
- Verify actual ChatGPT account plan's access to custom MCP **read and write** tools; do not promise full writes on personal Plus/Pro.
- Validate Basic Memory stable IDs, rename behavior, note metadata, search and read/write capabilities against deployed version.
- Choose Python version, FastMCP/SDK version, migration mechanism and packaging.
- Confirm trusted approval/control-plane integration can authenticate independently of model arguments; validate whether intended client supports write operations.
- Record observed endpoints/versions in an ADR. **No unsupported assumption is a valid contract.**

**Acceptance:** authenticated client calls a protected hello/read-only discovery endpoint; Basic Memory reference resolves by supported interface, with errors handled.

## Phase 1: SQLite core

- Implement migrations for entities, namespaces, datasets, schemas, records, tags, links, revisions, idempotency, audit.
- Enforce JSON Schema validation and schema version policy.
- Implement optimistic revisions, soft delete, duplicate retry protection.
- Add internal autonomy profiles, delegation grants, review plans/decisions, policy events and R0-R4 evaluation with server-owned auth context.
- Test restart persistence, concurrent writers, rollback and invalid migrations.

**Acceptance:** an agent creates a new namespace/dataset and stores/queries/tagged records without writing SQL or server code.

## Phase 2: Memory relationships and search

- External Basic Memory reference adapter, stable identity mapping and reference refresh.
- Bidirectional link traversal, missing note resilience, permission-aware discovery, FTS5.
- Add bounded `context_lookup` retrieval, source provenance, applicability/trigger metadata and active/superseded precedence.
- Test model-facing retrieval instructions without relying on unavailable Agent Skills.

**Acceptance:** one Basic Memory instruction note links to two datasets and individual records; note renames do not break links when upstream identity is stable. Chat model can retrieve sheet context via explicit tool calls; no guaranteed automatic ChatGPT activation claim.

## Phase 3: MCP and ACL hardening

- Publish final tools with typed schemas, pagination, scopes, ACL and structured errors.
- Rate limits, quotas, audit redaction, prompt-injection boundaries, schema change gates.
- Build trusted exact-diff review/approval control plane (not model-callable), revocation, and single-use approval checks.
- Run all cases in `planning/autonomy/test-cases.md` including anti-replay, escalation, partial failure and conflict handling.
- Verify negative tests and real remote MCP client.

**Acceptance:** unauthorized search/link/mutation requests cannot reveal or change data. Scoped R1/R2 work without repeated confirmation as permitted; R3 needs exact verified review; R4 always denied.

## Phase 4: Proxmox release

- VM deployment manifests, internal Basic Memory connection, protected HTTPS endpoint.
- Backup/restore, health checks, metrics, logging, release procedure and upgrade tests.
- Threat review and operational runbook.

**Acceptance:** fresh VM deployment from docs works, restore drill succeeds and secrets stay out of the repo.

## Later, optional

- More expressive typed dataset joins and uniqueness constraints
- Semantic embeddings and reranking
- Web UI for registry/links
- Bulk import/export, dry-run automation, scheduled maintenance
- Multi-user sharing, delegated access and synchronization

Do not implement optional features before the core contracts are tested.
