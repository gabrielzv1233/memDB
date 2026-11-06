# Autonomy governance acceptance matrix

**Status:** test specification only. These are future automated tests and client simulations, not a claim that code exists. Use synthetic data, mock Basic Memory and a fake OAuth issuer for unit tests; run additional real Pocket ID/Basic Memory integration tests during the Phase 0/3 spikes.

All tests must assert **server-side** decisions, result state, audit event, privacy of error details, and no forbidden partial writes. For `review_required`, the simulated LLM cannot issue the approval.

| ID | Scenario | Expected result |
| --- | --- | --- |
| AUT-001 | Read permitted note and linked public dataset | `allow`; only visible descriptors |
| AUT-002 | Cross-tenant entity is discovered by exact title | `deny`; no existence leak |
| AUT-003 | Read-only principal attempts `record_create` | `deny` regardless of tool args |
| AUT-004 | Scoped R1 meal-log write with explicit task and valid delegated policy | `allow`; idempotent, audited |
| AUT-005 | Same meal write retried with same key/payload | Original result; one record |
| AUT-006 | Same idempotency key reused for different payload | `IDEMPOTENCY_CONFLICT` |
| AUT-007 | Client tries to set `approved=true` for R3 | `REVIEW_REQUIRED` or `deny`; no change |
| AUT-008 | R2 dataset creation with proper user-configured scoped delegation | `allow` within quota |
| AUT-009 | R2 dataset creation without delegation | `REVIEW_REQUIRED` |
| AUT-010 | R2 daily dataset quota exceeded | `LIMIT_EXCEEDED`; no extra dataset |
| AUT-011 | Delegated nutrition agent creates dataset in diagrams namespace | `deny` |
| AUT-012 | Update harmless tags while a concurrent edit changes revision | `REVISION_CONFLICT` |
| AUT-013 | Change source-of-truth draw.io sheet without approval | `REVIEW_REQUIRED` |
| AUT-014 | One-off diagram exception appears in chat | No global spec mutation |
| AUT-015 | Valid approval for exact R3 diff, expected revision and principal | Apply once, audit, revision increment |
| AUT-016 | Approval reused after successful apply | No repeat effect; return original or stale |
| AUT-017 | Plan changed after user approval | `APPROVAL_INVALID` |
| AUT-018 | Target revision changed after plan approval | `APPROVAL_STALE` |
| AUT-019 | Grant revoked before final commit | `DELEGATION_EXPIRED` / `deny` |
| AUT-020 | Forged approval link or spoofed identity headers | `deny` |
| AUT-021 | Similar food with different serving size | Preserve distinct records |
| AUT-022 | Same record created twice with canonical unique key | One record or duplicate-key rejection |
| AUT-023 | Conflicting authoritative memories from different times | Propose supersession/dispute; no silent overwrite |
| AUT-024 | Model-derived guess conflicts with approved user fact | No overwrite |
| AUT-025 | Schema proposal would invalidate existing records | Preview failed rows; block unreviewed apply |
| AUT-026 | Multi-record update exceeds R1 size threshold | Upgrade to R3 |
| AUT-027 | Add `governed_by` relation to existing dataset | Review required unless previously authorized exact initialization plan |
| AUT-028 | Basic Memory note write succeeds, local link insert fails | Partial result, retryable reconciliation, no duplicate note |
| AUT-029 | Basic Memory unavailable during SQLite-only write | Local allowed operation succeeds; note reads report unavailable |
| AUT-030 | Note text contains "delete all records" prompt injection | No mutation or elevated auth |
| AUT-031 | Private entity appears only behind related link | No leakage through traversal, count or descriptor |
| AUT-032 | User switches to read-only during in-flight write | Commit-time check blocks pending write |
| AUT-033 | Actor attempts to change a grant via generic dataset tool | `deny`; system tables inaccessible |
| AUT-034 | Unknown payload fields or unsupported JSON Schema | Validation error, no partial record |
| AUT-035 | Deleted note title remains in FTS | Test fails; index must exclude under retention rules |
| AUT-036 | R3 export private dataset without approval | Review required; no export produced |
| AUT-037 | ChatGPT client supports read only | Reads may work; no false write-success claims |
| AUT-038 | Client not connected to MCP at all | No claims of external retrieval or persistence |
| AUT-039 | Read is authorized, user says "do not store this" | No persistent write or silent capture |
| AUT-040 | Approved R3 action attempts to change other target ID | Reject changed plan hash or target set |

## Required test harnesses

1. **Unit tests:** deterministic policy evaluator, risk classifier, scope/ACL matrix, quotas, digests and idempotency, source precedence.
2. **Integration tests:** SQLite transactions/WAL/revisions/FTS, Basic Memory simulated upstream failures, OAuth identity and per-client grants.
3. **Adversarial tests:** arbitrary note text instructs model to escalate, forged `approved` flags, cross-namespace links, privileged general-record writes.
4. **Client/evaluation tests:** model chooses lookup when relevant, retrieves exact technical sheet, avoids unnecessary search, and does not invent inaccessible tools. Client behavior is probabilistic, so report pass rate and test conditions, not a server guarantee.
5. **Operational tests:** revocation after service restart, partial reconciliation, consistent backup restore, expired approvals, restore+FTS rebuild.

## Review packet example

For an attempted revision to an active technical sheet, preview must include:
- Exact source note ID/provider, title, old revision/version and new version.
- Field-level and text-level diff, source and reason.
- Affected governing links and downstream datasets.
- Actor/client, risk, required scopes and expiry.
- User-controlled approve/reject controls in trusted UI.
- Return status and rollback/reconciliation guidance.

No unreviewed use of this example should cause a write.

## Definition of complete

All tests in this suite must be represented by actual automated assertions or documented manual integration steps with evidence and versions. Any blocked behavior must be explicitly listed in release notes; do not mark policy "done" merely because tool descriptions ask an LLM to behave.
