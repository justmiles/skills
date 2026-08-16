# DBML CLI and JS tooling

Source of truth: [dbml.dbdiagram.io/cli](https://dbml.dbdiagram.io/cli),
[dbml.dbdiagram.io/database-support](https://dbml.dbdiagram.io/database-support),
[dbml.dbdiagram.io/js-module/core](https://dbml.dbdiagram.io/js-module/core)
(from the [holistics/dbml](https://github.com/holistics/dbml) repo).

## Install

```bash
npm install -g @dbml/cli
```

Requires Node.js ≥ 18 (from `@dbml/cli@3.7.1`).

## Database support matrix

| Database | Import (SQL → DBML) | Export (DBML → SQL) | Connector (`db2dbml`) |
|---|:---:|:---:|:---:|
| PostgreSQL | ✓ | ✓ | ✓ |
| MySQL | ✓ | ✓ | ✓ |
| MSSQL (SQL Server) | ✓ | ✓ | ✓ |
| Oracle | ✓ | ✓ | ✓ |
| Snowflake | ✓ | — | ✓ |
| BigQuery | — | — | ✓ |

## `dbml2sql` — DBML → SQL

```bash
dbml2sql schema.dbml                    # defaults to PostgreSQL
dbml2sql schema.dbml --mysql            # or --postgres / --mssql / --oracle
dbml2sql schema.dbml -o schema.sql      # write to a file instead of stdout

# multi-file project (module system) — pass the entry point, imports resolve automatically
dbml2sql project.dbml --postgres
```

```
dbml2sql <path-to-dbml-file>
         [--mysql|--postgres|--mssql|--oracle]
         [-o|--out-file <output-filepath>]
```

## `sql2dbml` — SQL → DBML

```bash
sql2dbml dump.sql --postgres
sql2dbml --mysql dump.sql -o mydatabase.dbml
```

```
sql2dbml <path-to-sql-file>
         [--mysql|--postgres|--mssql|--postgres-legacy|--mysql-legacy|--mssql-legacy|--snowflake|--oracle]
         [-o|--out-file <output-filepath>]
```

The `*-legacy` variants use the older parser: quicker, less accurate. Prefer
the non-legacy dialect unless it fails on your input.

## `db2dbml` — generate DBML directly from a live database

```bash
db2dbml postgres 'postgresql://dbml_user:dbml_pass@localhost:5432/dbname?schemas=public'
db2dbml postgres '...' -o schema.dbml
```

```
db2dbml postgres|mysql|mssql|snowflake|bigquery|oracle
        <connection-string>
        [-o|--out-file <output-filepath>]
```

Connection string shapes:

| Target | Example |
|---|---|
| postgres | `postgresql://user:password@localhost:5432/dbname?schemas=schema1,schema2` |
| mysql | `mysql://user:password@localhost:3306/dbname` |
| mssql | `Server=localhost,1433;Database=master;User Id=sa;Password=...;Encrypt=true;TrustServerCertificate=true;Schemas=schema1,schema2;` |
| snowflake | `SERVER=<account>.<region>;UID=...;PWD=...;DATABASE=...;WAREHOUSE=...;ROLE=...;SCHEMAS=schema1,schema2;` |
| bigquery | path to a service-account JSON credential file (`{}` for empty = use Application Default Credentials) |
| oracle | `username/password@[//]host[:port][/service_name]` |

Regenerating with `db2dbml` (rather than hand-editing) is the safer path
when the schema's source of truth is a live database — hand-edited DBML
drifts silently otherwise.

## Recommended agent workflow

After writing/editing `.dbml` source, run `dbml2sql <file>.dbml` (or the
dialect flag matching the user's target database) and confirm it produces
SQL without errors before considering the task done. If `@dbml/cli` isn't
installed and the user hasn't asked you to install it, say the source is
unvalidated rather than asserting it's correct — DBML's grammar has enough
sharp edges (composite-PK-via-index only, FK column ordering on one-to-one
refs, backtick-vs-quote rules) that eyeballing isn't reliable.

## `@dbml/core` — JS/TS library

For programmatic parsing/conversion instead of the CLI (e.g. inside a build
script or editor integration):

```bash
npm install @dbml/core
```

```javascript
const { importer, exporter, Parser, ModelExporter } = require('@dbml/core');

// SQL -> DBML
const dbml = importer.import(sqlDDLString, 'postgres');

// DBML -> SQL
const sql = exporter.export(dbmlString, 'mysql');

// Parse to a Database object (single file)
const parser = new Parser();
const database = parser.parse(dbmlString, 'dbmlv2');   // 'dbmlv2' = current parser; 'dbml' is deprecated

// Multi-file project parsing
parser.setDbmlSource('/main.dbml', mainFileContents);
parser.setDbmlSource('/users.dbml', usersFileContents);
const project = parser.parseDbmlProject('/main.dbml');  // throws CompilerError on syntax/binding errors

// Database object -> SQL/DBML/JSON
const out = ModelExporter.export(database, 'postgres');
```

- `format` accepts: `'mysql' | 'mysqlLegacy' | 'postgres' | 'postgresLegacy' | 'dbml' | 'schemarb' | 'mssql' | 'mssqlLegacy' | 'snowflake' | 'json' | 'dbmlv2' | 'oracle'` (import/parse); a smaller set for export (`'mysql' | 'postgres' | 'oracle' | 'dbml' | 'mssql' | 'json'`).
- `ImportOptions`/`ExportOptions.includeRecords` (default `true`) controls
  whether `Records` blocks are emitted in DBML output.
- Use `'dbmlv2'` over `'dbml'` when parsing — `'dbml'` is deprecated and
  missing newer features (records, multifile, table-group notes/colors).

`@dbml/connector` fetches a live database's schema as JSON for
`importer.generateDbml(schemaJson)` — same underlying data `db2dbml` uses.

## Companion products (not part of the open-source CLI/core)

- **[dbdiagram.io](https://dbdiagram.io)** — free web-based ER diagram
  visualizer/editor for DBML.
- **[dbdocs.io](https://dbdocs.io)** — free web-based database documentation
  site generator from DBML source.
- **[runsql.com](https://runsql.com)** — free SQL playground for validating
  queries against a mock schema.

These are hosted products, not packages this skill installs — point the
user at them for visualization/doc-hosting rather than trying to replicate
that locally.
