# Autonomous dataset organization and schema management

**Status:** proposed agent behavior and server rules. The model may *propose* new data structures; server-side policy determines whether changes are authorized.

## Reuse before creation

For every structured write, first run targeted discovery of candidate namespaces/datasets. Inspect schema, description, tags, aliases, canonical keys, governing notes, record sample shape if permitted, ownership and visibility.

Score candidates in this order:
1. Exact canonical dataset ID / slug in the user request.
2. Compatible structure and purpose, including existing record relationships and governing note.
3. Correct principal/namespace/visibility boundary.
4. Human description, tags, synonyms and title.
5. Data retention and authority requirements.

A dataset for nutritional *reference foods* is distinct from an *append-only meal history*, even if both carry `nutrition` tags. Prefer linking two appropriately focused datasets to one policy note over putting everything into a single amorphous table.

## Decision tree

- **Existing compatible dataset:** reuse, do not create. Add aliases/tags when a genuine matching search synonym is missing.
- **Existing nearly compatible schema:** propose a *compatible extension* and show impact; do not silently add fields or overload unrelated columns.
- **Existing dataset but wrong privacy/owner boundary:** never write into it or link across denied ACLs. Select/create correctly scoped dataset or return permission error.
- **Different identity/semantics:** create a separate dataset if the intended task and grants permit it; link as related where appropriate.
- **Ambiguous naming or duplicate candidates:** return discovery options and stop before mutation.

## Agent-defined dataset plan

A plan should include:
- `namespace_id`, `slug`, `title`, `description`, `tags`, `triggers`, `owner` and `classification`.
- Required/optional fields, JSON Schema version, units, allowable enum values, canonical identity fields and planned search indexes.
- Expected read/write volume and retention needs (if known).
- Intended linked governing/related memories and relation types.
- Data provenance policy and whether records are events or mutable reference records.
- Collision/compatibility check results, privacy impact, and policy risk.
- Permission and quota outcome, estimated write count and expected revisions.

`dataset_plan` does not perform SQL DDL or write any database rows. `dataset_create` accepts a plan reference or validated payload; server repeats all checks and enforces policy at commit.

### Starting with a small schema

An AI need not predict every future field. Start with the smallest valid schema that represents the actual expected entries. Allow later versioned extensions rather than accepting unbounded arbitrary nested blobs or creating hundreds of one-off datasets. A draft schema may use optional `details`, but its type/size must be bounded.

### Metadata hygiene

Generate **specific human descriptions**, tags and trigger words to aid model discovery. Bad: "stores stuff." Good: "Reusable restaurant food items with portions and nutritional values; queried before creating meal log entries." Avoid overly broad tags such as `all` or `important` that dilute relevance. Metadata edits to a non-authoritative dataset can be R1; changing purpose/ownership or governing rules is R3.

## Autonomy boundary for structural changes

- Creating a dataset is **R2**, not categorically forbidden. An authenticated user with a matching scoped delegation can let an AI create new logical datasets without a separate click on each one, within quota.
- Without delegation, the AI can still prepare the plan and request approval. A model-supplied "the user asked" flag is not an approval credential.
- A new source-of-truth policy relationship `governed_by` is normally R3. For *initial setup*, a trusted user-approved/delegated combined creation plan can include it if exact target and permission bounds were displayed/authorized.
- Modifying active schema version, moving many records, mass rename, dropping fields or changing private classification is R3. **Propose, preview, approve, apply**, then verify.
- Autonomous creation must not implicitly ingest every conversation or start auto-populating records.

## Dataset lifecycle

`proposed` → `active` → `deprecated` → `archived` → optional explicit deletion.

Schema versions: immutable. Old records retain their schema version or are explicitly migrated. Historical event entries are not overwritten by later source catalog updates. Dataset/entity ownership and access are independent of physical database file placement.

## Query hygiene and duplicate prevention

- Use stable record UUIDs; optional canonical keys must be validated server-side.
- Require explicit units and provenance for numeric fields.
- Search existing records before adding a likely duplicate, using canonical key and normalized fields.
- Agent recommendations may include **suggested** similarity matches, but server unique constraints/idempotency decide duplicate writes.
- Never let a model propose raw unrestricted SQL, Python expressions, arbitrary file paths or network URLs as query filters.
- Respect per-task write caps, rate limits and bounded cursor pagination.

## Worked example: linking a food reference and meal log

1. User has authorized food logging. Agent discovers a Basic Memory note on default food preferences and existing `nutrition.foods`.
2. Agent finds `nutrition.meals` with append-only behavior, or plans it if absent.
3. A new `nutrition.meals` can be created through R2 delegated policy; without it, the engine returns a review plan.
4. Explicit "log lunch" becomes one R1 event write, with timestamp, portion and source/uncertainty; do **not** turn it into a durable global preference.
5. The meal event links to the selected food record. The catalog record is reused, not copied into a new `food_of_the_day` dataset.
6. Revising a calorie estimate later must not silently recalculate prior logged meals; retain the original snapshot and provenance.

## Test goals

- Reuse the existing catalog instead of creating a duplicate.
- Create a genuinely new logical dataset under scoped delegation.
- Reject dataset creation in a denied namespace.
- Enforce separate privacy boundaries and non-leaking discovery.
- Propose schema evolution with a failed-row impact report.
- Prevent mass rewrite without reviewed approval.
- Validate clear tags/description and schema fields.
- Keep event history immutable by default.
