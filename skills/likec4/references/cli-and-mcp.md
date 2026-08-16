# LikeC4 CLI and MCP server

Source of truth: [likec4.dev/tooling](https://likec4.dev/tooling/cli/) and
[likec4.dev/tooling/mcp](https://likec4.dev/tooling/mcp/).

## Install

```bash
# project dev dependency (preferred if the repo already has a package.json)
npm i -D likec4
pnpm add -D likec4
yarn add -D likec4
bun add -d likec4

# or run without installing
npx likec4 <command>
pnpx likec4 <command>
bunx likec4 <command>
```

## Commands

All commands accept `--help` for full flag lists; check that against what's
actually installed, since flags drift between releases.

```bash
# Validate the project's .c4/.likec4 sources — run this before handing off source
likec4 validate

# Check/apply canonical formatting
likec4 format --check
likec4 format          # alias: likec4 fmt

# Local dev server with live preview (hot reload on source change)
likec4 serve           # aliases: likec4 start, likec4 dev
  --port <n> --hmr-port <n> --listen <host> --no-react-hmr --use-dot --public-dir <dir>

# Build a static, browsable site of all views
likec4 build -o ./dist
  --output <dir> --base <path> --use-hash-history --use-dot
  --webcomponent-prefix <prefix> --title <title> --output-single-file

# Preview a build output
likec4 preview -o ./dist

# Export rendered diagrams
likec4 export png -o ./assets
likec4 export jpg -o ./assets --quality 90
likec4 export json -o dump.json
likec4 export drawio -o ./diagrams

# Generate other diagram formats / embeddable code
likec4 gen mermaid     # alias: likec4 gen mmd
likec4 gen dot
likec4 gen d2
likec4 gen plantuml
likec4 gen react

# Language server (used by editor integrations, not usually invoked directly)
likec4 lsp --stdio
likec4 lsp --socket 3000
```

**Recommended agent workflow**: after writing/editing `.c4` source, run
`likec4 validate` and fix any reported errors before considering the task
done. If the CLI isn't installed and the user hasn't asked you to install it,
say the source is unvalidated rather than asserting it's correct.

## MCP server (exposing the model to AI agents)

LikeC4 ships an MCP server so agents can query an existing model
programmatically instead of re-reading raw source files.

### Setup

```bash
# Claude Code
claude mcp add likec4 -- npx -y @likec4/mcp

# Run directly (stdio transport, default)
npx @likec4/mcp

# HTTP transport on a specific port
likec4 mcp -p 1234

# Global install
npm install -g @likec4/mcp
```

The VSCode extension registers this MCP server automatically (via stdio)
whenever the extension is active — no separate setup needed in that editor.

### Available tools (non-exhaustive; 18+ as of this writing)

- `search-element` — find elements by identifier, title, kind, or metadata.
- `find-relationships` — direct and indirect connections between elements.
- `query-graph` — navigate element hierarchy and single-hop associations.
- `find-relationship-paths` — multi-hop relationship chains via BFS.

Use this MCP server (when connected) to answer questions about an *existing*
LikeC4 project's model instead of grepping source — it understands the
merged model, not just one file.

## Editor support

- VSCode extension: `likec4.likec4-vscode` (Marketplace) or the Open VSX
  equivalent — syntax highlighting, live preview, validation-on-save, and
  the bundled MCP server.
