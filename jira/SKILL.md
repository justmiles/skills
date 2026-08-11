---
name: jira
description: "Manage Jira issues, epics, sprints, and boards from the terminal via the ankitpokhrel/jira-cli (`jira` command). Use when the user wants to list, search, view, create, edit, transition, assign, comment on, or link Jira issues."
---

# Jira CLI

Interact with Jira from the command line using [`jira-cli`](https://github.com/ankitpokhrel/jira-cli)
(the `jira` binary): search and view issues with JQL, create/edit/transition issues,
manage epics and sprints, add comments and worklogs, and open issues in the browser.
Works with both Jira Cloud and on-premise (Server/Data Center) instances.

Reference: https://github.com/ankitpokhrel/jira-cli

## Setup

### Installation

Check if it is installed:

```bash
jira version
```

If not, install with one of:

```bash
# Homebrew (macOS/Linux)
brew tap ankitpokhrel/jira-cli && brew install jira-cli

# Go
go install github.com/ankitpokhrel/jira-cli/cmd/jira@latest

# Scoop (Windows)
scoop bucket add extras && scoop install jira-cli

# Nix
nix-env -f '<nixpkgs>' -iA jira-cli-go

# Docker
docker run -it --rm ghcr.io/ankitpokhrel/jira-cli:latest
```

Or download a prebuilt binary for your platform from the
[releases page](https://github.com/ankitpokhrel/jira-cli/releases).

### Authentication

Get an API token and export it as `JIRA_API_TOKEN` **before** running any command.
Add it to your shell profile (e.g. `~/.bashrc`) so it is always available. `jira-cli`
can also read the token from `.netrc` or the system keychain.

```bash
export JIRA_API_TOKEN="your-token-here"
```

What the token should be depends on the install type:

| Install type           | `JIRA_API_TOKEN` value                                    | Extra                          |
| ---------------------- | --------------------------------------------------------- | ------------------------------ |
| Cloud                  | API token from https://id.atlassian.com/manage/api-tokens | —                              |
| On-premise, basic auth | Your Jira login password                                  | —                              |
| On-premise, PAT        | Personal Access Token from your Jira profile              | `export JIRA_AUTH_TYPE=bearer` |
| Client certificates    | Token (optional, alongside certs)                         | `export JIRA_AUTH_TYPE=mtls`   |

### Initialize config

Run the interactive setup once. It prompts for install type (Cloud / Local), auth
type, server URL, login email, and a default project + board, then writes a config
file to `~/.config/.jira/.config.yml`:

```bash
jira init
```

Override the config path with `--config <path>` or the `JIRA_CONFIG_FILE` env var —
handy for switching between multiple Jira instances.

## Usage

Confirm setup is working with:

```bash
jira me   # prints the current authenticated user
```

### Listing & searching issues

```bash
jira issue list                       # recent issues in the default project
jira issue list -a$(jira me)          # assigned to me
jira issue list -s"In Progress"       # by status
jira issue list --created -7d         # created in the last 7 days
jira issue list -tBug -yHigh          # by type and priority
jira issue list -q "summary ~ cli AND status = Open"   # raw JQL
```

Output formats (great for scripting / piping to `jq`):

- `--plain` — plain text, no interactive TUI
- `--csv` — CSV
- `--raw` — raw JSON from the Jira API

```bash
jira issue list --plain --columns key,status,summary
jira issue list --raw | jq -r '.[].key'
```

### Viewing an issue

```bash
jira issue view ISSUE-1
jira issue view ISSUE-1 --comments 5   # include the 5 most recent comments
```

### Creating issues

```bash
jira issue create                                          # interactive
jira issue create -tBug -s"Login fails" -yHigh -lbug --no-input
jira issue create -tStory -s"New feature" -PEPIC-42        # attach to an epic
jira issue create --template /path/to/template.tmpl        # from a template
```

Common flags: `-t` type, `-s` summary, `-b` body/description, `-y` priority,
`-l` label (repeatable), `-a` assignee, `-P` parent epic, `--no-input` to skip prompts.

### Editing, assigning, transitioning

```bash
jira issue edit ISSUE-1 -s"New summary" -yHigh
jira issue edit ISSUE-1 --label -old --label new    # remove "old", add "new"

jira issue assign ISSUE-1 "Jane Doe"
jira issue assign ISSUE-1 $(jira me)                 # assign to self
jira issue assign ISSUE-1 x                          # unassign

jira issue move ISSUE-1 "In Progress"               # transition status
jira issue move ISSUE-1 Done --comment "Completed"
```

### Comments, worklogs, links

```bash
jira issue comment add ISSUE-1 "Looks good to me"   # Markdown supported
jira issue worklog add ISSUE-1 "2h" --comment "Investigation"

jira issue link ISSUE-1 ISSUE-2 Blocks
jira issue unlink ISSUE-1 ISSUE-2
jira issue link remote ISSUE-1 https://example.com "Related doc"
```

### Cloning & deleting

```bash
jira issue clone ISSUE-1 -s"Copy" -a$(jira me) -H"find:replace"
jira issue delete ISSUE-1 --cascade                 # also delete subtasks
```

### Epics

```bash
jira epic list                    # all epics (interactive)
jira epic list --table            # table view
jira epic list KEY-1              # issues within an epic
jira epic add KEY-1 ISSUE-1 ISSUE-2      # add issues to an epic
jira epic remove ISSUE-1                  # remove an issue from its epic
```

### Sprints

```bash
jira sprint list                  # sprint explorer
jira sprint list --current        # issues in the active sprint
jira sprint list --prev           # previous sprint
jira sprint list SPRINT_ID        # a specific sprint
jira sprint add SPRINT_ID ISSUE-1 ISSUE-2    # add issues to a sprint
```

### Projects, boards, releases, browser

```bash
jira project list
jira board list
jira release list                 # requires releases/versions enabled
jira open                         # open the project in a browser
jira open ISSUE-1                 # open a specific issue
```

## Tips

- Most `create`/`edit` commands are interactive unless you pass `--no-input`.
- Scope any command to a different project or board with `-p PROJECT` / `-b BOARD`.
- For automation, prefer `--plain`/`--csv`/`--raw` and pass explicit flags so no
  TTY prompt blocks the command.

## Troubleshooting

- **`401 Unauthorized`** — Check `JIRA_API_TOKEN` is exported in the current shell.
  For on-premise PAT auth, also set `JIRA_AUTH_TYPE=bearer`.
- **`command not found: jira`** — Ensure the install location (e.g. `$(go env GOPATH)/bin`)
  is on your `PATH`, or reinstall via a package manager.
- **Wrong/instance not found** — Re-run `jira init`, or point `JIRA_CONFIG_FILE` at the
  right config for a multi-instance setup.
- **Custom fields / non-English installs** — Manually set `epic.name`, `epic.link`, and
  `issue.types.*.handle` in `~/.config/.jira/.config.yml`.
