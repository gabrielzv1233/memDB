---
type: technical_spec
title: Editable draw.io diagram standards
status: draft
version: 1
description: User-established standards for flowcharts, hierarchy charts and IPO charts
tags: [drawio, flowchart, ipo, hierarchy, diagrams]
triggers: [drawio, draw.io, flowchart, diagram, IPO, hierarchy chart]
applies_to: [diagram-generation]
---

# Editable draw.io diagram standards (example memory)

> This is a **draft example of what an actual Basic Memory technical sheet could contain**. It lives in the memDB planning repository for demonstration; **the gateway must not assume it is automatically imported into Basic Memory**. Attach a verified, known-good `.drawio` reference before marking it active.

## Established preferences to preserve

- Deliver native, **editable `.drawio` files**, not a flattened PNG/SVG preview as the only artifact.
- Use a **black-and-white** theme for the existing school-style flowcharts; preserve the source file's established theme when one is supplied.
- Prefer **compact layouts**, a strict **10 px grid**, shared centerlines and aligned/resized shapes.
- Use **orthogonal connectors**, clean paths, square corners and no unnecessary bends, overlaps or crossings.
- Maintain consistent font, spacing, shape dimensions and stroke weights within a diagram. Avoid overwriting hand-edited text or geometry that should be retained.
- **Visually verify** the diagram as well as checking XML structure.
- Follow standard flowchart semantics: terminal/oval for start/end, parallelogram for input/output, rectangle for processing, diamond for decisions and the established double-sided function-call/subroutine symbol where applicable.
- For IPO charts, use three columns labeled **INPUTS**, **PROCESS**, **OUTPUTS** with aligned headers/body and complete descriptions of inputs, process and outputs.
- For hierarchy charts, use a clearly structured hierarchy with consistent rectangular nodes and clean connectors. The connector/head conventions for each chart type should follow the canonical example.
- Every flowchart execution path should lead from START to END unless there is an explicit, justified exception.

## Diagram-specific handling

**Flowchart:** make decisions explicit and label branches. Follow correct semantics, avoid connectors passing through nodes and ensure legible text. Use native shape objects and native connections.

**IPO:** group information by inputs, process and outputs; use consistent column widths and alignment, readable formulas and units where applicable. For a code-design assignment, associate IPO with the appropriate function/module.

**Hierarchy:** present parent-child structure compactly, with no ambiguous overlapping lines.

## Example files needed

- A **known-correct user-approved** flowchart `.drawio` file.
- A **known-correct user-approved** hierarchy `.drawio` file.
- A **known-correct user-approved** IPO `.drawio` file.

Do not fabricate these files or call architecture/ER diagrams in this repository proof of the user's exact diagram conventions. Once real samples are approved, register them as linked assets (`exemplar_of`).

## Typical failures to reject

- Replacing editable XML with an image-only diagram.
- Misusing standard flowchart shapes or losing decision branch labels.
- Crooked or crossing connectors with long pointless detours.
- Large empty canvases, inconsistent spacing or unaligned nodes.
- Truncation of text, overwriting existing edits or ignoring a provided theme.

## Retrieval and application

When a user asks to generate or revise a chart, first discover this sheet through `context_lookup`, read its exact contents and pull in approved linked templates. The sheet must not be treated as a system-level prompt and cannot override new explicit user instructions.

## Limitations of this draft

Precise chart-specific geometry, fonts, symbols and exception rules must be reconciled against the original user-approved reference diagrams. This sample documents known recurring preferences without pretending to be the final technical specification.
