# ADR 0001: Basic Memory adapter + generic SQLite datasets

**Status:** Proposed
**Date:** 2026-11-05

## Context

A user wants AI agents to create structured datasets linked to rich knowledge/memory notes, with tags, descriptions and bidirectional references. Basic Memory provides Markdown-first knowledge storage and search, but the structured dataset engine requires exact schema validation, relational links, audit, transactions and programmatic queries. One Proxmox VM will host this behind Pocket ID authentication.

## Decision

Build one custom Python MCP gateway with a versioned adapter to separately deployed Basic Memory, and one physical SQLite file containing a generic entity/dataset/record registry. Store records as JSON validated against immutable versioned JSON Schemas. Build typed graph links across local entity IDs and Basic Memory external references. Use SQLite FTS5 for local metadata discovery. Do not run a second generative model or give agents arbitrary SQL.

## Why

- Basic Memory remains independent and upgradable.
- One database file simplifies deployment while retaining logical subdivisions.
- An agent can add data structures via sanctioned tools without code changes.
- Permission checks, revision control and auditing are centralized.
- Technical sheets remain full-length Markdown instead of over-compressed memories.

## Alternatives considered

1. Arbitrary SQL tables per AI-created dataset: flexible but hard to permission, migrate and validate safely.
2. Everything as Basic Memory Markdown: poor fit for exact relational queries and growing record collections.
3. A graph DB/large RDBMS from day one: adds operational weight before scale demands it.
4. Local LLM watching messages: unnecessary and unavailable for opaque client conversations.

## Consequences

More custom code for schema evolution, validation and graph traversal. External note writes/read availability are not transactionally coupled to SQLite. Search quality initially depends on keywords/FTS; embeddings are optional. OAuth compatibility must be verified separately.

## Revisit when

A real workload demonstrates unacceptable SQLite write contention, complex joins, multi-tenant isolation or unsupported Basic Memory adapter requirements.
