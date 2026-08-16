---
name: plaud
description: "Access your Plaud recordings — browse, search, read transcripts, download audio, and view AI summaries"
---

# Plaud CLI

Command-line access to your Plaud recordings: browse and search recordings, read
timestamped transcripts, download audio, and view AI-generated summaries. Wraps
the Plaud developer API (`plaud` command, npm package `@plaud-ai/cli`).

Reference: https://docs.plaud.ai/plaud-mcp-cli/cli

## Setup

Requires Node.js ≥ 20. Run via `npx` — no global install needed:

```bash
npx -y @plaud-ai/cli <command>
```

Then authenticate (opens a browser; tokens are saved to `~/.plaud/tokens.json`
automatically):

```bash
npx -y @plaud-ai/cli login
```

> Throughout this doc, `plaud <command>` is shorthand for
> `npx @plaud-ai/cli <command>`. Run each command that way, or add an alias
> (`alias plaud='npx @plaud-ai/cli'`).

Before running recording commands, confirm the user is signed in. A `401` or
exit code `2` means the session is invalid — run `npx -y @plaud-ai/cli login` again.

## Authentication

| Command        | What it does                                    |
| -------------- | ----------------------------------------------- |
| `plaud login`  | Sign in via browser; tokens saved automatically |
| `plaud logout` | Revoke authorization and sign out               |
| `plaud me`     | Display current account information             |

## Browsing recordings

| Command                  | What it does                           |
| ------------------------ | -------------------------------------- |
| `plaud files`            | List the latest recordings (paginated) |
| `plaud recent`           | Recordings from the last 7 days        |
| `plaud recent --days 30` | Extend the window (any number of days) |
| `plaud today`            | Only today's recordings                |

`plaud files` options:

- `-p, --page <n>` — Page number, 1–1000 (default: 1)
- `-s, --page-size <n>` — Items per page, 10–100 (default: 20)

```bash
plaud files --page 2 --page-size 50
```

Each listed recording shows: `id`, `name`, `created_at` (ISO 8601), and
`duration`.

## Searching

`plaud search <keyword>` runs a case-insensitive, client-side keyword search over
up to 500 recent recordings.

Options:

- `--from <YYYY-MM-DD>` — Range start (inclusive)
- `--to <YYYY-MM-DD>` — Range end (inclusive)
- `--max <n>` — Max results (default: 50)

```bash
plaud search "weekly" --from 2026-04-01 --to 2026-04-30 --max 10
```

## Accessing a recording

Each command takes a recording `id` (get it from `files`, `recent`, or `search`).

| Command                 | Output                                |
| ----------------------- | ------------------------------------- |
| `plaud file <id>`       | Full metadata and availability status |
| `plaud audio <id>`      | A 24-hour download URL for the audio  |
| `plaud transcript <id>` | Timestamped transcript text           |
| `plaud summary <id>`    | AI-generated summary (Markdown)       |

`transcript` and `summary` accept `-o <filename>` to write output to a file:

```bash
plaud transcript abc123 -o meeting-transcript.txt
plaud summary abc123 -o meeting-summary.md
```

`plaud file <id>` returns, in addition to the list fields: `start_at` (ISO 8601),
`serial_number` (device), and boolean availability flags `audio`, `transcript`,
and `summary`. Check these flags before fetching — a transcript or summary may
not be ready for every recording.

## Utility

- `plaud version` — Installed version and build details
- `plaud update` — Check for a newer version and print the upgrade command
- With `npx`, use the latest release explicitly: `npx @plaud-ai/cli@latest <command>`

## Configuration

Optional config file at `~/.plaud/cli.yaml`:

```yaml
api_base: "https://platform.plaud.ai/developer/api"
timeout: 30000 # milliseconds
```

Environment variables override the config file:

- `PLAUD_API_BASE` — API endpoint
- `PLAUD_CLIENT_ID` / `PLAUD_CLI_CLIENT_ID` — OAuth client id
- `PLAUD_CLIENT_SECRET` — OAuth client secret
- `PLAUD_AUTH_URL`, `PLAUD_TOKEN_URL`, `PLAUD_REFRESH_URL` — Auth endpoints

## Exit codes

| Code | Meaning                                   |
| ---- | ----------------------------------------- |
| 0    | Success                                   |
| 1    | Invalid arguments or unknown error        |
| 2    | Authentication failed — run `plaud login` |
| 3    | Network error                             |
| 4    | Request timeout                           |

## Troubleshooting

- **401 / exit code 2** — Run `plaud login`.
- **Token issues** — Delete `~/.plaud/tokens.json` and re-authenticate.
- **Browser login fails** — Manually open the printed URL.
- **`command not found`** — Invoke via `npx @plaud-ai/cli` (or set the alias above).
- **npm 404 / stale package** — `npm cache clean --force`, then retry `npx`.

Remove stored credentials and config:

```bash
rm -rf ~/.plaud
```
