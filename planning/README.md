# memDB planning hub

This is the **design workspace**, not the app source tree. Start with architecture, then data model, MCP API, and security. Agents should read `../AGENTS.md`.

| Document | Purpose |
| --- | --- |
| [Architecture](architecture.md) | Boundaries, components, request flows and Basic Memory adapter |
| [Data model](data-model.md) | Entity registry, tags, schemas, records, links, revisions, indices |
| [MCP contract](mcp-contract.md) | Discovery, read/write tools, validation, pagination and errors |
| [Security and Pocket ID](security-pocketid.md) | OIDC, OAuth for MCP, scopes, authorization and network boundaries |
| [Operations](operations.md) | Proxmox VM deployment, backup, recovery and observability |
| [Roadmap](roadmap.md) | Implementation phases and acceptance tests |
| [Decisions](decisions/0001-core-architecture.md) | Initial architecture decision and alternatives |
| [Example: food](examples/nutrition.md) | Memory-to-dataset association example |
| [Diagrams](diagrams/) | Editable architecture and entity relationship drawings |

## The product model

A **memory** is an external Basic Memory note with an external ID, title, tags and description (Basic Memory owns its content). A **dataset** is a named, described, tagged collection of validated JSON records in memDB. A **record** is a typed row in a dataset. **Links** can join any registered entity, including Basic Memory notes, datasets and records. All entities can be discovered by name, tag, description, or relation.

Example: a Basic Memory note "Food tracking rules" links to both `nutrition.foods` and `nutrition.meals` datasets. An AI can create the datasets and schemas with tools instead of a developer hardcoding them. The AI follows the linked note when working with the data.

## Non-goals for v1

- Replacing Basic Memory or duplicating its knowledge search engine.
- Running a local generative LLM to interpret conversations.
- Having the MCP gateway automatically see every ChatGPT message.
- Giving an AI arbitrary DDL/DML SQL privileges.
- Implementing a full visual dashboard or Obsidian clone.
- Assuming ChatGPT's built-in memory guarantees tool invocation.

## Open design questions

- Which supported Basic Memory API/MCP transport and metadata endpoints will be used in the deployed version?
- Should the v1 gateway expose a composite search, or separate search followed by explicit link expansion?
- How should service accounts and human users share or segregate private memory namespaces?
- Which ChatGPT custom MCP transport and OAuth capabilities are available to the actual account/client?

These must be resolved in implementation spikes and ADRs, not guessed.
