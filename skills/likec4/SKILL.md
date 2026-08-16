---
name: likec4
description: >-
  Author, edit, and validate software architecture diagrams as code using the
  LikeC4 DSL (.c4 / .likec4 files) — the C4 model (context, container,
  component, deployment) expressed as a text-based specification/model/views
  language. Use whenever someone asks to create or update an architecture
  diagram, a C4 model/context/container/component/deployment diagram, or asks
  about LikeC4 syntax, the likec4 CLI, or the LikeC4 MCP server. Covers the
  full DSL grammar (specification, model, relationships, views, predicates,
  deployment, styling) and the CLI/MCP tooling so output is syntactically
  correct on the first try.
triggers:
  - likec4
  - c4 model
  - c4 diagram
  - architecture as code
  - architecture diagram
  - context diagram
  - container diagram
  - component diagram
  - deployment diagram
  - .c4 file
  - .likec4 file
metadata:
  homepage: https://likec4.dev
  spec: https://likec4.dev/dsl/specification
  cli: npm package `likec4` (https://likec4.dev/tooling/cli/)
  mcp: npm package `@likec4/mcp` (https://likec4.dev/tooling/mcp/)
---

# LikeC4

[LikeC4](https://likec4.dev) is a DSL + toolchain for describing software
architecture as code, based on the [C4 model](https://c4model.com). You write
`.c4` (or `.likec4`) text files describing elements, relationships, and views;
the toolchain renders them as diagrams (React app, static site, PNG/SVG,
Mermaid/D2/PlantUML/drawio) and can expose the model to AI agents via MCP.

## When to use

Any request to create, edit, or review an architecture diagram — system
context, container, component, or deployment diagrams — where the deliverable
is (or should be) LikeC4 source rather than a hand-drawn diagram. Also use
when someone asks about LikeC4 syntax, the `likec4` CLI, or its MCP server.

## Core mental model

A LikeC4 **project** is a directory of `.c4`/`.likec4` files (workspace root
= nearest `likec4.config.json`, or the nearest `package.json`/`.git` if
absent). **All files in the project are merged into one model** — there is no
`import` statement; an element defined in one file is referenced by its
qualified name from any other file in the project. Split files by concern,
not by language boundary:

```
specification.c4       # element/relationship kinds, tags, colors — usually one file
model.c4                # or split per subsystem: backend.c4, frontend.c4, ...
views.c4                # or co-located with the elements they visualize
deployment.c4
```

Each file holds one or more of four top-level blocks — `specification`,
`model`, `deployment`, `views` (plus `global` for shared styles/predicate
groups) — and a project can have many files contributing to each block.

**There are no built-in element or relationship kinds.** Every `kind` used in
`model { }` or `deployment { }` (e.g. `person`, `system`, `service`,
`database`) must first be declared in `specification { element <kind> }` (or
`deploymentNode <kind>` for deployment nodes) — using an undeclared kind is a
compile error, not a default/fallback.

## Workflow

1. **Check for an existing project root** — look for `likec4.config.json` or
   existing `.c4`/`.likec4` files before assuming a fresh project.
2. **Define kinds first** in `specification { }` (element kinds, relationship
   kinds, tags, colors) — see
   [`references/dsl-reference.md`](references/dsl-reference.md#specification-block).
3. **Model elements and relationships** in `model { }` — see
   [`references/dsl-reference.md`](references/dsl-reference.md#model-block).
4. **Define views** in `views { }` using `include`/`exclude` predicates to
   scope what each diagram shows — see
   [`references/dsl-reference.md`](references/dsl-reference.md#views-block).
5. **Validate before handing off**: `npx likec4 validate` (and
   `npx likec4 format --check` for style). Fix errors — don't hand back
   source you haven't validated if the CLI is available.
6. Optionally **export** a rendered artifact: `npx likec4 export png -o
   ./assets` or `npx likec4 build -o ./dist` for a browsable static site. See
   [`references/cli-and-mcp.md`](references/cli-and-mcp.md).

Full grammar reference: [`references/dsl-reference.md`](references/dsl-reference.md).
CLI + MCP server reference: [`references/cli-and-mcp.md`](references/cli-and-mcp.md).
Worked examples: [`examples/basic-model.c4`](examples/basic-model.c4) (specification
+ model + views) and [`examples/deployment.c4`](examples/deployment.c4)
(deployment topology).

## Gotchas

- **Kind before use.** Declare every element/relationship/deploymentNode kind
  in `specification { }` before referencing it in `model { }` / `deployment { }`.
- **Tags must come first.** Inside an element/relationship body, `#tag`
  declarations must precede other properties (`title`, `description`, ...).
- **Naming.** Identifiers: letters, digits, `-`, `_` — cannot start with a
  digit or contain a `.` (periods are the namespace separator for qualified
  refs like `cloud.backend.api`).
- **No duplicate names within the same parent** (same constraint applies to
  deployment nodes under the same parent).
- **`extend` needs the fully-qualified name** (`extend cloud.service2`, not
  `extend service2`) when adding to an element from a different file/scope.
- **Scoped views need qualified `include`s relative to their scope** — `view
  of cloud.backend { include api }` resolves `api` to `cloud.backend.api`,
  not a top-level `api`.
- **Unscoped `include *`** only pulls in top-level elements; use `.{*,**,_}`
  suffixes (`cloud.*` children, `cloud.**` descendants, `cloud._` expand) to
  reach nested ones.
- **Markdown text** (descriptions, summaries) needs triple-quoted strings:
  `description '''...'''`.
- If the `likec4` CLI isn't available/installed, say so rather than guessing
  whether source is valid — the grammar has enough sharp edges (kind
  ordering, tag ordering, qualified names) that eyeballing isn't reliable.
