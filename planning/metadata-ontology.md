# Unified metadata, tags, identity and relationship semantics

**Status:** proposed. The metadata model is shared conceptually by Basic Memory notes and memDB entities. The canonical stores remain distinct.

## Common descriptor

Every search result and `entity_describe` response should use these fields where available:

| Field | Meaning | Rules |
| --- | --- | --- |
| `id` | Stable local UUID | Always local to memDB; cannot be reassigned |
| `kind` | `namespace`, `dataset`, `record`, `memory`, `asset` | Explicit enum |
| `title` | Human-readable name | Nonempty, bounded length |
| `description` | Short statement of what this holds and when to use it | Searchable; not secretly executable instructions |
| `slug` | Stable machine-oriented lookup key | Unique in applicable namespace |
| `namespace` | Logical group like `nutrition`, `diagrams`, `projects` | Does not imply database isolation |
| `tags` | Search/discovery labels | Canonical normalized label IDs; alias handling |
| `status` | `draft`, `active`, `disputed`, `superseded`, `archived` | Affects default discovery |
| `created_at`, `updated_at` | System timestamps | UTC, server-controlled |
| `valid_from`, `valid_until` | When a claim/rule is applicable | Optional, not equivalent to record creation |
| `source` | Where information came from | Provider, ID, actor/provenance reference |
| `scope` | Contexts where a memory applies | Optional versioned metadata |
| `triggers` | Keywords/synonyms to help *find* it | Hints only, not proof or mandatory invocation |
| `related_count` | Number of visible related entities | ACL-filtered only |
| `revision` | Optimistic concurrency revision | Monotonic for locally owned entities |
| `owner` / `acl` | Access grants | Enforced server side, not trusts model text |

For external Basic Memory notes, the local metadata is a **cache/projection**, marked with `last_verified_at`, `provider`, `external_id`, and `is_stale`. Canonical author/editor is Basic Memory. Do not silently overwrite upstream note YAML based on local cache changes.

## Example technical sheet frontmatter

```yaml
---
type: technical_spec
title: Draw.io Diagram Standards
status: active
version: 1
tags: [drawio, diagrams, flowchart, ipo, hierarchy]
triggers: [draw.io, drawio, flow chart, diagram, IPO]
applies_to: [diagram-generation]
description: Exact editable diagram standards and reference diagrams
---
```

This is **illustrative**: field support and schema enforcement in Basic Memory must be checked against the deployed version.

## Metadata for a dataset

```json
{
  "kind": "dataset",
  "namespace": "nutrition",
  "slug": "foods",
  "title": "Food Catalog",
  "description": "Reference foods and serving variants used by meal logs",
  "tags": ["food", "nutrition", "reference"],
  "triggers": ["log food", "calories", "protein", "meal"],
  "status": "active",
  "schema_version": 1,
  "searchable_fields": ["name", "restaurant"],
  "governed_by": ["memory-reference-uuid"]
}
```

## Tags versus relationships versus scope

- **Tags** support broad classification: `nutrition`, `drawio`, `coding`.
- **Descriptions** explain meaning and intended use, especially important for model search.
- **Scope** indicates applicability: `diagram-generation`, `this-project`, `home-network`.
- **Relationships** establish explicit structure or authority: `governed_by`, `derived_from`, `exemplar_of`.
- **Schema** defines record shape and validation. It is **not** a replacement for metadata.
- **Namespaces** provide organizational boundaries. A namespace's ACL must be checked even if a link originates elsewhere.

Tags should not be treated as a user permission system, and a cross-namespace link must not override ACLs.

## Stable external identities

Prefer `(provider, upstream_stable_id)` as an immutable unique key for Basic Memory references. Slugs, file paths, titles and permalinks are *lookup aliases*, not necessarily stable identities. If upstream stability cannot be verified, document fallback and collisions during Phase 0, do not pretend rename-safe linking is guaranteed.

## Proposed constraints

Descriptions limited to 1,000 characters by default, title 200, tags 32 per entity, triggers 32, and one-hop descriptor expansion limited to 20 visible links. Final limits must be checked against realistic data. Reject untrusted metadata types and excessively nested custom metadata.

## Privacy/retention

Minimize sensitive metadata in result previews: even a title or tag can be private. Do not return hidden entity counts, hidden relationship endpoints, or exact-match hints to unauthorized callers. A deleted upstream note must eventually vanish from local FTS, its title cache and searchable tags according to the configured retention policy.
