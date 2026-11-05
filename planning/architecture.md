# System architecture (target)

## Runtime layout

One Proxmox VM hosts memDB, SQLite storage, a Basic Memory instance, and an authenticated HTTPS ingress. Basic Memory is independently deployed/upgraded and is **not part of this repository**.

```text
ChatGPT / Codex / other MCP clients
              |
              | HTTPS MCP; OAuth authorization
              v
    Protected ingress / reverse proxy
              |
              v
       memDB MCP Gateway
          |         |
          |         +---- SQLite dataset engine
          |                - registry + metadata
          |                - JSON-schema validation
          |                - records and links
          |                - indexed search + audit
          |
          +--- Basic Memory adapter ---> Basic Memory server
                                          |
                                          +--- Markdown vault and own index
              ^
              |
        Pocket ID (OIDC identity provider)
```

**Trust boundary:** reverse proxy and OAuth integration are not interchangeable. The ingress handles TLS/routing; the gateway validates credentials/audience/scopes or explicitly trusts a separately verified auth service via a hardened, authenticated channel. Never trust client-supplied `X-User` headers.

## Logical modules

1. `gateway`: MCP transports, input parsing, pagination, consistent errors.
2. `auth`: OIDC discovery/JWKS validation, token issuer/audience/expiry/scope checks, principal context, ACL checks.
3. `catalog`: namespace, dataset and schema lifecycle. Explicit metadata.
4. `records`: JSON record validation, query, filtering, transactional changes and revisions.
5. `links`: graph between memory references, dataset entities and individual records.
6. `search`: FTS-backed metadata/data discovery and pagination; adapter-assisted Basic Memory search where supported.
7. `adapter.basic_memory`: isolated upstream client, mapping stable external note IDs to local references, handling stale/deleted notes.
8. `audit`: append-only write events, actor and correlation IDs, redaction rules.

## External memory references

Basic Memory notes are not copied as canonical memDB records. A local entity reference stores `provider=basic_memory`, `external_id`, title/description/tags snapshot, `last_verified_at` and resolution state. Refresh via supported Basic Memory APIs. If a note is renamed, keep the same external ID; if removed, mark its reference unavailable without deleting historical links. **Confirm Basic Memory's stable identifier guarantees before implementation.**

## Search and context resolution

`discover(query)` returns compact scored **descriptors**, combining memDB metadata search and, if supported, Basic Memory search results; it does not dump entire datasets or complete notes. The client explicitly requests `describe` or `read` and follows links. Distinguish reliable keyword search from optional embedding search; do not claim semantic matching without a configured provider. No additional generative model is required.

## Reliability

- SQLite is authoritative for datasets and local link metadata.
- Basic Memory is authoritative for note bodies.
- Cross-service changes are **not** globally atomic. Use staged operations and reconciliation rather than pretending there is a distributed transaction.
- Every mutating operation supports idempotency keys, revision checks and an audit entry.
- Core local dataset operations should work when Basic Memory is down; external note fetches should return `UPSTREAM_UNAVAILABLE`.

See [operations](operations.md), [security](security-pocketid.md) and [diagrams/architecture.drawio](diagrams/architecture.drawio).
