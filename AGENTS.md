# AGENTS.md

This repo is a collection of re-usable agent skills for Claude Code, Cursor,
and other AI coding agents. It is **content, not an application** — there is no
build step, package manager, or test suite. Each skill is a directory of Markdown
(plus optional supporting assets) that agents discover and load.

Skills are distributed with the [`skills`](https://github.com/vercel-labs/skills)
CLI, which copies a skill's files into a project's (or global) skills directory:

```bash
# Add every skill in this repo
npx skills add justmiles/skills

# Add a single skill
npx skills add justmiles/skills/skills/<name>
```

## Layout

```
skills/<name>/SKILL.md        # required: the skill itself
skills/<name>/references/     # optional: deep-reference docs loaded on demand
skills/<name>/examples/       # optional: runnable/reference examples
skills/<name>/assets/         # optional: CSS/JS/fonts/etc. shipped with the skill
```

Current skills: `dbml`, `finops`, `jira`, `likec4`, `one-ui`.

`CLAUDE.md` is a symlink to this file — edit `AGENTS.md`, never `CLAUDE.md`.

## SKILL.md conventions

Every `skills/<name>/SKILL.md` starts with a YAML frontmatter block delimited by
`---`. Required keys:

- `name` — the skill's identifier.
- `description` — one paragraph covering what it does **and when to use it**
  (agents match on this). Use a YAML block scalar (`>-`) for multi-line text.

Optional keys used in this repo: `triggers` (list of activation keywords) and
`metadata` (e.g. `homepage`, `requires`, `spec`).

## Working in this repo

- Keep skills self-contained: reference files inside the skill via relative paths.
- Prefer progressive disclosure — put the essentials in `SKILL.md` and push long
  detail into `references/`.
- **When you add or rename a skill, update `README.md`** so its skill list and
  `npx skills add ...` snippets stay accurate.
- Before committing, `.openhands/pre-commit.sh` validates that every
  `skills/*/SKILL.md` has frontmatter with `name` and `description`.
