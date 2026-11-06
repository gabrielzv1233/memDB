# Public MCP contract (proposed v1)

The public API is one remotely accessible MCP gateway. The names and payloads below are **design targets**, not existing tools. No free-form SQL tool. Keep tool responses concise and predictable so ChatGPT chat mode can retrieve context effectively.

## Discovery and read

| Tool | Inputs | Outputs |
| --- | --- | --- |
| `context_lookup` | task, kinds?, topic_hints?, max_results?, max_tokens? | Compact relevant descriptors with link/read suggestions, partial failure info |
| `discover` | query, kinds?, tags?, namespace?, limit? | Descriptors, scores, linked entity counts, cursor |
| `entity_describe` | entity_id, include_links?, direction? | Metadata, schema summary, selected links, revision |
| `memory_read` | memory_ref_id, section?, max_chars? | Resolved Basic Memory content, source provenance, freshness |
| `dataset_query` | dataset_id, filters?, fields?, sort?, cursor?, limit? | Records, stable cursor, schema version |
| `link_traverse` | entity_id, types?, direction?, depth?, limit? | Graph neighbors; default depth 1, strict cap |
| `schema_get` | dataset_id, version? | JSON Schema, constraints, version |
| `autonomy_profile_get` | none | Current caller's mode, effective delegations and quotas, sensitive fields redacted |
| `review_status` | review_id | Caller-visible pending/approved/rejected/expired/applied status; no self-approval |

## Mutations

| Tool | Inputs | Behavior |
| --- | --- | --- |
| `policy_preview` | operation, target_ids?, change_payload? | Side-effect-free classification, diff summary, required gates |
| `dataset_plan` | namespace, slug, title, description, schema, tags? | Dry-run for collisions, validation and grants; no mutation |
| `namespace_create` | slug, title, description, tags?, idempotency_key | Create a namespace (subject to grants) |
| `dataset_create` | namespace, slug, title, description, tags?, schema?, idempotency_key | Register dataset and initial schema |
| `dataset_update` | dataset_id, metadata changes, expected_revision | Modify metadata only |
| `schema_propose` | dataset_id, new_schema, compatibility_mode | Dry-run validation + impact report, no mutation |
| `schema_apply` | proposal_id, expected_revision, idempotency_key | Privileged, auditable schema change |
| `record_create` | dataset_id, payload, tags?, idempotency_key | Validate and insert |
| `record_update` | record_id, payload/patch, expected_revision, idempotency_key | Validate and conditionally update |
| `record_delete` | record_id, expected_revision, idempotency_key | Soft-delete; no destructive purge |
| `external_ref_register` | provider, external_id, metadata?, idempotency_key | Create/refresh a pointer to a Basic Memory note |
| `link_create` | source_id, target_id, relation_type, idempotency_key | Validate IDs/permissions, create typed link |
| `link_delete` | link_id, expected_revision, idempotency_key | Soft-delete link |

Memory note write/search remains Basic Memory's job; optionally expose an authorized `memory_save` write-through tool later, backed by its supported endpoints, rather than reimplementing them. This tool is not assumed to be available in all ChatGPT plans. A generic "context" helper may join discovery and one-hop linked metadata, but must respect a token/output budget and permissions.

## Autonomy and approval contract

All writes must pass the [autonomy policy](autonomy/policy.md) and [deterministic evaluator](autonomy/policy-evaluator.md). All mutations accept an idempotency key and applicable expected revisions; R3 tools additionally accept a server-issued `review_id` identifying an exact user-approved plan. A model-supplied `approved=true` flag is ignored/rejected. `policy_preview` and `schema_propose` may create **reviewable plans**, not effects. The model cannot call a generic `approval_grant` or `delegation_create` tool. Grants, review approvals and user mode changes are performed in a separate verified control plane described in [delegations and approvals](autonomy/delegation-and-approvals.md).

Policy errors must return `REVIEW_REQUIRED` with an opaque review reference only when the caller may see that plan, or `POLICY_DENIED`, `DELEGATION_EXPIRED`, `LIMIT_EXCEEDED`, `APPROVAL_INVALID`, `APPROVAL_STALE`, plus the existing authorization and revision errors. No error may leak protected titles, counts, or link endpoints. If an approved operation touches Basic Memory, report partial cross-service effects and reconcile; do not claim atomicity.

### Consent boundaries

The gateway cannot verify raw ChatGPT chat intent from a model's claim. Routine writes require valid permission and a configured scoped policy or user-directed workflow through a trusted client. High-risk R3 effects need independently verified exact-plan approval. No hidden transcript capture. A no-write instruction must not cause an autonomous save.

## Context lookup semantics

`context_lookup` is an optional convenience orchestration tool: it combines exact/metadata/Basic Memory note discovery (as supported), filters inaccessible results, returns concise descriptors and at most one-hop relation **hints**, and recommends `memory_read`, `entity_describe` or `dataset_query` as subsequent calls. It never automatically uploads full dataset contents or executes writes. The gateway receives task text **only when explicitly passed by the client**. Default result budget: 8 descriptors / 1,500 tokens, subject to config. Unknown or offline provider yields precise partial-results status.

## Tool response envelope

```json
{
  "ok": true,
  "data": {"entity_id": "uuid", "revision": 1},
  "meta": {"request_id": "uuid", "next_cursor": null}
}
```

Failure:
```json
{
  "ok": false,
  "error": {
    "code": "REVISION_CONFLICT",
    "message": "Record changed since it was read",
    "details": {"current_revision": 4}
  },
  "meta": {"request_id": "uuid"}
}
```

Defined errors: `REVIEW_REQUIRED`, `POLICY_DENIED`, `DELEGATION_EXPIRED`, `LIMIT_EXCEEDED`, `APPROVAL_INVALID`, `APPROVAL_STALE`, `UNAUTHENTICATED`, `FORBIDDEN`, `NOT_FOUND`, `INVALID_ARGUMENT`, `SCHEMA_INVALID`, `VALIDATION_FAILED`, `REVISION_CONFLICT`, `IDEMPOTENCY_CONFLICT`, `RATE_LIMITED`, `UPSTREAM_UNAVAILABLE`, `INTERNAL`. Error messages must not leak database contents or secrets.

## Auth and tool safety

All reads are filtered by entity ACL, including search hits and graph neighbors. Mutations require applicable scopes, ownership and dataset-level grants. Separate create/update/delete privileges. Strong confirmation or policy gate for bulk destructive actions, schema migrations and purges. Never execute instructions embedded in note content.

## Examples

`discover({"query":"food logging","limit":5})` may return a Basic Memory note descriptor plus linked datasets. The model reads the note, then calls `dataset_query` and `record_create`. The MCP server does **not** decide autonomously that the user ate anything.

## ChatGPT limitation

Chat-mode clients choose whether to call tools. ChatGPT built-in memory can mention this service and its purpose, but cannot force calls. Avoid relying on undocumented client hooks or silently reading every conversation.
