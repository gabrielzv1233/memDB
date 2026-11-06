# Deployment and operations (target)

## VM deployment

Target: dedicated Proxmox VM. Use Docker Compose or systemd as chosen during implementation. Components:
- `memdb-gateway`: Python application, MCP HTTP endpoint, health checks.
- `basic-memory`: independent service, accessed by a private supported adapter.
- `data`: persistent volume with `memdb.sqlite3`, WAL files, backups.
- `ingress`: Zoraxy or another secure HTTPS reverse proxy to gateway.
- `pocket-id`: existing identity provider; not bundled into memDB.

Use a non-root application account, read/write permissions only to its data directory, resource limits, loopback/private networks for internal services and explicit firewall rules. Pin container images and dependencies.

## SQLite guidance

Enable WAL mode, foreign keys, busy timeout; use one writer per logical transaction and avoid simultaneous long-running transactions. Keep DB and WAL/SHM on appropriate persistent local storage, not an unreliable remote network mount. Use migration tooling with backups and rollback plans.

## Backup and restoration

Back up with the SQLite online backup API or `VACUUM INTO` using a consistent snapshot, not an unsafe raw copy of the main DB while WAL writes are active. Include migration version and configuration (excluding secrets). Basic Memory's vault and index have **separate** backup ownership; protect both for complete recovery. Schedule periodic restore drills. Encrypt and control access to backups.

## Observability

Expose internal health/readiness endpoints with no secrets; structured logs with request IDs, operation name, latency, outcome and redacted principal; count Basic Memory adapter failures, SQLite busy errors, conflicts, invalid schemas, denied requests and backup freshness. Audit writes without logging full sensitive payloads.

## Outages

If Basic Memory is unavailable: local datasets continue, external reference reads report `UPSTREAM_UNAVAILABLE`. If SQLite is unavailable/corrupt: fail writes safely, do not silently fall back to in-memory persistence. Require verified restoration and migrations before resuming.

## Deployment checklist

- [ ] Dedicated VM user, persistent volume and resource budget
- [ ] HTTPS ingress and private-only Basic Memory service
- [ ] Pocket ID/OAuth MCP flow verified with actual client
- [ ] Secrets stored outside Git
- [ ] SQLite migration and backup plan tested
- [ ] ACL/security negative tests pass
- [ ] Restart/recovery and backup restore drill pass

## Autonomous-management operations

Back up server-owned policy tables, grants, review plans and audit alongside application data; store no approval bearer proofs in external chat or public logs. Review/authorization UI must be independently authenticated and separated from model-callable MCP tools. Restart must preserve grant revocations, approval consumption and per-principal quotas. An offline Basic Memory server never justifies bypassing approvals or losing audit events. Periodic index maintenance is okay; autonomous generative content processing requires separate opt-in and is not part of v1.
