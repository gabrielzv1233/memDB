# Delegations, approvals and revocation

**Status:** proposed server-enforced contract. Design goal: let an AI manage benign organization without asking the user about every small change, while protecting expensive, private and irreversible changes.

## Three configurable user profiles

| Profile | Behavior |
| --- | --- |
| `read_only` | Search/read only, regardless of available write tools |
| `assisted` (default) | R1 writes during authorized user-directed workflows; R2/R3 require applicable delegation or trusted review |
| `delegated` | Additional R1/R2 operations within explicit user-managed grants and hard limits; R3 still requires reviewed approval |

The gateway must treat OAuth scopes, namespace/entity ACLs, user-specific delegation, limits and operation risk as **cumulative gates**. A broad OAuth scope alone never grants automatic high-risk approval. Deny by default when the grant is absent, stale, expired or ambiguous.

## Delegation format (illustrative)

```json
{
  "id": "grant-uuid",
  "principal": "issuer-subject",
  "client_id": "authorized-client-id",
  "mode": "delegated",
  "allowed_operations": [
    "record.create",
    "record.update",
    "entity.tags.update",
    "dataset.create"
  ],
  "namespace_ids": ["namespace-uuid"],
  "max_risk": "R2",
  "max_new_datasets_per_day": 3,
  "max_records_per_task": 10,
  "expires_at": "2026-12-05T00:00:00Z",
  "revoked_at": null
}
```

Bind delegation to stable verified principal, OAuth client identity (where provided), namespace, operation family, expiry, quotas and policy version. Grant creation/change/revocation is a trusted user action outside model-invocable ordinary MCP tools. Absence of reliable client identity must reduce, not widen, access.

Do not store bearer credentials in grant rows or expose them in logs. Grant records and their revocations are server-owned, auditable and persist through restart.

## Trusted approval protocol

**Problem:** a model parameter like `approved=true` is trivially forgeable. A trusted approval must be issued by an authorized server-controlled approval UI or another authenticated out-of-band channel that independently displays the exact plan to the user.

1. Agent calls a **propose/dry-run** tool for an R2/R3 operation. The gateway computes the exact affected IDs, field-level before/after diff, estimated record count, data classification, current revisions and risk.
2. Gateway returns `REVIEW_REQUIRED`, a random review ID and a short summary, not a capability to apply immediately. The review ID is only a locator.
3. Authenticated user opens a trusted approval screen on the gateway, sees the full normalized diff, grants an *allow-once* confirmation or rejects it. No model-supplied text alone can count as review evidence.
4. Gateway mints an internal, single-use approval proof bound to review ID, exact canonical plan hash, principal, authorized client identity, target set, policy version, expected revisions, issued/expiry times and approval user identity. Store proof server-side; the agent only references `review_id` after approval.
5. On `apply`, server independently retrieves the proof and verifies all properties plus current scopes/grants/ACL. It atomically consumes proof with the local SQLite write in one transaction. If Basic Memory upstream is involved, stage/record the external side effect and reconcile explicitly.
6. Any changed plan/diff, stale revision, expired grant, different caller/client, exhausted proof or denied review fails. User can revoke pending reviews.

A server-generated review link may be exposed to the model *only* as a short-lived pointer to a safe application URL. The model may tell the user that approval is required, but cannot click its own approval UI and authenticate as the user. Never send a privileged confirmation URL with embedded reusable bearer token in a tool response.

## Policy evaluation and lifecycle

Proposed local tables (separate from application data schemas):
- `autonomy_profiles`: principal, mode, policy_version, updated_at
- `delegations`: principal_id, client_id, namespace, allowed_ops, max_risk, expires_at, revoked_at, quota_json, revision
- `review_plans`: id, principal, normalized_operation_json, digest, target_revisions_json, risk, expires_at, status
- `review_decisions`: review_id, actor, decision, time, auth_context_ref, plan_digest
- `policy_events`: actor/client, decision, reason_code, operation, target_ref_redacted, time, correlation_id, policy_version

These are not executable migrations. Enforce unique identities, foreign keys, audit retention policy and idempotent state transitions.

Review states: `proposed` → `approved`/`rejected`/`expired`/`cancelled`; `approved` → `applying` → `applied` or `failed`. Replay of an already applied review must return the original result for the identical idempotent request, not run it twice. Do not assert global atomicity across Basic Memory and SQLite.

## Revocation and user controls

Owners can list/revoke grants, switch to `read_only`, inspect pending approvals and recent autonomous actions, and undo a supported revision. Revocation takes effect at the next server authorization check, including before commits of in-flight operations. It does not automatically reverse earlier writes. `autonomy_profiles` and `delegations` are never writable from general dataset tools.

## Failure and privacy

If the approval UI cannot be reached or the client does not support a way to present it, report `REVIEW_REQUIRED` with a safe review reference; do not silently proceed or claim user approved. On a read-only ChatGPT connection, generating a plan does not grant write capability. Review previews must be permission-filtered; approval messages may not expose secrets via notification banners.

## Tests

- Model-submitted `approved=true` is rejected.
- User approves one exact diff; different diff using same review ID fails.
- Approval is single-use and expires/rejects correctly.
- Delegation for nutrition cannot authorize other namespaces.
- Revoked grant blocks pending commits.
- A client change cannot reuse a different client's grant.
- Valid OAuth token without correct entity grants is denied.
- In-flight revision change after approval returns stale-plan error.
