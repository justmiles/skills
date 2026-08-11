---
name: confluence
description: Upload markdown documents to Confluence Cloud using markdown2confluence CLI. Use when the user wants to publish, upload, sync, or push markdown files to Confluence, or mentions Confluence documentation publishing.
---

# Confluence Markdown Upload

Upload markdown documents to Confluence Cloud using the `markdown2confluence` CLI.

## Prerequisites

### Installation

Check if installed:

```bash
markdown2confluence --version
```

If not installed, download for Linux:

```bash
curl -LO https://github.com/justmiles/go-markdown2confluence/releases/download/v3.4.6/go-markdown2confluence_v3.4.6_linux_x86_64.tar.gz
sudo tar -xzvf go-markdown2confluence_v3.4.6_linux_x86_64.tar.gz -C /usr/local/bin/ markdown2confluence
```

For macOS:

```bash
curl -LO https://github.com/justmiles/go-markdown2confluence/releases/download/v3.4.6/go-markdown2confluence_v3.4.6_darwin_x86_64.tar.gz
sudo tar -xzvf go-markdown2confluence_v3.4.6_darwin_x86_64.tar.gz -C /usr/local/bin/ markdown2confluence
```

### Authentication

Set environment variables for authentication:

```bash
export CONFLUENCE_ENDPOINT="https://yourcompany.atlassian.net/wiki"
export CONFLUENCE_USERNAME="your-email@company.com"
export CONFLUENCE_PASSWORD="your-api-token"
```

**Get an API token**: https://id.atlassian.com/manage/api-tokens

Alternative: Use Personal Access Token (useful for SSO):

```bash
export CONFLUENCE_ACCESS_TOKEN="your-access-token"
```

## Usage

### Upload a single file

```bash
markdown2confluence --space 'SPACE_KEY' path/to/file.md
```

### Upload a directory

```bash
markdown2confluence --space 'SPACE_KEY' path/to/docs/
```

### Upload under a parent page

```bash
markdown2confluence \
  --space 'SPACE_KEY' \
  --parent 'Parent Page Title' \
  path/to/docs/
```

### Upload with document title from markdown header

```bash
markdown2confluence \
  --space 'SPACE_KEY' \
  --use-document-title \
  path/to/docs/
```

### Upload only recently modified files

Useful for recurring syncs:

```bash
markdown2confluence \
  --space 'SPACE_KEY' \
  --modified-since 30 \
  path/to/docs/
```

### Exclude patterns

```bash
markdown2confluence \
  --space 'SPACE_KEY' \
  --exclude '.*draft.*' \
  --exclude '.*temp.md' \
  path/to/docs/
```

## CLI Reference

| Flag | Description |
|------|-------------|
| `-s, --space` | Confluence space key (required) |
| `--parent` | Parent page title for nesting |
| `-t, --title` | Override page title |
| `--use-document-title` | Use `# Title` from markdown as page title |
| `-m, --modified-since N` | Only upload files modified in last N minutes |
| `-x, --exclude PATTERN` | Exclude files matching regex pattern |
| `-c, --comment` | Add comment to page |
| `-d, --debug` | Enable debug logging |
| `-w, --hardwraps` | Render newlines as `<br />` |

## Confluence Macros

Insert Confluence macros using fenced code blocks with `CONFLUENCE-MACRO`:

```markdown
# My Page

\`\`\`CONFLUENCE-MACRO
name:toc
schema-version:1
  minLevel:2
\`\`\`

## Section 1
```

Macro syntax:

```
name:macro-name
schema-version:1
  parameter-name:value
  another-param:value
```

## Workflow

1. **Verify authentication** - Check env vars are set
2. **Identify space key** - Find from Confluence URL (e.g., `/wiki/spaces/MYSPACE/`)
3. **Identify parent page** (optional) - Page title to nest content under
4. **Run upload** - Execute markdown2confluence command
5. **Verify in Confluence** - Check pages were created/updated

## Troubleshooting

**401 Unauthorized**: Check `CONFLUENCE_USERNAME` and `CONFLUENCE_PASSWORD` (API token)

**404 Not Found**: Verify space key exists and you have access

**Parent page not found**: Ensure parent page title matches exactly (case-sensitive)

**SSL errors**: Use `--insecuretls` for self-signed certificates

## Docker Alternative

```bash
docker run -e CONFLUENCE_ENDPOINT -e CONFLUENCE_USERNAME -e CONFLUENCE_PASSWORD \
  -v $(pwd):/docs justmiles/markdown2confluence \
  --space 'SPACE_KEY' /docs/
```
