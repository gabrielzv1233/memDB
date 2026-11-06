# Autonomous management policy suite

**Status:** v1 design baseline, not implemented. **Owner:** memDB architecture. **Date:** 2026-11-05.

These documents govern what an AI client **may request**, what the memDB gateway **must authorize**, and which operations must remain user-controlled. They are normative design requirements for implementation; the service does not autonomously observe ChatGPT conversations or run a background generative LLM.

## Reading order

1. [Policy and risk levels](policy.md): default behavior, capability boundaries, action decisions.
2. [Delegations and approvals](delegation-and-approvals.md): safe ongoing autonomy without asking on every small write.
3. [Memory reconciliation](reconciliation.md): dedupe, contradictions, supersession, evidence and rollback.
4. [Dataset organization](dataset-organization.md): deciding when to reuse vs create, schema evolution and exact records.
5. [Governance test matrix](test-cases.md): cases that agents and backend must pass.
6. [Autonomy workflow](../diagrams/autonomy-policy.drawio): editable decision-flow diagram.

Related specifications: [MCP contract](../mcp-contract.md), [metadata ontology](../metadata-ontology.md), [memory lifecycle](../memory-lifecycle.md), [security](../security-pocketid.md), [acceptance scenarios](../acceptance-scenarios.md).

## Terms

- **User request:** the current human instruction to the model. This is *intent context*, not cryptographic proof to the gateway.
- **Principal:** authenticated user/service from verified OAuth identity. Authorization is checked independently.
- **Agent:** an LLM or program using the public MCP tools on a principal's behalf.
- **Delegation:** an explicit, revocable server-managed grant to perform a bounded category of actions without case-by-case approval.
- **Policy decision:** server result `allow`, `deny`, or `review_required`, with reasons and constraints.
- **Approval:** fresh evidence issued by a trusted approval route for one exact reviewed operation.
- **Plan:** a bounded, inspectable proposal including actor, target, intended changes, risks, and expected revisions.
- **Autonomous:** an allowed tool operation chosen and invoked by a client within an existing human-authorized task or previously configured delegation. Does **not** mean surveillance or unrequested background actions.

## Core requirements

1. Default-deny actions not covered by a valid scope and an entity grant, even when an LLM is convinced it is appropriate.
2. Low-risk actions need not create a confirmation prompt when a user previously delegated them.
3. No silent durable retention of ordinary chat content without a user-controlled retention policy.
4. Technical standards and source-of-truth memories have higher protection than search tags and record descriptions.
5. No agent can approve its own privileged action. Approval must be tied to the **exact effect**, not a reusable "yes" token.
6. Changes must be attributable, idempotent where possible, and revision-checked. Give users a concrete rollback/recovery path.
7. Local SQLite transactional changes and Basic Memory note changes are **not atomic together**. Report and reconcile partial completion.
8. Preserve data minimization and privacy by default, including in previews, logs and reminders.
9. These are **server-enforced gates**, not instructions that can be bypassed with clever prompt wording.
10. Keep normal ChatGPT chat usable: text technical sheets + MCP retrieval, no dependency on Agent Skills.

## Default autonomy profile

The proposed default is **assisted**, with optional **delegated** mode per namespace and operation category. `read_only` is available. A user can revoke delegation without disabling permitted reads. Disallowed or unapproved actions are never converted into partial destructive operations.

**Product decision needing real-account validation:** the gateway may support writes that a given ChatGPT plan/client does not permit. In that case, clients can still read where supported, and writes require another authorized client. Do not claim ChatGPT automatically invokes memDB.
