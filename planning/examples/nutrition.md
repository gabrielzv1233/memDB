# Example: an agent creates food datasets from a memory

This is a **hypothetical design example**, not imported user data and not authoritative nutritional information.

1. Agent discovers a Basic Memory note called "Food tracking conventions" tagged `nutrition`, `instructions`.
2. Agent reads it to learn how to handle serving units, uncertainty, timestamps and existing foods.
3. Agent calls `dataset_create` for `nutrition.foods` with schema `name, serving_g, calories, protein_g`.
4. Agent creates `nutrition.meals` with fields `eaten_at, quantity, food_ref` (record-reference validation provided by engine policy).
5. Agent registers or resolves the note reference, then links it to both datasets with `governed_by`.
6. Agent adds known foods to the catalog, linking meal entries to actual food record IDs. Estimated values must be marked as estimates.
7. Later: `discover("log lunch")` returns the conventions note and dataset descriptors. The AI reads relevant rules and adds a meal with an idempotency key.

### Metadata example

```json
{
  "title": "Food catalog",
  "description": "Reusable nutritional reference entries used for meal logging",
  "tags": ["nutrition", "reference", "foods"],
  "links": [
    {"relation_type": "governed_by", "target": "basic_memory:<external_note_id>"}
  ]
}
```

### Important boundaries

The AI creates datasets only through authorized operations. The storage engine does not guess nutrition facts, decide meal timing, or call itself. Updates to schema must be validated; existing meals must not silently change when catalog values are edited. Audit/provenance tracks who wrote records and whether a nutrition value was estimated.
