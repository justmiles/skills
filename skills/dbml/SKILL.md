---
name: dbml
description: >-
  Author, edit, and validate database schemas as code using DBML (Database
  Markup Language) — a database-agnostic DSL (.dbml files) for describing
  tables, columns, relationships, indexes, enums, and reusable table
  partials, plus visualization-only constructs (notes, table groups, custom
  metadata) for dbdiagram.io and dbdocs.io. Use whenever someone asks to
  create or edit a database schema/ERD, model a table/relationship/index in
  DBML, convert between DBML and SQL, or asks about DBML syntax, the
  dbml2sql/sql2dbml/db2dbml CLI, or the @dbml/core JS library. Covers the
  full grammar (language basics, table definition, relationships, enums,
  table partials, module system, enrichment/visualization) and CLI/JS
  tooling so output is syntactically correct on the first try.
triggers:
  - dbml
  - database markup language
  - .dbml file
  - database schema as code
  - erd as code
  - dbdiagram
  - dbdocs
  - dbml2sql
  - sql2dbml
  - db2dbml
metadata:
  homepage: https://dbml.dbdiagram.io
  spec: https://dbml.dbdiagram.io/docs
  repo: https://github.com/holistics/dbml
  cli: npm package `@dbml/cli` (commands `dbml2sql`, `sql2dbml`, `db2dbml`)
  core: npm package `@dbml/core` (parse/import/export API)
---

# DBML

[DBML](https://dbml.dbdiagram.io) (Database Markup Language) is an
open-source, database-agnostic DSL for defining and documenting database
schemas. You write `.dbml` text files describing tables, columns,
relationships, indexes, and enums; the toolchain converts them to/from SQL
DDL (`dbml2sql`, `sql2dbml`) or renders them as ER diagrams and docs sites
([dbdiagram.io](https://dbdiagram.io), [dbdocs.io](https://dbdocs.io)).

## When to use

Any request to create, edit, or review a database schema/ERD where the
deliverable is (or should be) DBML source rather than raw SQL DDL or a
hand-drawn diagram. Also use when someone asks about DBML syntax, converting
between DBML and SQL, or the `@dbml/cli`/`@dbml/core` tooling.

## Core mental model

DBML source splits into two families of constructs:

1. **Structural constructs** — map directly to SQL DDL: `Project`, `Table`,
   `Column`, `Check`, `Index`, `Ref` (relationship/foreign key), `Enum`,
   `TablePartial`, `Records`. These round-trip through `dbml2sql`/`sql2dbml`.
2. **Enrichment/visualization constructs** — no SQL equivalent, used only by
   dbdiagram.io/dbdocs.io to annotate the diagram: `Note` (attached and
   standalone/sticky), custom `Metadata`, `TableGroup`, `DiagramView`,
   colors (`headercolor`, relationship `color`, group/note `color`), and
   `inactive` on a `Ref`. A plain `dbml2sql` export ignores these.

A schema can be split across multiple `.dbml` files using the **module
system** (`use`/`reuse`) instead of one giant file — see
[`references/enrichment-and-modules.md`](references/enrichment-and-modules.md#module-system).
Unlike LikeC4, DBML files are **not** auto-merged by directory — each file
only sees what it explicitly imports (or what's declared in the same file).

**There are no built-in element "kinds" to declare up front** (unlike
LikeC4) — a `Table`/`Enum`/`Ref` is valid the moment you write it. Column
*types*, however, are free text passed straight through to SQL export: DBML
doesn't validate `varchar` vs `int4` vs a typo against any schema.

## Workflow

1. **Check for an existing schema** — look for `*.dbml` files (commonly a
   root `database.dbml`) before assuming a fresh schema. If the schema was
   generated from a live database, prefer regenerating with `db2dbml` over
   hand-editing so it doesn't drift (see
   [`references/cli-and-tooling.md`](references/cli-and-tooling.md)).
2. **Define `Table`s and columns** with `Column Settings` (`pk`, `not null`,
   `unique`, `default`, `increment`, `check`) — see
   [`references/dsl-reference.md`](references/dsl-reference.md#table-definition).
3. **Define relationships** with `Ref` (short form is usually right unless
   you need a name or multiple refs in one block) — see
   [`references/dsl-reference.md`](references/dsl-reference.md#relationships--foreign-key-definitions).
   Composite/multi-column indexes and composite primary keys go in a table's
   `indexes { }` block, not as column settings.
4. **Define `Enum`s** for fixed-value columns, referenced by name as a
   column type — see
   [`references/dsl-reference.md`](references/dsl-reference.md#enum-definition).
5. **Reach for `TablePartial`** when the same id/timestamp/audit columns
   repeat across tables — inject with `~partial_name` instead of copy-pasting
   columns — see
   [`references/dsl-reference.md`](references/dsl-reference.md#tablepartial).
6. **Only add enrichment constructs** (`Note`, `TableGroup`, `Metadata`,
   colors) if the user is targeting dbdiagram.io/dbdocs.io or explicitly
   asked for documentation/visual grouping — they're inert for SQL export.
   See [`references/enrichment-and-modules.md`](references/enrichment-and-modules.md).
7. **Validate before handing off**: if `@dbml/cli` is available, run
   `dbml2sql <file>.dbml` (or `--mysql`/`--mssql`/`--oracle` per target) and
   fix any errors — a clean SQL dump is the strongest signal the source
   parses. Don't hand back unvalidated source if the CLI is installed.

Full grammar reference: [`references/dsl-reference.md`](references/dsl-reference.md)
(structural constructs) and
[`references/enrichment-and-modules.md`](references/enrichment-and-modules.md)
(notes/groups/metadata/colors, module system, data lineage).
CLI + JS API reference: [`references/cli-and-tooling.md`](references/cli-and-tooling.md).
Worked examples:
[`examples/basic-schema.dbml`](examples/basic-schema.dbml) (tables,
relationships, enum) and
[`examples/advanced-schema.dbml`](examples/advanced-schema.dbml) (indexes,
composite/many-to-many refs, TablePartial, TableGroup, notes, metadata).

## Gotchas

- **Composite primary keys are index syntax, not column syntax.**
  `(id, country) [pk]` inside `indexes { }` — there's no way to mark two
  separate columns `[pk]` and have DBML treat them as one composite key.
- **One-to-one FK column order matters.** In long/short-form `Ref`, the
  **second** column listed is the foreign key: `users.id - user_infos.user_id`
  makes `user_infos.user_id` the FK. In inline form, whichever column carries
  `[ref: - ...]` is the FK, regardless of position.
- **`?` on a `Ref` operator marks that side nullable/optional**, e.g.
  `ref: >? users.id` — don't confuse this with `not null`/`null` column
  settings, which control the column itself, not the relationship.
- **`Note:` (attached, inside a `Table`/`Column`/`Index`/`TableGroup` body)
  and standalone `Note name { }` (sticky note, its own canvas element) are
  different things** — attaching a note to an element vs. placing a free
  sticky note. Both take the same string/multi-line-string value.
- **Multi-line strings use triple single quotes** (`'''...'''`), not triple
  double quotes — for `Note`, `description`-style long text, etc. Regular
  string values use single quotes; double quotes are for quoting identifiers
  with spaces (`"column name"`).
- **Expressions (defaults, check constraints, index expressions) use
  backticks**, not quotes: `` default: `now()` ``, `` check: `age > 0` ``.
  Backtick content disables DBML's own value validation — it's passed
  through as-is to the target SQL dialect.
- **`use` is not transitive** — importing a file that itself `use`s a third
  file does not bring that third file's elements along; only `reuse` chains
  across files. See
  [`references/enrichment-and-modules.md`](references/enrichment-and-modules.md#module-system).
- **Column types are unchecked free text.** `varchar(255)`, `decimal(10,2)`,
  and a misspelled `varchr` are all equally "valid" DBML — the parser
  doesn't know SQL types. A type with a space needs double quotes:
  `"double precision"`.
- **`Table` settings vs `Column` settings vs `Index` settings are three
  separate `[ ]` lists** with different allowed keys — e.g. `headercolor`
  is table-level only, `pk`/`unique`/`type` are index-level, `increment` is
  column-level. Don't mix them up when porting from memory of one context.
- If `@dbml/cli` isn't installed and the user hasn't asked you to install
  it, say the source is unvalidated rather than asserting it's correct —
  see [`references/cli-and-tooling.md`](references/cli-and-tooling.md#install).
