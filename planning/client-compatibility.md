# ChatGPT chat, Codex and other MCP client compatibility

**Status:** research-based constraints (2026-11-05) + required live integration tests. Do **not** confuse self-hosting a valid MCP server with being able to use every tool from any ChatGPT plan.

## ChatGPT is the main target

The user primarily uses normal ChatGPT chat mode and cannot rely on Agent Skills being available. Therefore retrieval must be model-tool-based: a **short built-in-memory pointer** and optional client instructions refer the model to memDB; memDB exposes a compact `context_lookup` and explicit read/data tools. There is **no server-side way to force ChatGPT to search every conversation**.

OpenAI current help documentation: https://help.openai.com/en/articles/12584461-connecting-custom-mcp-servers-in-chatgpt

Important plan constraint from current docs: **Full custom MCP connections and write actions are documented for Business and Enterprise/Edu workspaces**; Pro users are documented as having read/fetch custom connections. The help article does **not** establish full-write availability for a personal Plus subscription. This must be tested with the actual available account and client before promising in-chat dataset creation or saves. Do not assume functionality is enabled merely because the MCP endpoint is valid.

## Functional capability levels

| Capability | Client requirement | Fallback |
| --- | --- | --- |
| Search and read technical sheet | Connection to custom MCP with read access | User-supplied sheet or native context |
| Query datasets | Read-only custom MCP tool access | External app or API client |
| Create/update records | Custom MCP write actions allowed by client's plan/policy | Codex, user-run app, or other authorized MCP client |
| Automatically check each message | Host-level middleware under user's control | Best-effort model instruction/reminder |
| Use existing Agent Skills | Client/feature must support skills | Plain Markdown technical sheets over MCP |
| Authorization with Pocket ID | Compatible OAuth flow and token audience | Conformant OAuth broker or supported client connection |

No claim that a normal ChatGPT Plus conversation can perform MCP writes is made here. A separate authorized app can still use the same memDB backend.

## Source and protocol references

- OpenAI ChatGPT custom MCP setup: https://help.openai.com/en/articles/12584461-connecting-custom-mcp-servers-in-chatgpt
- MCP authorization specification: https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2025-11-25/basic/authorization.mdx
- OpenAI API remote MCP reference: https://developers.openai.com/api/docs/guides/tools-connectors-mcp

MCP HTTP authorization requires protected-resource discovery and access tokens intended for the resource. Pocket ID's OIDC login, user-info claims and session cookie **do not by themselves establish a standards-conformant MCP OAuth resource server**. Verify refresh tokens, PKCE, registration support, scopes, audience, well-known metadata and redirect flow with a real client. Never accept an ID token as a resource access token or pass upstream bearer tokens through blindly.

## Phase-0 client matrix to fill with evidence

| Test | ChatGPT web | Codex | Custom client |
| --- | --- | --- | --- |
| Connect to remote HTTPS MCP | Unknown, test actual account | Unknown | Under our control |
| Complete Pocket ID OAuth | Unknown | Unknown | Under our control |
| Read `context_lookup` | Unknown | Unknown | Under our control |
| Invoke `record_create` | Plan/policy dependent, must verify | Unknown | Can support by design |
| Token refresh/reconnect | Unknown | Unknown | Under our control |
| Called on implicit request | Best effort only | Client dependent | Deterministic possible |

For each tested client record date, version, account entitlement, transport, exact tools enabled, screenshots/error text without secrets, and pass/fail reason in a future `planning/compatibility/` report.

## Acceptance gate

Do not spend the majority of implementation effort on data writes until one intended chat client can authenticate and make a real permitted read call. If ChatGPT Plus cannot connect custom MCPs, keep memDB useful through Codex/custom clients and document that limitation rather than claiming it works in ChatGPT normal chat.
