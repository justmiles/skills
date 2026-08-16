# DBML enrichment, module system & data lineage

Source of truth: [dbml.dbdiagram.io/docs](https://dbml.dbdiagram.io/docs)
(condensed from `syntax/enrichment-visualization.md`, `syntax/module-system.md`,
and `syntax/data-lineage.md` of the
[holistics/dbml](https://github.com/holistics/dbml) repo). Everything in
this file is **visualization/documentation-only** (Note, TableGroup,
Metadata, colors, DiagramView) or **project-organization-only** (module
system, data lineage) — none of it has a SQL DDL equivalent, and
`dbml2sql`/`sql2dbml` round-tripping drops or ignores it. For tables,
columns, relationships, enums, see
[`dsl-reference.md`](dsl-reference.md).

## Note Definition

Two forms attach a note to the *element it's declared inside*
(`Table`/`Column`/`Index`/`TableGroup`/`Project`):

```dbml
Table users {
  id int [pk]
  name varchar

  Note: 'This is a note of this table'
  // or, equivalently:
  Note {
    'This is a note of this table'
  }
}
```

- **Project note** — often the schema's long-form Markdown description:
  ```dbml
  Project DBML {
    Note: '''
    # DBML - Database Markup Language
    Simple, readable DSL for database structures.
    '''
  }
  ```
- **Column note**: `status varchar [note: 'replace text here']` — use a
  triple-quoted multi-line string for longer notes.
- **Index note**: `created_at [name: 'created_at_index', note: 'Date']`.
- **TableGroup note**: either `TableGroup g [note: '...'] { ... }` or a
  `Note: '...'` line inside the group body.

## Custom Metadata

Free-form key-value annotations (data classification, SLA, owner, etc.) on
`Table`, `Column`, `TableGroup`, `Note` (sticky notes), and columns inside a
`TablePartial`. Values are string or color literals only.

**Inline** — in the element's `[...]` settings list:

```dbml
Table users [owner: "data-team", sla_hours: "24", pii: "true"] {
  id int [pk, masking: "partial"]
  email varchar [classification: "confidential"]
}
```

A key with no value (`[owner]`) or a duplicate key in the same list
(`[owner: "a", owner: "b"]`) is an error.

**Metadata block** — declared separately, targeting an element by kind +
name (useful for annotating an element defined elsewhere, including in
another file):

```dbml
Metadata Table users {
  owner: 'scott'
  note: 'scott is the owner'
}

Metadata Column users.id {
  pii: 'true'
  masking: 'partial'
}
```

**Precedence** when the same key is set more than once, lowest to highest:
inline settings → metadata blocks in imported files (later import wins) →
metadata blocks in the current file (later block wins).

## Sticky Notes

A standalone note placed on the diagram canvas, not attached to a table —
distinct from the `Note:`/`Note { }` *inside* a table body above.

```dbml
Note single_line_note {
  'This is a single line note'
}

Note multiple_lines_note {
  '''
  This is a multiple lines note
  This string can span over multiple lines.
  '''
}

Note reminder [author: "docs", color: #F4D03F] {
  'Remember to review this schema'
}

// no background — floating text
Note no_color [color: none] {
  'This note has no background color'
}
```

## TableGroup

Visual grouping of related tables (has no SQL meaning — purely for
dbdiagram.io/dbdocs.io layout).

```dbml
TableGroup e_commerce [note: 'e-commerce tables', color: #3498DB] {
  merchants
  countries

  // alternative to the inline `note:` setting above
  Note: 'Contains tables that are related to e-commerce system'
}
```

Settings: `note`, `color` (see [Colors](#colors)), plus free-form custom
metadata (`[team: "growth"]`).

## DiagramView

Named, filtered views of the diagram — pick a subset of tables/notes/table
groups/schemas to display:

```dbml
DiagramView full_view {
  Tables { * }
  Notes { * }
  TableGroups { * }
  Schemas { * }
}

DiagramView sales_view {
  Tables {
    users
    orders
    products
  }
}

DiagramView mixed_view {
  Tables { * }
  Notes { reminder_note }
  TableGroups { group_2 }
  Schemas { core }
}
```

## Colors

Hex only, shorthand or full: `#rgb` or `#rrggbb`.

| Where | Setting |
|---|---|
| Table header | `Table users [headercolor: #3498DB] { ... }` |
| Relationship line | `Ref: a.x > b.y [color: #79AD51]` |
| TableGroup background | `TableGroup g [color: #3498DB] { ... }` |
| Sticky note background | `Note n [color: #F4D03F] { '...' }` (or `color: none` for no background) |

## Inactive Ref

Mark a relationship as inactive (rendered as a dotted line) to document a
relationship that isn't the current focus, without deleting it:

```dbml
Ref: posts.user_id > users.id [inactive]
Ref: posts.user_id > users.id [delete: cascade, inactive]
```

---

## Module system

Splits one schema across multiple `.dbml` files. **Unlike a single-file
schema, files do NOT auto-see each other** — each file only has what it
declares itself plus what it explicitly `use`s.

### Import all

```dbml
// base.dbml
Table users { id int [pk] }
Table orders { id int [pk] }
```

```dbml
// main.dbml — everything from base.dbml becomes available here
use * from './base'

Ref: orders.user_id > users.id
```

The `.dbml` extension in the path is optional (`'./base'` == `'./base.dbml'`).

### Selective import

```dbml
use {
  table auth.users as u
  table auth.roles as r
} from './auth'

use {
  schema auth       // all elements under that schema
} from './auth'

use {
  tablegroup auth_core   // the group + every table in it
} from './auth'
```

Importable `type` keywords (case-insensitive): `table` (brings its records
and refs along), `enum`, `tablepartial`, `note` (sticky note), `schema`,
`tablegroup`.

**Aliasing** with `as` avoids name collisions between files that both
define e.g. `users` — once aliased, only the alias is visible, not the
original name:

```dbml
use { table users as auth_users } from './auth'
use { table users as billing_users } from './billing'
```

### Re-exporting with `reuse`

`use` only makes elements visible in the file that wrote it — a file that
imports *that* file won't see them. `reuse` passes them through:

```dbml
// common/index.dbml
reuse * from './users'
reuse * from './orders'
```

```dbml
// main.dbml — users/orders ARE visible here because common/index used `reuse`
use * from './common/index'
```

Had `common/index.dbml` used plain `use` instead of `reuse`, `main.dbml`
would not see `users`/`orders` — **`use` is not transitive**.

Circular imports between files are fine (DBML is declarative).

---

## Data Lineage (`Dep`)

Documents data *dependency* between tables/columns (e.g. this table is a
dbt model built from those tables) — not a foreign key, and not exported to
SQL.

```dbml
// short form — arrow always points upstream -> downstream
Dep: raw_orders -> stg_orders
Dep: stg_orders <- raw_orders                       // same thing, reversed arrow

Dep: raw_orders.amount -> stg_orders.revenue         // column-level
Dep: stg_orders -> fct_orders.revenue                // table -> column (aggregate)
Dep: raw_events.payload -> stg_events                // column -> table (unpacked JSON)

// inline form
Table fct_orders [dep: <- stg_orders] {
  id int
  revenue decimal [dep: -> report_revenue.total]
}
```

**Dependency block** — group every edge that feeds one transformation step
(e.g. a dbt model), with metadata about the transformation itself:

```dbml
Dep order_staging [color: #79AD51] {
  raw_orders -> stg_orders
  raw_payments -> stg_orders
  raw_orders.amount -> stg_orders.revenue

  note: 'Join orders with payments, compute revenue'
  materialized: table
  query: '''
    Transformation query
  '''
  owner: 'data-team'
}
```

All edges in one block must target the same downstream table (or columns of
it); each directed edge must be unique (reversed pairs/different levels
count as distinct edges).
