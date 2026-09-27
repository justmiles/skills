# <project-name>

## Commands

All commands run through devbox (`devbox run <script>`), defined in `devbox.json`:

| Command               | Description                                                                               |
| --------------------- | ----------------------------------------------------------------------------------------- |
| `devbox run build`    | `go build ./...` — compile all packages                                                   |
| `devbox run test`     | `go test ./...` — run the full test suite                                                 |
| `devbox run lint`     | `golangci-lint run ./...` — lint all packages                                             |
| `devbox run run`      | `go run main.go` — run the program                                                        |
| `devbox run coverage` | Run tests with a coverage profile, then enforce thresholds via `go tool go-test-coverage` |
| `devbox run deadcode` | `go tool deadcode ./...` — report unreachable functions                                   |

Run a single test in a specific package:

```bash
go test ./internal/<name>/ -run TestName -v
```

## Architecture

<!-- High-level description of what the program does and how data flows. -->

Package structure:

- `main.go` — wiring only. Reads configuration from the environment, constructs
  and connects components, and starts the program. Contains no business logic.
- `internal/<name>/` — each integration or domain. Each package exports a single
  entry-point function; internals stay unexported. When a package grows past ~2
  files (or ~2 distinct concerns), split it into a sub-package.

## Conventions

- **Config via environment variables only.** No config files. Missing required
  variables exit immediately at startup with a clear error message.
- **120-character line limit.**
- **No comments that restate the code** — comment the _why_, not the _what_.

## Linting

`.golangci.yml` (golangci-lint v2). Treat linter failures as structural signals,
not style nits:

- `funlen` (lines: 80, statements: 60; test files exempt) → split the function
  or extract helpers.
- `maintidx` (under: 20) → the function is too complex; decompose it.
- `revive` file-length-limit (500) → split or subpackage as appropriate
- `lll` (line-length: 120) → wrap or restructure.

Other enabled linters: `errcheck`, `govet`, `staticcheck`, `revive`,
`testifylint`, `thelper`, `tparallel`, `usetesting`. Formatters: `gofmt`,
`goimports`.

## Coverage

`devbox run coverage` writes `coverage.txt` (gitignored) and checks it against
`.testcoverage.yml`. Adjust thresholds there:

```yaml
profile: coverage.txt
threshold:
  file: 80
  package: 70
```

The coverage tool `github.com/vladopajic/go-test-coverage/v2` is pinned as a Go
tool in `go.mod` (`go get -tool`) and invoked with `go tool go-test-coverage`.

## Dead code

`devbox run deadcode` runs `go tool deadcode ./...` (pinned via
`golang.org/x/tools/cmd/deadcode`) to report unreachable functions. Delete what
it flags rather than leaving it, unless it's a deliberately-exported public API.
