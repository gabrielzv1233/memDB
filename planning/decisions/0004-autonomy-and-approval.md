# ADR 0004: Bounded autonomous organization with trusted approval

**Status:** Proposed baseline  
**Date:** 2026-11-05

## Context

The goal is to let an AI manage tagged memories and dynamically created SQLite datasets naturally during normal chat. Requiring approval for every harmless tag or requested food-log write would be cumbersome. Giving an AI permanent arbitrary write authority would risk destructive merges, prompt-injection-based edits, accidental deletion and unauthorized private-data exposure. ChatGPT tool availability and capabilities vary by client/plan.

## Decision

Adopt server-enforced operation levels R0 through R4, per-user profiles and scoped expiring delegations. R1 routine edits can run under an appropriate user task and delegated permissions. R2 structural creation can run without repeated prompts under explicit per-namespace delegation and quotas. R3 authoritative, bulk and destructive operations require a trusted server-verified exact-plan review. R4 raw arbitrary SQL, role escalation, hidden conversation ingestion and forbidden cross-tenant access are rejected.

Agents may propose actions but cannot mint approval evidence. Every mutation is validated against OAuth identity, client identity where available, entity ACL, current revision, policy/delegation and idempotency constraints. User can revoke delegation. All meaningful writes are audited and reversible where technically feasible. Basic Memory notes stay upstream; cross-service partial failure is reconciled honestly.

## Alternatives rejected

1. Ask before every write: too slow and unpleasant for ordinary logs and metadata changes.
2. Grant models blanket SQLite write / shell access: difficult to secure and audit.
3. Trust model-supplied `approved=true`: forgeable and not independent approval.
4. Always run an auxiliary LLM to monitor chat: impossible for opaque ChatGPT normal chat and unnecessary.
5. Auto-merge contradictory notes by semantic similarity: easily loses provenance and specific exceptions.

## Consequences

Implementing the policy engine, trusted review UI, revocation and tests is extra work. A client unable to create datasets/write in normal ChatGPT cannot bypass that restriction; the same backend can still work from other authorized clients. Default autonomy is assisted, with optional delegation configured through trusted controls. No guarantee all write proposals can be completed directly inside ChatGPT.

## References

- [Policy levels](../autonomy/policy.md)
- [Delegation and review](../autonomy/delegation-and-approvals.md)
- [Evaluator](../autonomy/policy-evaluator.md)
- [Conflict resolution](../autonomy/reconciliation.md)
- [Dataset organization](../autonomy/dataset-organization.md)
- [Governance tests](../autonomy/test-cases.md)
