---
name: go-project
description: >-
  Initialize a new Go project, or bring an existing one into line with a
  standard set of tooling and conventions: a devbox environment (Go +
  golangci-lint) with build/test/lint/run/coverage scripts, golangci-lint v2
  config, go-test-coverage thresholds, and README/CLAUDE.md scaffolding. Use
  whenever someone wants to scaffold, bootstrap, or standardize a Golang
  project, set up devbox/golangci-lint/coverage tooling for Go, or "assert" /
  audit that an existing Go repo follows these conventions.
triggers:
  - go project
  - golang project
  - init go project
  - bootstrap go project
  - scaffold go
  - devbox go
  - golangci-lint setup
  - go coverage
  - go-test-coverage
  - go conventions
metadata:
  requires: devbox, Go, golangci-lint (v2), vladopajic/go-test-coverage/v2, x/tools deadcode
---

# Go Project Setup

Set up or assert a standard tooling baseline and set of conventions for a Go
project. Works in two modes:

- **Init** — the directory is empty or has no Go module. Scaffold everything.
- **Assert** — an existing project. Compare each file against the expected
  baseline, report drift, and update files to match (preserving project-specific
  content like the module path, README prose, and architecture notes).

Always work in **assert** mode against existing files: read what is there first,
diff against the templates in `assets/`, and only change what is missing or
wrong. Never clobber a project's real README description or CLAUDE.md
architecture section - merge into them.

## Workflow

1. Detect state: is there a `go.mod`? A `devbox.json`? A `.golangci.yml`?
2. If no `go.mod`, initialize the module (ask for / infer the module path):
   ```bash
   go mod init <module-path>
   ```
3. Create or reconcile each file below. The canonical contents live in
   [`assets/`](assets/) - copy them directly for init, or diff-and-patch for
   assert.
4. Pin the tools and tidy:
   ```bash
   go get -tool github.com/vladopajic/go-test-coverage/v2
   go get -tool golang.org/x/tools/cmd/deadcode
   go mod tidy
   ```
5. Verify the toolchain: `devbox run build`, `devbox run lint`, `devbox run test`.
6. Report what was created vs. already-compliant vs. changed.

## Files

Copy these from [`assets/`](assets/). They are the source of truth.

| File | Purpose | Template |
| --- | --- | --- |
| `devbox.json` | Reproducible env + task scripts | [assets/devbox.json](assets/devbox.json) |
| `.golangci.yml` | golangci-lint v2 config | [assets/.golangci.yml](assets/.golangci.yml) |
| `.testcoverage.yml` | Coverage thresholds | [assets/.testcoverage.yml](assets/.testcoverage.yml) |
| `.gitignore` | Ignore `coverage.txt` (append, don't replace) | [assets/gitignore](assets/gitignore) |
| `README.md` | Project docs (fill in real content) | [assets/README.template.md](assets/README.template.md) |
| `CLAUDE.md` | Agent instructions | [assets/CLAUDE.template.md](assets/CLAUDE.template.md) |

### devbox.json

Packages: `go@latest`, `golangci-lint@latest`. Scripts:

| Script | Command |
| --- | --- |
| `build` | `go build ./...` |
| `test` | `go test ./...` |
| `lint` | `golangci-lint run ./...` |
| `run` | `go run main.go` |
| `coverage` | `go test -cover -coverprofile=coverage.txt ./...` then `go tool go-test-coverage --config .testcoverage.yml` |
| `deadcode` | `go tool deadcode ./...` — report unreachable functions |

### .golangci.yml (v2 format)

Enabled linters: `errcheck`, `govet`, `revive`, `staticcheck`, `maintidx`
(`under: 20`), `funlen` (`lines: 80`, `statements: 60`, excluded from
`_test.go`), `lll` (`line-length: 120`), `testifylint`, `thelper`, `tparallel`,
`usetesting`. `revive` carries a `file-length-limit` rule of `500`. Formatters:
`gofmt`, `goimports`.

### Coverage

Pin `github.com/vladopajic/go-test-coverage/v2` as a Go tool (`go get -tool`),
so it appears in `go.mod` and runs via `go tool go-test-coverage`. Thresholds in
`.testcoverage.yml`: `file: 80`, `package: 70`, profile `coverage.txt`. Add
`coverage.txt` to `.gitignore`.

### Dead code

Pin `golang.org/x/tools/cmd/deadcode` as a Go tool too, and run
`devbox run deadcode` (`go tool deadcode ./...`) to surface unreachable
functions. Treat its output as a cleanup prompt: delete dead code rather than
leaving it, unless it's a deliberately-exported public API.

## Conventions

Bake these into `CLAUDE.md` and follow them when writing code in the project:

- **Config via environment variables only.** No config files. A missing required
  var exits immediately with a clear error at startup.
- **`main.go` is wiring only** - it assembles dependencies and contains no
  business logic.
- **Each integration or domain lives in `internal/<name>/`** and exports a
  single entry-point function.
- **120-char line limit.** No comments that merely restate the code.
- **Split a package once it grows beyond ~2 files** (or ~2 distinct concerns)
  into a sub-package.

### Linter failures are structural signals, not style nits

- `funlen` fires → split the function or extract helpers.
- `maintidx` fires → the function is doing too much; decompose it.
- `revive` file-length-limit fires (or a file passes ~2 concerns) → promote it
  to a sub-package.

Treat these as a prompt to restructure, not to bump the threshold.
