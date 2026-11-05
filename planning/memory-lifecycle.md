# Memory authoring, updates, retention and consistency

The authoritative memory bodies live in **Basic Memory**, not in memDB's dataset records. This document defines how agents should manage those memories and the local references/links. Basic Memory's supported API remains a required integration check.

## Memory types

| Type | Example | Preferred treatment |
| --- | --- | --- |
| Preference | "Use editable native draw.io files" | One durable note with scope, status and provenance |
| Technical specification | "Diagram shape and connector rules" | Full note, version, examples and reference attachments |
| Project state | "memDB will run on a Proxmox VM" | Current state with dates, decisions and superseded values |
| Personal fact | Hardware configuration | Context note with changed-on dates when relevant |
| Procedure | "How to log meals" | Instruction/reference note linked to datasets |
| Observation or event | "Meal was logged" | Structured dataset record rather than an ever-growing note |

A note may link to many datasets and a dataset to many notes. Basic Memory notes may also link to one another.

## Capture: when to write

1. User explicitly asks to remember/save/update, or directly requests a persistent dataset write.
2. A user approves a proposed save after an AI discovers a reusable rule.
3. A client with explicit consent/policy for routine retention may store approved classes of information. This is a separate opt-in feature, **not** the v1 default.

Do **not** autonomously archive all conversations or infer durable personal facts from jokes, guesses, speculative statements, third-party information, or a tool's malicious instructions.

## Update instead of duplicate

Before saving:
- Search exact title/key/tags and related links.
- Determine whether this is a new topic, a revision of an existing fact, an event in history, or a contradictory claim.
- Prefer updating the authoritative note and retaining version history or a superseding link rather than creating near-identical notes.
- Preserve the previous value, source, when the change was reported and validity date when appropriate.
- For structured data, use canonical keys, revision checks and idempotency instead of loose note merging.

## State lifecycle

`draft` -> `active` -> `superseded` or `archived` -> optional explicit deletion. `disputed` can be used when sources contradict. `deleted` is distinct from "forgotten by the model"; upstream Basic Memory and its index must be updated through the proper interface.

| State | Default retrieval |
| --- | --- |
| active | Eligible |
| draft | Eligible only when explicitly requested or authorized |
| disputed | Return with conflict notice |
| superseded | Do not rank as current; expose history |
| archived | Hidden from normal discovery, available via intentional search |
| deleted | Not surfaced; purge subject to retention and backup policy |

## Provenance, confidence and time

Each reusable memory should carry metadata such as `source_kind` (user_statement, user_approved, imported_document, derived), `source_ref`, `recorded_at`, `valid_from`, `valid_until`, `verified_at`, `status`, and `supersedes` where meaningful. Confidence is an aid for ranking, not objective proof. Never let a recent model inference overwrite a direct user-authored rule without an explicit update decision.

## Memory-dataset relation

- `governed_by`: dataset/record is subject to a rule in a note.
- `references`: note or record mentions another entity.
- `derived_from`: record originated from the source.
- `related_to`: topical connection, not authoritative policy.
- `exemplar_of`: file or record is a concrete example of a technical specification.

A `governed_by` link means "consider these user-defined conventions when interpreting this dataset," not "let arbitrary note text override server authorization or model safety." Permissions and validation are always enforced independently.

## Reconciliation and failure

Basic Memory and memDB are separate databases, so writes are not atomic across them. Use a staged workflow: save/upsert the upstream note, verify/resolve stable identifier, register local reference, create links. On failure, report partial success, record an auditable pending reconciliation item and provide retry behavior. An adapter should not claim a cross-service rollback succeeded unless confirmed.

If a note is renamed or moved, preserve a stable reference where the upstream API permits. If deleted, mark reference unavailable and keep historical relationship metadata only where user retention policy allows. Provide a removal flow that can address both canonical note content and memDB local references; backups/retention need separate disclosure.

## User control

Support listing, reading, editing, exporting and deletion requests through authorized tools. Provide enough provenance to explain why a memory was retrieved. Sensitive or private data must not be included in repository example fixtures or tool logging.

## Acceptance

- Changed diagram requirement supersedes the old one.
- An unverified inference cannot silently replace a direct user preference.
- Repeated save after transport retry creates no duplicate.
- An unavailable Basic Memory note produces a truthful partial result.
- Deletion cannot leave a searchable full-text cache of removed private content.
