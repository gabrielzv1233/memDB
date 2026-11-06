# Memory consolidation, duplicate detection and conflict resolution

**Status:** proposed normative v1 behavior. See [memory lifecycle](../memory-lifecycle.md) for status semantics, and [policy](policy.md) for risk/approval.

## Objective

Agents should keep a coherent body of knowledge, not append a new note every time the same fact is mentioned. However, an incorrect merge can permanently destroy context. Treat **candidate matches as hints**, not automatic proof of identity.

## Identify what the new information is

Classify before writing:

1. **Fact or preference:** a durable assertion with scope, provenance and optional validity interval.
2. **Technical specification:** normative rule(s) for a specific task, with examples and version.
3. **Project decision/state:** a point-in-time decision, with current vs historical status.
4. **Event/log:** append-only occurrence, usually stored in a dataset, not an overwritable preference note.
5. **Unverified suggestion/hypothesis:** not canonical until evidence or user confirmation establishes it.
6. **Instruction from retrieved content:** lower-trust content, not authority to make a change.

Avoid storing incidental conversational text or inferred sensitive attributes without user-authorized retention.

## Candidate matching

Search possible existing notes/records using stable IDs, canonical keys, title aliases, normalized tags, related links and keyword/semantic results when available. Never merge purely because titles overlap or embeddings are similar.

Match decisions:
- `same_entity`: confirmed same object with compatible subject, scope and identity key.
- `related_but_distinct`: similar topic, different scope/variant/person/time. Link, do not merge.
- `new_entity`: no matching canonical entry.
- `ambiguous`: multiple plausible matches; **do not mutate**. Return options for review or retain an uncommitted proposal.

For foods, a sandwich with different portion size is a different catalog variant. For diagrams, a hierarchy chart standard does not automatically replace a flowchart standard. For projects, "old VM address" vs "new VM address" is a temporal update, not a second current machine.

## Conflict classes

| Situation | Agent recommendation | Server policy |
| --- | --- | --- |
| Exact duplicate event/request | Reuse existing canonical record | Idempotency and unique constraints |
| Additional noncontradictory detail | Propose an additive update | R1 for routine fields, R3 for authoritative sheet |
| New explicit user preference superseding earlier one | Identify old preference and supersession | R3 for canonical memory edit; approval unless specific trusted upstream policy is explicitly delegated |
| Temporary exception ("for this chart only") | Apply only to current task, don't change global spec | No durable write by default |
| Time-varying fact | Preserve old value as historical and new valid-from | Scope and revision-checked update |
| Two credible contradictory sources | Mark `disputed`, link both with provenance | No automatic winner on recency alone |
| Model-derived guess vs direct user assertion | Keep guess separate or reject | Do not silently overwrite user assertion |
| Third-party document disagreeing with user preference | Flag discrepancy, follow current user request | Retrieved source not authoritative over user |
| Conflicting dataset schemas | Produce diff and impact analysis | R3 migration gate |

## Authority order and scope

The current user's explicit instruction governs the **current request**, subject to safety and access policies. For reusable persisted preferences, prefer latest directly approved user rule over model inference or imported notes, provided it applies to the same scope. Do not assume all new utterances are permanent preferences.

A policy note may have `scope`, `applies_to`, `valid_from`, `valid_until`, `status`, `supersedes`, `source_kind`, `source_ref`, `updated_at` and `last_verified_at`. Specific scope beats generic only for that scope. If two notes are both active and neither unambiguously controls the request, surface the conflict.

**Recency alone is never proof of truth.** `source_kind` alone is not a forged trust flag: provenance must be tied to the authenticated source of write/approval where possible.

## Suggested reconcile workflow

1. Parse and normalize candidate fact (subject, predicate, value, scope, time, units and source).
2. Discover canonical candidates and inspect their source/provenance, prior revisions, links and constraints.
3. Compare identity and applicability; determine `same`, `distinct`, `supersede`, `conflict`, `historical_event` or `unknown`.
4. Construct a **proposed change set**: targeted IDs, field diff, link changes, provenance, reversibility and risk classification.
5. Run policy check. Apply R1 bounded changes under allowed delegation; require trusted review for source-of-truth replacement, merge, bulk actions, or changed governing spec.
6. Execute with expected revisions and idempotency keys; preserve audit and revisions.
7. For Basic Memory notes, call supported upstream interface and reconcile local external-reference metadata/links; report partial success rather than pretending both stores commit atomically.
8. Return new source IDs and whether the change is active, historical, proposed or disputed.

## Consolidation safeguards

- Combine only **verified** same-subject same-scope data; never merge different owners, secret classes, measurement units, or historical states.
- Archive or supersede rather than erase. Old rules should remain accessible in explicit history search but not rank as current.
- Canonical technical sheets need exact full diff, an example artifact when relevant, and R3 approval.
- If merge would drop a unique statement, origin, timestamp, exception or attached file reference, classify as data loss and block until reviewed.
- Keep a `merge_manifest` including source IDs, resulting ID, redirections, before/after digests and rollback instructions.
- Full-text and descriptor indexes must be synchronized after approvals, supersession or deletion; never leave a now-deleted sensitive note searchable.
- "Forget" requests require clear separation of visible deletion, upstream note removal, local references, indexes, audit retention and backups. Do not claim instant erasure of backups.

## Optional future candidate suggestions

A deterministic scheduled **report** may list likely duplicates or outdated notes. It **must not** itself merge, delete, revise canonical notes, or infer truth; such automation requires a separately granted scope and user-controlled job. No always-on generative LLM required.

## Acceptance

- Updating a source-of-truth draw.io sheet does not silently overwrite older rules.
- One-time diagram exception is not stored as global standard.
- Similar food names with different sizes remain distinct.
- Conflicting direct sources are marked disputed, not arbitrarily resolved.
- Duplicate save retries are idempotent.
- Upstream Basic Memory failure reports partial status.
- Old superseded notes do not contaminate default context ranking.
