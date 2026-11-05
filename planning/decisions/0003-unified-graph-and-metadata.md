# ADR 0003: Unified entity descriptors and many-to-many linking

**Status:** Proposed  
**Date:** 2026-11-05

## Problem

The user does not want a hardcoded food-specific app. They want agents to define datasets and attach them to memory notes, each with tags and descriptions, and to link individual records if useful.

## Decision

One SQLite physical database contains logical datasets, generic JSON records, entity metadata and typed many-to-many links. External Basic Memory notes use local reference entities. All entities have stable IDs, titles, descriptions, tags, optional trigger keywords and timestamps. Link traversal is bidirectional, permission-aware and bounded.

Schema versions are immutable, writes use idempotency and optimistic concurrency, and notes remain canonical upstream.

## Consequences

The service supports arbitrary future domains without server-code changes, but requires careful schema migrations, unique keys, graph/ACL enforcement and cross-service reconciliation. Tags and note descriptions aid retrieval; they do not grant permissions or execute commands.
