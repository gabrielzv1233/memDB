# Implementation handoff for coding agents

## Mission

Implement the memDB **SQLite dataset engine and unified MCP gateway**, without forking/reimplementing Basic Memory. Preserve `planning/` as architecture and behavioral reference. Keep runtime under `src/memdb/`, tests under `tests/`, deployment under `deploy/`.

## Read order

1. `../AGENTS.md`
2. `README.md`
3. `architecture.md`
4. `context-discovery.md` and `model-instructions.md`
5. `metadata-ontology.md` and `data-model.md`
6. `dataset-lifecycle.md` and `mcp-contract.md`
7. `adapter-basic-memory.md`
8. `autonomy/README.md`, `autonomy/policy.md`, `autonomy/delegation-and-approvals.md`, `autonomy/policy-evaluator.md`, `autonomy/reconciliation.md`, `autonomy/dataset-organization.md` and `autonomy/test-cases.md`
9. `security-pocketid.md` and `client-compatibility.md`
10. `acceptance-scenarios.md` and `roadmap.md`

## Work packages

### 0. Feasibility spike

Prove remote authenticated MCP read access from an actual intended client, document ChatGPT plan/permission limitations and validate Basic Memory's supported APIs and stable note identifiers. Record blocking limitations early.

### 1. Storage skeleton

Choose migration tool and Python packaging, implement one SQLite file, schema, FTS and transaction boundaries. Unit-test entity IDs, namespaces, metadata, ACL and audit. Implement internal delegation/review tables and evaluator without exposing them as generic datasets.

### 2. Dynamic datasets

Implement schema registry, JSON Schema validation, generic record operations, revisions, idempotency, record tags and bounded query filters. No arbitrary SQL tool. Test accidental duplicates and malicious JSON payloads.

### 3. Knowledge integration

Adapter to separate Basic Memory, external reference reconciliation, links in both directions and ACL-filtered discovery. Avoid copying whole notes into local DB. Implement `context_lookup` returning small descriptors and explicit next reads.

### 4. Security, autonomous approvals and operations

Pocket ID-compatible OAuth, token verification, per-namespace grants, R0-R4 classification, delegation revocation, trusted exact-plan review, rate limiting, Proxmox deployment, backup/restore, health checks and threat tests.

### 5. Client instructions and real demos

Publish compact model-facing tool descriptions; test flowchart rules and food database round-trip from an authorized client. Record client compatibility without claiming unsupported ChatGPT write actions.

## Issue conventions

Suggested labels: `phase:0` through `phase:4`, `component:gateway`, `component:storage`, `component:adapter`, `component:auth`, `type:doc`, `type:test`, `blocked:client`. These are **suggestions**, not labels currently created in GitHub.

## PR checklist

- [ ] Tests cover the relevant acceptance scenario IDs.
- [ ] Tool names and payloads match `mcp-contract.md`.
- [ ] Schema changes and migration/rollback documented.
- [ ] No raw SQL, secrets, production data or private vault in Git.
- [ ] Permissions apply to search, traversal, read and write.
- [ ] Mutations pass R0-R4 policy gates, with independent trusted approval for R3 and no model-minted grants.
- [ ] All relevant `AUT-*` cases from the autonomy matrix are tested or explicitly blocked.
- [ ] Tool outputs include provenance and precise error status.
- [ ] Any changed product design has a decision document.
- [ ] No claim that docs represent implemented features.
