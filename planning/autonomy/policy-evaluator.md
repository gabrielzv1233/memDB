# Policy evaluation contract and deterministic decision order

**Status:** detailed v1 implementation specification, no runtime code committed.

## Inputs and identity

The server must obtain identity from verified transport credentials, not from model arguments:

- `principal_id`, `client_id` when trustworthy, effective OAuth scopes.
- Entity owner, inherited namespace ACLs, visibility/classification, target revision.
- Operation type plus **actual** computed diff, target set, record count, sensitive fields and relation semantics.
- Stored autonomy profile, delegation records, quotas, revocations and approvals.
- Idempotency key, request ID and transaction context.

`reason` and `user_intent_text` supplied by the agent can be logged for explanation but are **not** trusted authorization evidence.

## Mandatory evaluation order

1. **Authenticate** transport request, validate issuer/audience/expiry and client binding as available. Missing/invalid -> `UNAUTHENTICATED`.
2. **Normalize** operation, bounded JSON payload, UUIDs, namespace and declared target IDs; reject invalid fields and raw execution syntax.
3. **Load targets with permissions checks** and calculate the exact effect. Avoid information disclosure in missing-vs-hidden errors.
4. **Assess data classification** and risk from effects, including affected record count, source-of-truth status, `governed_by` links, schema version and cross-namespace access. Use highest applicable risk.
5. **Check scopes and ACL grants**. Missing privilege -> `FORBIDDEN`, regardless of prior delegation or review.
6. **Check denylist and profile.** R4 always deny. `read_only` forbids writes. Enforce explicit user's non-retention setting where known to the server.
7. **Check delegations or reviewed approval** for the effective risk tier and target exact set; reject missing/stale/expired/revoked limits.
8. **Check quotas and idempotency**, then optimistic revisions. A repeated identical idempotent request may return prior response without a second side effect.
9. **Execute local transaction** with commit-time recheck of revisions, revocation and active policy. Persist audit and consumed approval atomically with local change.
10. **Handle upstream effects** via a staged outbox/reconciliation strategy (not an imaginary distributed transaction); report partial success and upstream provenance explicitly.
11. **Return structured outcome**, enough for the model to know whether the data actually changed.

Unknown risk is never implicitly R1; fail to `REVIEW_REQUIRED` or `deny` until mapped.

## Proposed decision envelope

```json
{
  "ok": false,
  "error": {
    "code": "REVIEW_REQUIRED",
    "message": "Changing a governing technical specification requires user approval.",
    "details": {
      "review_id": "review-uuid",
      "risk": "R3",
      "expires_at": "2026-11-05T21:00:00Z",
      "summary": "Proposed change to one canonical specification"
    }
  },
  "meta": {
    "request_id": "request-uuid",
    "policy_version": "v1"
  }
}
```

Protected entity IDs and previews should be omitted from public errors unless the caller is independently authorized to see them. Error code names are defined in the MCP contract.

## Explicitly separate intent, permission and approval

- `intent`: what the user appears to be asking the LLM to do. The gateway may not be able to verify the underlying chat message.
- `permission`: which user/client scopes and ACL grants the server can verify.
- `delegation`: what the verified user previously allowed this actor/client to automate.
- `approval`: a trusted decision for the exact proposal, independently obtained and verified.
- `execution`: actual write, subject to all above checks.

A tool request containing a user quote is still **client-supplied text**. Do not treat it as a substitute for server-bound delegation or trusted approval.

## Multi-operation plans

Operations can depend on one another (create dataset, register note reference, link two entities). A plan must list each sub-operation, dependencies, target owner/namespace, highest risk and exact expected effects. The gateway must either:
- apply purely SQLite steps atomically in one transaction; or
- stage external Basic Memory calls via durable outbox, with explicit completion/partial status.

No split strategy may allow privileged sub-operations after a denial. On retries, idempotency keys and plan IDs must prevent duplicate datasets and references.

## Effective risk precedence

R4 prohibited > R3 destructive/authoritative/bulk > R2 structural > R1 routine > R0 read. A long record update batch cannot be broken into ten R1 calls merely to evade R3 review; track cumulative per-task/rolling-window counts.

## Implementation-facing checks

- Replay-safe `review_id` and idempotency state machine with database uniqueness/locking.
- Normalized plans produce stable content digests using canonical serialization (including schema, targets and revisions).
- Delegations/approvals bound to authenticated principal and client, not usernames supplied by the model.
- Database-backed monotonic policy/ACL revision invalidates stale reviewed plans.
- Reviewer identity recorded independently of the agent identity.
- Exposed fields never include secrets, approval proof materials or hidden metadata.
