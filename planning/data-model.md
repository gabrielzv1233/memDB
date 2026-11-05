# Data model and SQLite engine specification (target)

## Design constraints

Use **one physical SQLite database** with unlimited logical `namespace.dataset` collections. Use a **generic record store** with validated JSON payloads for v1 rather than creating arbitrary SQL tables per dataset. Dynamic schemas are registry entries, not SQL DDL. This avoids arbitrary SQL and disruptive migrations for every agent-defined dataset.

The following is the **proposed conceptual schema**, not executable DDL. Formal migrations and constraints belong in implementation.

| Table | Key fields | Purpose |
| --- | --- | --- |
| `entities` | id UUID PK, kind, namespace_id?, title, description, created_at, updated_at, deleted_at | Universal index for namespaces, datasets, records and external notes |
| `namespaces` | entity_id FK, slug UNIQUE, owner_principal, visibility | Logical grouping |
| `datasets` | entity_id FK, namespace_id FK, slug UNIQUE per namespace, active_schema_version, options_json | Dataset metadata |
| `dataset_schemas` | dataset_id FK, version, json_schema, created_at, actor | Immutable schema versions |
| `records` | entity_id FK, dataset_id FK, schema_version, payload_json, revision, created_at, updated_at, deleted_at | Validated JSON records |
| `record_revisions` | record_id, revision, payload_json, actor, timestamp, reason | Historical versions |
| `external_refs` | entity_id FK, provider, external_id, last_verified_at, availability | Basic Memory references |
| `tags` | id, normalized_slug UNIQUE, display_label | Canonical tags |
| `entity_tags` | entity_id, tag_id | Many-to-many tagging |
| `links` | id, source_entity_id, target_entity_id, relation_type, created_at, actor, deleted_at | Typed directional links |
| `idempotency` | principal_id, key, request_fingerprint, response_ref, expires_at | Deduplicate retries |
| `audit_events` | id, principal_id, action, entity_id, before_revision, after_revision, occurred_at, request_id, details_redacted | Append-only write trail |
| `entity_search` | FTS5 virtual index over title, description, tags and optional extracted record text | Local discovery |

All IDs are stable UUIDs. SQLite foreign keys ON. `UNIQUE(dataset_id, record_id)`-style membership is validated when creating links. Active dataset slugs cannot collide within a namespace. Normalize tags and slugs (lowercase ASCII slug; preserve human display name). Timestamps UTC RFC 3339. Private storage must be encrypted at the VM/volume layer if at-rest encryption is required.

## Metadata model

Every entity can have a human title, description, tags, typed custom metadata, timestamps, owner and links. Dataset descriptions should explain **purpose and expected records**, not merely repeat the name. Per-record tags allowed. Metadata is machine-readable and safely bounded in size.

## Dataset schema example

```json
{
  "namespace": "nutrition",
  "slug": "foods",
  "title": "Food catalog",
  "description": "Reusable nutritional references and serving variants",
  "tags": ["food", "nutrition"],
  "schema": {
    "type": "object",
    "properties": {
      "name": {"type": "string", "minLength": 1},
      "serving_g": {"type": "number", "exclusiveMinimum": 0},
      "calories": {"type": "number", "minimum": 0},
      "protein_g": {"type": "number", "minimum": 0}
    },
    "required": ["name"],
    "additionalProperties": false
  }
}
```

## Schema evolution

Schemas are versioned and immutable once used. New writes validate against active version. Updating a schema must declare one of:
- **Compatible**: old records still valid, no rewrite needed.
- **Migrated**: transactional explicit conversion with preview, failed row report, snapshot and rollback plan.
- **Dual read**: old rows retain schema version and are interpreted with historical schema until migrated.

No silent dropping/coercion of data. Reject invalid schema definitions and excessive nesting/size. Enforce JSON Schema Draft 2020-12 subset in v1, documenting any unsupported keywords. Unique field constraints (e.g., food canonical key) are explicit dataset settings with a separately indexed deterministic key. Do not imply SQLite JSON arrays magically provide relational foreign keys; v1 dataset-to-dataset references use entity links.

## Relationships

Typed links: `references`, `governed_by`, `derived_from`, `related_to`, `contains`, plus validated custom types. Represent each link once with `source` and `target`; queries can traverse both directions. Do not silently create reciprocal duplicate rows. Links to external memories remain if the upstream note goes missing; mark them dangling/unavailable on resolution.

## Query and indexing

- FTS5 indexes titles/descriptions/tags and explicitly selected searchable JSON fields, not entire arbitrary blobs or secrets by default.
- Exact filtering supports allowlisted paths/operators with parameterized SQLite queries; reject unknown or expensive operations.
- Cursor-based pagination with stable sort keys and max page sizes.
- No arbitrary JOIN, SQL, or unbounded table scans exposed to LLMs.
- Optional embeddings are a later separate feature; SQLite FTS5 is sufficient for first release.

## Transactions and concurrency

Use `BEGIN IMMEDIATE` for write transaction boundaries as appropriate; database busy timeout and WAL mode with documented checkpoint behavior. Revision integers must match expected values on updates or return `REVISION_CONFLICT`. Dataset schema changes and record writes serialize where required. Retriable requests use idempotency keys scoped to principal and request fingerprint. Soft delete and explicit administrative purge are different operations.

## Safety limits (provisional)

Default read page size 50, maximum 200; cap payload size at 1 MiB/record and schema nesting at an implementation-defined safe limit; require deliberate migration for widening limits. Put these settings in config with tests. Treat record content and memory text as **untrusted data**, never executable tool instructions.
