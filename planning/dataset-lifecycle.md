# Agent-managed datasets: creation, evolution and links

**Status:** design target. The engine does not know in advance that "food" is special. The model can register new datasets and define their structure, while the gateway enforces server-side constraints.

## One database, many virtual collections

The SQLite file `memdb.sqlite3` stores generic registry rows and JSON records. A logical dataset such as `nutrition.foods` is **not** a newly created SQLite file or a new arbitrary SQL table. Namespaces and datasets have own titles, descriptions, tags, schemas and linked memories. This offers "mini-databases" while keeping query safety, migrations and permissions centralized.

## Dataset creation workflow

1. **Discover existing entities:** search `discover` for similar names, descriptions and tags. Avoid duplicate `food-list`, `foods`, `meal_foods` with the same purpose.
2. **Choose a namespace:** reuse existing namespace when suitable; create a new one only with required permission.
3. **Design a schema:** JSON Schema object fields, types, required/optional fields, bounds, units and validation. Include provenance fields where appropriate. Define searchable fields deliberately.
4. **Dry-run:** planned `dataset_plan` tool evaluates schema validity, naming collisions, size limits, and proposed grants; returns warnings and a stable plan ID. This is a **proposed addition** to `mcp-contract.md`, not yet implemented.
5. **Apply:** `dataset_create` with a unique idempotency key, owner, namespace, tags, descriptor and validated schema.
6. **Link:** register or resolve governing Basic Memory notes; create typed links to the dataset and optionally individual records.
7. **Use:** `record_create`, `dataset_query`, `record_update` enforce the active schema and authorization. The model never receives arbitrary SQL execution authority.
8. **Revisit:** add metadata aliases, amend rules, version the schema or retire the dataset through explicit operations.

## Flexible first, strict later

An agent may start with a *sparse but still validated schema* (e.g. required `name` and optional JSON `details`) instead of guessing all fields in advance. Schema version 1 must still be valid and bounded; unstructured `additionalProperties: true` records should be avoided as the long-term default.

When enough records reveal a stable shape, propose a new schema and migration report. Never silently coerce unknown strings into numbers, rename a field without mapping, or drop data that fails the new schema.

## Record identity and de-duplication

Local UUID remains stable. Datasets can define one or more canonical unique keys, e.g. a food's normalized name + brand + serving unit, but these are dataset-specific and must be indexed/validated. "Same name" is not always identical: 8-inch and 16-inch sandwiches are separate variants, not automatically merged.

Repeated requests with the same principal/idempotency key and same payload return the original result; a different payload with same key returns `IDEMPOTENCY_CONFLICT`. Use expected revisions to prevent lost updates.

## Exactness, units and calculations

Dataset schema supports field `description`, `unit`, `nullable`, allowed enums, source and uncertainty qualifiers. Avoid implicit "grams or ounces" ambiguity. For nutrition: distinguish `estimate`, `label_value` and `user_supplied` provenance, plus serving size and date. Aggregate logged meals from immutable quantities/record snapshots, not simply from current catalog values that may change.

Generalized calculations may use a safe allowlist of aggregation operators: count, sum, min, max, average for numeric fields with valid units; complex domain formulas can be later explicit tool extensions. Do not offer arbitrary Python expressions or SQL execution.

## Relational datasets without bespoke tables

The engine can store `nutrition.foods` and `nutrition.meal_logs` as separate logical datasets. Records refer to each other through stable typed links (or a validated reference-field abstraction implemented on top of entity links). Query responses can include related records by bounded, permission-checked expansion. Cross-dataset references are not raw unvalidated UUID strings.

## Authorization and approvals

An agent can create a dataset only when its authenticated principal has `memdb:write` and suitable namespace grants. Schema changes and structural destructive operations require an additional `memdb:schema` or `memdb:admin` permission. User confirmations in clients may also apply; the server cannot rely solely on UI confirmations.

## Example lifecycle

- Memory note: `Food Tracking Rules` (Basic Memory)
- Dataset 1: `nutrition.foods` (name, portion, calories, protein)
- Dataset 2: `nutrition.meals` (consumed_at, quantity, record links)
- Relationship: both datasets `governed_by` memory note.
- The next model first retrieves the memory, then executes exact record reads and authorized writes.

## Failures and rollback

If schema apply fails halfway, roll back the SQLite transaction. If dataset creation succeeds but the Basic Memory external-reference linking fails, report partial success and leave an auditable, retryable reconciliation task. Do not erase the dataset silently.

## Testing goals

- Create a dataset entirely through generic tools.
- Reject invalid schema/unsupported keyword.
- Avoid accidental creation of duplicate namespaces/datasets.
- Add record and query exact numeric values.
- Reject cross-namespace forbidden links.
- Update schema while preserving historical records and a rollback path.
- Ensure records remain discoverable after service restart.

## Autonomous organization policy

The agent should *reuse before creating*. Creating a new logical dataset is R2 and may execute without repeated prompts **only** when covered by a user-configured scoped delegation or trusted review; schema application, mass rewrites and privileged governing links are R3. Dataset creation and link operations are evaluated server-side, not by trusting a model-supplied user confirmation. See [organization workflow](autonomy/dataset-organization.md), [risk matrix](autonomy/policy.md) and [approval process](autonomy/delegation-and-approvals.md).
