# External documentation and assumptions

**Checked:** 2026-11-05. These are references for design decisions, **not** guarantees about which versions or account features are deployed.

## Basic Memory

- https://docs.basicmemory.com/start-here/what-is-basic-memory
- https://docs.basicmemory.com/reference/mcp-tools-reference
- https://docs.basicmemory.com/concepts/semantic-search

Basic Memory's published reference describes MCP note read/write/edit, search and context-building operations. Actual supported interface, IDs, metadata and transports must be measured against installed version during Phase 0.

## ChatGPT remote MCP

- https://help.openai.com/en/articles/12584461-connecting-custom-mcp-servers-in-chatgpt
- https://developers.openai.com/api/docs/guides/tools-connectors-mcp

At the check date, OpenAI's Help Center documents full custom MCP capabilities for Business and Enterprise/Edu, and limited read/fetch for Pro. It does not establish full custom MCP write availability for personal Plus. Normal chat can choose when to call tools; no guaranteed per-message memory lookup. Test actual client and account. Custom remote MCP must be available to client via an approved network method.

## MCP OAuth

- https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2025-11-25/basic/authorization.mdx

HTTP MCP resource servers support OAuth protected resource metadata, authorization server discovery and audience-bound access tokens. Pocket ID OIDC functionality alone is not proof that its issued tokens/metadata satisfy a particular ChatGPT OAuth MCP flow. Test or use a compatible broker.

## Project-specific verification backlog

- [ ] Record Basic Memory version and read/search/write response examples (sanitized).
- [ ] Record Pocket ID version and supported metadata/claims (no credentials).
- [ ] Test client's connection, token refresh and actual read/write privileges.
- [ ] Confirm note ID remains stable through rename/edit/move.
- [ ] Validate diagram standards against real approved .drawio examples.
