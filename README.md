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
npx skills add justmiles/skills/confluence
```

### jira

Manage Jira issues, epics, sprints, and boards from the terminal via the
[ankitpokhrel/jira-cli](https://github.com/ankitpokhrel/jira-cli) (`jira` command).
Use when the user wants to list, search, view, create, edit, transition, assign,
comment on, or link Jira issues.

```bash
npx skills add justmiles/skills/jira
```

### outline-wiki

Search and manage Outline wiki documents and collections via the `ol` CLI.

```bash
npx skills add justmiles/skills/outline-wiki
```

### plaud

Access your Plaud recordings — browse, search, read transcripts, download audio,
and view AI summaries.

```bash
npx skills add justmiles/skills/plaud
```
