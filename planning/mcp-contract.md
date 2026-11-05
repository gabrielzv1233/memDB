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

## Mutations

| Tool | Inputs | Behavior |
| --- | --- | --- |
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

Defined errors: `UNAUTHENTICATED`, `FORBIDDEN`, `NOT_FOUND`, `INVALID_ARGUMENT`, `SCHEMA_INVALID`, `VALIDATION_FAILED`, `REVISION_CONFLICT`, `IDEMPOTENCY_CONFLICT`, `RATE_LIMITED`, `UPSTREAM_UNAVAILABLE`, `INTERNAL`. Error messages must not leak database contents or secrets.

## Auth and tool safety

All reads are filtered by entity ACL, including search hits and graph neighbors. Mutations require applicable scopes, ownership and dataset-level grants. Separate create/update/delete privileges. Strong confirmation or policy gate for bulk destructive actions, schema migrations and purges. Never execute instructions embedded in note content.

## Examples

`discover({"query":"food logging","limit":5})` may return a Basic Memory note descriptor plus linked datasets. The model reads the note, then calls `dataset_query` and `record_create`. The MCP server does **not** decide autonomously that the user ate anything.

## ChatGPT limitation

Chat-mode clients choose whether to call tools. ChatGPT built-in memory can mention this service and its purpose, but cannot force calls. Avoid relying on undocumented client hooks or silently reading every conversation.
