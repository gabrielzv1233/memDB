# Security: Pocket ID, OAuth and MCP

## Authentication architecture

Pocket ID is the intended OIDC identity provider. A protected HTTPS endpoint (possibly behind Zoraxy) exposes **only the public gateway MCP service**. The gateway must enforce its own principal identity and authorizations. The separate Basic Memory service and SQLite file are private to the VM.

**Important:** an OIDC login and an OAuth access token accepted by a remote MCP resource are not automatically interchangeable. Before implementation, verify the chosen ChatGPT/other MCP client's supported transport and OAuth discovery/authorization mechanism, the token audience, PKCE requirements and Pocket ID capabilities. If Pocket ID cannot issue tokens suitable for the resource server, use a standards-compliant authorization server/broker; do not treat an ID token as an API access token.

## Required security properties

1. TLS outside the VM. Limit inbound paths to approved gateway routes; rate limit.
2. Verify signature against trusted JWKS and allowed algorithms, exact issuer, audience, expiry, not-before and required scopes. Cache JWKS with rotation support.
3. Authenticate user identity based on verified token claims; do not trust arbitrary `Remote-User`, `X-OIDC-*` or other headers originating from clients.
4. If proxy authentication is used, secure proxy-to-service traffic, pin trusted proxy origins and remove spoofable incoming identity headers. Prefer gateway-side token validation.
5. Support scoped principals (`reader`, `editor`, `schema_admin`, `operator`) mapped explicitly from verified identity and approved grants, not merely from an unvalidated group name.
6. ACLs must apply to namespaces, datasets, individual records where configured, links and search results; graph traversal cannot reveal private neighbors.
7. Credentials and secrets are environment/secret-store values, never committed to Git. Sanitize logs and audit events.
8. Protect dangerous changes (schema migration, namespace deletion, purge) with separate authorization and explicit user action.
9. Rotate signing keys and client secrets; apply CSRF defenses where web-browser sessions are used.
10. Do not expose Basic Memory's unauthenticated transport or memDB admin endpoints directly to the Internet.

## Proposed permissions

| Scope / grant | Access |
| --- | --- |
| `memdb:read` | Search/read permitted resources |
| `memdb:write` | Record and metadata mutations in granted namespaces |
| `memdb:schema` | Schema proposals and approved applications |
| `memdb:admin` | Namespace policy, data export and destructive operations |

Scopes indicate potential capability; entity-level ACLs still decide actual access. Service accounts should default to minimal scope. Keep user-specific food records private by default.

## Threat cases to test

- Fake identity headers bypassing proxy
- Valid issuer token with wrong audience
- Expired, not-yet-valid or malformed tokens
- Privileged token missing required scope
- Search or link traversal leaking another user's private record
- Cross-namespace link permission escalation
- Prompt injection in memory/record text
- Replay/idempotency-key collision
- Compromised Basic Memory service and malicious upstream response
- SSRF through unvalidated external reference identifiers

## Pocket ID rollout checklist

Document actual Pocket ID issuer URL, scopes, clients, group claims, JWKS and supported flows **after verification** in private deployment configuration. Never guess hardcoded domains or secrets. Test remote MCP client sign-in and token refresh before implementing the entire application.

## Autonomy-specific authorization

Use the [autonomy policy](autonomy/policy.md) and [delegation approval protocol](autonomy/delegation-and-approvals.md). OAuth scopes and entity ACLs are necessary but not sufficient to bypass R2/R3 controls. Delegations are server-owned, scoped and revocable. Exact R3 approvals must originate from a trusted authenticated review UI/control plane with a server-calculated diff; a model-provided `confirmed`/`approved` argument has no authority. Ordinary dataset tools must never be able to mutate grant, review or policy tables. Verify revocation and approval state again at commit time.
