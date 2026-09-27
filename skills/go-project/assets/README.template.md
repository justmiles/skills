# <project-name>

<!-- One paragraph: what this project does and why it exists. -->

## Prerequisites

- [devbox](https://www.jetify.com/devbox) (provides Go and golangci-lint)
- Required environment variables:
  - `<VAR_NAME>` — <what it is / where to get it>

<!-- List every env var the program reads. Missing required vars cause an
     immediate exit with a clear error. -->

## Quickstart

```bash
# Enter the reproducible environment (installs Go + tooling).
devbox shell

# Configure via environment variables (no config files).
export <VAR_NAME>=<value>

# Run.
devbox run run
```

## Dev commands

| Command | Description |
| --- | --- |
| `devbox run build` | Compile all packages (`go build ./...`) |
| `devbox run test` | Run the test suite (`go test ./...`) |
| `devbox run lint` | Run golangci-lint (`golangci-lint run ./...`) |
| `devbox run run` | Run the program (`go run main.go`) |
| `devbox run coverage` | Run tests with coverage and check thresholds |
