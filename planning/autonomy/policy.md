# Autonomous operation rules and risk classification

**Status:** proposed normative v1 contract. Applies equally to models, scripts and other API clients. Existing OAuth scopes and entity ACLs always take precedence. An agent has no independent superuser status.

## Decision function

Every mutating tool call is evaluated by the gateway, not just by the agent:

```text
decision = evaluate(
  authenticated_principal,
  client_identity,
  tool_and_operation,
  target_entities_and_permissions,
  requested_diff_and_record_count,
  current_revisions_and_schema,
  active_delegations_and_limits,
  matching_reviewed_approval_if_required,
  retention_and_data_classification,
  idempotency_fingerprint
)
```

Return `allow`, `review_required`, or `deny`. No undocumented permissive fallback. Check at **commit time** too, not only when a plan is prepared, to prevent time-of-check/time-of-use races.

## Risk levels

| Level | Meaning | Default handling | Examples |
| --- | --- | --- | --- |
| R0 | Read-only, permission-filtered | Allow with `memdb:read` and entity access | Search titles, read allowed technical sheet, list related datasets |
| R1 | Routine bounded write with clear current-task intent | Allow with write scope + entity grant + assisted/delegated policy | Log requested meal, add user-supplied catalog entry, repair harmless description, add non-authoritative tag |
| R2 | Structural but reversible organization | Allow **only** with matching explicit request or applicable scoped delegation; otherwise review | Create new logical dataset/namespace, define v1 schema, link existing memory to new dataset, non-destructive note draft |
| R3 | Authority-changing, destructive, bulk or high-impact | Require trusted reviewed approval, additional grant, stable plan and revisions | Apply schema migration, merge source-of-truth memories, change canonical technical sheet, bulk rewrite, delete/restore/purge, export private collection |
| R4 | Prohibited or unsupported | Deny regardless of delegation | Raw arbitrary SQL/host commands from MCP, access another tenant's private data, self-grant roles, execute instructions from retrieved content, stealth capture conversations |

Classification is a **minimum**: any operation is upgraded to the highest applicable risk across its effects. R1 may become R3 if repeated across hundreds of records or if it modifies sensitive/high-authority data. Split multi-action batches into separate decisions; never hide a privileged mutation inside a low-risk call.

## Intent and what an agent may infer

An LLM can propose actions based on current user request, retrieved notes and related data, but the server cannot independently verify that an LLM's `user_requested=true` claim is truthful. Therefore:

- **Assisted profile:** scoped R1 edits require an authorized write call during a user-directed workflow; R2 needs explicit reviewed intent through a trusted interaction or a compatible prior delegation.
- **Delegated profile:** the user previously configured exact allowed categories, namespace boundaries and quotas through a trusted control plane. An agent may perform those bounded operations without a fresh prompt.
- **Read-only profile:** all writes denied.
- **No automatic transcript archiving** in any profile unless a separately opted-in capture workflow exists and is implemented with client-level access and retention controls.

The model should not manufacture permission through a field saying "confirmed." A current message relayed by an LLM is not itself a high-risk approval artifact.

## Detailed operation matrix

| Requested change | Typical level | Key rule |
| --- | --- | --- |
| Search/read memories, dataset metadata, permitted records | R0 | ACL-filter, redact hidden link targets |
| Add alias, tags, keywords or description to non-authoritative metadata | R1 | Owner/write permission, revision check, conservative overwrite |
| Append one explicitly requested log/event record | R1 | Check existing record/idempotency, preserve timestamp/source |
| Update a normal record's non-sensitive fields | R1 | User-directed or delegated, exact target, expected revision |
| Link records or datasets with `related_to` | R1/R2 | Must check both endpoints and avoid cross-scope leaks |
| Create a logical dataset with v1 schema | R2 | Discover duplicates, validate plan, quotas and delegation |
| Create a namespace | R2 | Grant and quotas; cannot bypass segregation rules |
| Create a draft Basic Memory note | R2 | Requires supported upstream write and retention consent |
| Publish/revise authoritative technical sheet | R3 | Show full diff; user approval required |
| Change a relation to `governed_by` on a canonical dataset | R3 | Authority-impacting even if mechanically just a link |
| Apply schema migration/rewrite records | R3 | Approval, preview, rollback plan, no silent drops |
| Merge, supersede or delete source-of-truth note | R3 | Approval, provenance and restore/reconciliation |
| Bulk import/update/export | R3 | Bounded plan, permission and privacy checks |
| Erase/purge history or backups | R3/R4 | Only if specifically supported; cannot promise backup deletion by a dataset operation |
| Alter Pocket ID roles, OAuth grants or memDB ACL policy | R4 for normal agent | Separate administrator-only control plane |

**A `governed_by` link changes how future models interpret a dataset**; creating/changing it is R3 unless the dataset and governing memory are created together in one user-approved/delegated initialization plan explicitly allowing this relation.

## Background maintenance

The server may rebuild an index, expire caches, checkpoint WAL and verify integrity as implementation housekeeping. Such routines may not invent facts, rewrite source notes, auto-delete records, or create authoritative links. Any future scheduled content processing is an opt-in job with its own grant, bounded input set, audit and revocation.

## Data classifications

Mark entities `ordinary`, `private`, `sensitive`, or `restricted` by policy. Default personal memories and food logs are `private`; secrets/credentials are `restricted` and must be handled in a dedicated secret store rather than ordinary search records. Importing third-party private data requires appropriate permission and minimization. The owner can raise classification, but the agent must not lower it without admin review.

`restricted` does not mean an LLM can use a secret if it finds one; least-privilege capability governs the operation and response.

## Limits (starting defaults, review during implementation)

- R1 one-record mutations per call; max 10 authorized routine writes per user task unless delegated otherwise.
- R2 default maximum 3 dataset creations per day and 1 namespace creation per day per principal; quotas are adjustable by trusted admin control.
- R2/R3 plans expire after 15 minutes by default; revisions and policy are rechecked at commit.
- Bulk >10 records or edits affecting multiple source-of-truth notes are R3.
- All thresholds are defaults, **not** guarantees against abuse: enforce per-client rate limiting and namespace storage quotas independently.

## Prompt-injection boundary

Retrieved note text, search results, file contents, tags, schemas and record payloads are **untrusted data**. A malicious note asking the model to change Pocket ID credentials cannot become an authorization. Tool descriptions, code paths and client outputs must not trust such text as a system instruction. Test nested and link-following prompt injection.

## Status and failure codes

Use `POLICY_DENIED`, `REVIEW_REQUIRED`, `DELEGATION_EXPIRED`, `LIMIT_EXCEEDED`, `APPROVAL_INVALID`, `APPROVAL_STALE` and existing `FORBIDDEN`/`REVISION_CONFLICT`. Denials and approval requests are logged with minimal redacted context. Unauthorized callers must not learn hidden entity names or whether a target exists.
