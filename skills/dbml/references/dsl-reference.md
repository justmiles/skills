# DBML structural reference

Source of truth: [dbml.dbdiagram.io/docs](https://dbml.dbdiagram.io/docs)
(condensed from the `docs.md` and `syntax/language-basics.md` pages of the
[holistics/dbml](https://github.com/holistics/dbml) repo — verify against
the live docs if behavior here seems off). This doc covers the constructs
that map directly to SQL DDL. For notes/table groups/metadata/colors/module
system/data-lineage (visualization-only, no SQL equivalent), see
[`enrichment-and-modules.md`](enrichment-and-modules.md).

## Language basics

- **Comments**: `// single line` and `/* multi-line */`.
- **Multi-line strings**: triple single quotes, `'''...'''`. Indentation is
  normalized to the minimum leading-space count across all lines; `\` is a
  line continuation, `\\` and `\'` escape backslash/quote.
- **Brackets**: `{}` group indexes/constraints/table/enum/etc. bodies; `[]`
  holds settings; `'string'` is a string value; `"identifier with spaces"`
  quotes a name; `` `expression` `` is a raw/function expression (disables
  DBML's own value validation, passed through to SQL as-is).

## Project Definition

```dbml
Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Description of the project'
}
```

## Schema Definition

A schema exists implicitly the moment any `Table`/`Enum`/`Ref` references
it via `schema_name.table_name`. Omitting the schema prefix defaults to the
`public` schema.

```dbml
Table core.user {
  id integer [pk]
}
```

## Table Definition

```dbml
// belongs to the default "public" schema
Table table_name {
  column_name column_type [column_settings]
}

// belongs to a named schema
Table schema_name.table_name {
  column_name column_type [column_settings]
}
```

- `column_type` supports any type text; spaces need double quotes
  (`"double precision"`); parenthesized types (`varchar(255)`,
  `decimal(10,2)`) are passed through as-is.
- Table-level custom metadata: `Table users [owner: "data-team"]` (see
  [Custom Metadata](enrichment-and-modules.md#custom-metadata)).

### Table Alias

```dbml
Table very_long_user_table as U {
  id integer [pk]
}

Ref: U.id < posts.user_id
```

## Column Definition

### Column Settings

```dbml
Table buildings {
  address varchar(255) [unique, not null, note: 'to include unit number']
  id integer [pk, unique, default: 123, note: 'Number']
}
```

| Setting | Meaning |
|---|---|
| `primary key` / `pk` | mark column as (part of a) primary key — for a **composite** PK use `indexes { (a, b) [pk] }` instead, not two `[pk]` columns |
| `null` / `not null` | nullability; defaults to nullable if omitted |
| `unique` | unique constraint |
| `default: value` | see [Default Value](#default-value) below |
| `increment` | auto-increment |
| `` check: `expr` `` | backtick expression; repeatable for multiple checks; for checks spanning multiple columns use a table-level [Check Definition](#check-definition) |
| `note: 'string'` | metadata note (enrichment-only, no SQL effect) |

Unsupported settings can be smuggled into the type text itself:
`id "bigint unsigned" [pk]`.

### Default Value

```dbml
Table users {
  id integer [primary key]
  username varchar(255) [not null, unique]
  source varchar(255) [default: 'direct']     // string
  created_at timestamp [default: `now()`]      // expression
  rating integer [default: 10]                 // number
  is_active boolean [default: true]            // boolean / null
}
```

- Number: `default: 123` or `default: 123.456`
- String: `default: 'some string'`
- Expression: `` default: `now() - interval '5 days'` `` (backtick-wrapped)
- Boolean/null: `default: true` / `default: false` / `default: null`

## Check Definition

For constraints spanning multiple columns (single-column checks can use the
column-level `check:` setting instead):

```dbml
Table users {
  id integer
  wealth integer
  debt integer

  checks {
    `debt + wealth >= 0` [name: 'chk_positive_money']
  }
}
```

## Index Definition

```dbml
Table bookings {
  id integer
  country varchar
  booking_date date
  created_at timestamp

  indexes {
    (id, country) [pk]                          // composite primary key
    created_at [name: 'created_at_index', note: 'Date']
    booking_date                                 // single-column index
    (country, booking_date) [unique]             // composite index
    booking_date [type: hash]
    (`id*2`)                                      // expression index
    (`id*3`, `getdate()`)
    (`id*3`, id)                                  // composite w/ expression
  }
}
```

Index settings: `type` (`btree` or `hash`), `name`, `unique`, `pk`, `note`.

## Relationships & Foreign Key Definitions

```dbml
Table posts {
  id integer [primary key]
  user_id integer [ref: > users.id]   // many-to-one, inline
}
```

| Operator | Meaning | Example |
|---|---|---|
| `<` | one-to-many | `users.id < posts.user_id` |
| `>` | many-to-one | `posts.user_id > users.id` |
| `-` | one-to-one | `users.id - user_infos.user_id` |
| `<>` | many-to-many | `authors.id <> books.id` |

Append `?` to either side of the operator to make that side **optional**
(nullable FK, rows need not match) — `>?` means the referenced side is
optional, `?>` means the source side is optional. Works with every operator.

```dbml
Table posts {
  id int [pk]
  user_id int [ref: >? users.id]   // post may have no user
}
```

**One-to-one FK ordering**: with long/short form, the **second** listed
column is the foreign key (`users.id - user_infos.user_id` → FK is
`user_infos.user_id`). With inline form, whichever column carries the `ref:`
setting is the FK, e.g.:

```dbml
Table user_infos {
  user_id integer [ref: - users.id]   // user_infos.user_id is the FK
}
```

### Three ways to write a `Ref`

```dbml
// Long form — needed for relationship settings on multiple refs, or a name
Ref name_optional {
  schema1.table1.column1 < schema2.table2.column2
}

// Short form
Ref name_optional: schema1.table1.column1 < schema2.table2.column2

// Inline form — no name, no relationship settings
Table schema2.table2 {
  id integer
  column2 integer [ref: > schema1.table1.column1]
}
```

If `schema_name` is omitted anywhere it defaults to `public`.

### Composite foreign keys

```dbml
Ref: merchant_periods.(merchant_id, country_code) > merchants.(id, country_code)
```

### Cross-schema relationship

```dbml
Table core.users {
  id integer [pk]
}

Table blogging.posts {
  id integer [pk]
  user_id integer [ref: > core.users.id]
}
// equivalent standalone form: Ref: blogging.posts.user_id > core.users.id
```

### Relationship Settings

```dbml
// short form
Ref: products.merchant_id > merchants.id [delete: cascade, update: no action]

// long form
Ref {
  products.merchant_id > merchants.id [delete: cascade, update: no action]
}
```

`delete:`/`update:` accept `cascade`, `restrict`, `set null`, `set default`,
`no action`. **Relationship settings and names are not supported on inline
form** — use short/long form if you need them. For the `color` setting, see
[Colors](enrichment-and-modules.md#colors).

### Many-to-many

Two ways to model it: a single `<>` relationship, or two `>`/`<`
relationships through a join table (physical-design/SQL-export behavior
differs between the two — prefer the join-table form when you need the SQL
export to actually create that table).

## Enum Definition

```dbml
enum job_status {
  created [note: 'Waiting to be processed']
  running
  done
  failure
}

enum v2.job_status { /* ... */ }   // schema-qualified

Table jobs {
  id integer
  status job_status
  status_v2 v2.job_status
}
```

Values with spaces/special characters need double quotes:

```dbml
enum grade {
  "A+"
  "A"
  "A-"
  "Not Yet Set"
}
```

## TablePartial

Reusable field/setting/index sets, injected into tables with a `~` prefix.

```dbml
TablePartial base_template [headerColor: #ff0000] {
  id int [pk, not null]
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]
}

TablePartial soft_delete_template {
  delete_status boolean [not null]
  deleted_at timestamp [default: `now()`]
}

Table users {
  ~base_template
  name varchar
  ~soft_delete_template
}
```

**Conflict resolution** when the same field/setting/index appears in more
than one place: (1) a local definition in the table itself always wins over
any partial; (2) between partials, the **last-injected** one (source order)
wins.

## Data Sample (Records)

Sample rows, for documentation/testing — has no bearing on table structure.

```dbml
Table users {
  id int [pk]
  name varchar
  email varchar
}

// outside the table, explicit columns
records users(id, name, email) {
  1, 'Alice', 'alice@example.com'
  2, 'Bob', 'bob@example.com'
}

Table comments {
  id int [pk]
  user_id int [ref: > users.id]
  title string

  // inside the table, implicit column list = all columns in declared order
  // (implicit lists are ONLY valid for records defined inside a table)
  records {
    1, 2, 'First comment'
  }
}
```

Each table may have at most one `records` block (inside or outside — not
both). Value syntax (CSV-style, type-checked against the column's SQL type):

| Type | Syntax |
|---|---|
| String | `'text'`, escape `'` as `\'` |
| Number | `42`, `3.14`, `-100`, `1.5e10` |
| Boolean | case-insensitive: `true`/`false`, `'Y'`/`'N'`, `'T'`/`'F'`, `1`/`0` |
| Null | `null`, `''` (non-string columns), or an empty field (`, ,`) |
| Timestamp/date | quoted, ISO 8601 or similar: `'2024-01-15 10:30:00'`, `'2024-01-15'` |
| Enum | `EnumName.value` or a matching string literal |
| Expression | backtick-wrapped, disables type checking: `` `now()` ``, `` `uuid_generate_v4()` `` |
