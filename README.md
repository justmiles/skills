# skills

A collection of [Agent Skills](https://github.com/vercel-labs/skills) for use with
Claude Code, Cursor, and other AI coding agents.

Skills are installed with the [`skills`](https://github.com/vercel-labs/skills) CLI,
which copies a skill's files into your project (or global) skills directory so your
agent can discover and use them.

## Usage

Add every skill in this repo:

```bash
npx skills add justmiles/skills
```

Or add an individual skill by appending its directory name (see below).

## Skills

### confluence

Upload markdown documents to Confluence Cloud using the `markdown2confluence` CLI.
Use when the user wants to publish, upload, sync, or push markdown files to
Confluence, or mentions Confluence documentation publishing.

```bash
npx skills add justmiles/skills --skill confluence
```

### dbml

Author, edit, and validate database schemas as code using DBML (Database Markup
Language) — a database-agnostic DSL for describing tables, columns, relationships,
indexes, enums, and partials. Use when modeling a schema/ERD in DBML, converting
between DBML and SQL, or working with the `dbml2sql`/`sql2dbml`/`db2dbml` CLI.

```bash
npx skills add justmiles/skills --skill dbml
```

### go-project

Initialize a new Go project, or bring an existing one into line with a standard
tooling baseline: a devbox environment (Go + golangci-lint) with
build/test/lint/run/coverage scripts, a golangci-lint v2 config, go-test-coverage
thresholds, and README/CLAUDE.md scaffolding. Use when scaffolding, bootstrapping,
or standardizing a Golang project, or asserting that an existing repo follows the
conventions.

```bash
npx skills add justmiles/skills --skill go-project
```

### jira

Manage Jira issues, epics, sprints, and boards from the terminal via the
[ankitpokhrel/jira-cli](https://github.com/ankitpokhrel/jira-cli) (`jira` command).
Use when the user wants to list, search, view, create, edit, transition, assign,
comment on, or link Jira issues.

```bash
npx skills add justmiles/skills --skill jira
```

### likec4

Author, edit, and validate software architecture diagrams as code using the LikeC4
DSL — the C4 model (context, container, component, deployment) expressed as text.
Use when creating or updating a C4 architecture diagram, or working with the
`likec4` CLI or the LikeC4 MCP server.

```bash
npx skills add justmiles/skills --skill likec4
```

### outline-wiki

Search and manage Outline wiki documents and collections via the `ol` CLI.

```bash
npx skills add justmiles/skills --skill outline-wiki
```

### plaud

Access your Plaud recordings — browse, search, read transcripts, download audio,
and view AI summaries.

```bash
npx skills add justmiles/skills --skill plaud
```
