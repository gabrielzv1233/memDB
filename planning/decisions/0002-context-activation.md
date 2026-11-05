# ADR 0002: Context reminder and just-in-time MCP retrieval

**Status:** Proposed  
**Date:** 2026-11-05

## Problem

Normal chat LLMs cannot store every nuanced technical sheet or personally relevant record in their built-in memory. User wants to work in normal ChatGPT chat rather than rely on Agent Skills. A remote MCP cannot read opaque conversation streams unless the client invokes it.

## Decision

Use a short user-controlled native-memory pointer to memDB, client-level instructions where available, one compact `context_lookup` tool, and explicit full-note reads and dataset queries. Provide optional deterministic keyword/metadata prefetch **only** in a client under our control. Do not run a second background generative model. Include source freshness, tags, scores, relationships and permissions in discovery. Never claim guaranteed automatic invocation in ChatGPT.

## Consequences

Works across different capable clients; avoids copying full context into built-in memory. Missed tool calls remain possible, especially in ChatGPT chat. User-initiated `@memDB` or explicit "check memDB" may still be necessary. Clients without write access can still read context if their plan/policy allows.

## Revisit when

Clients provide authenticated, explicitly authorized pre-message middleware or guaranteed memory triggers with documented APIs.
