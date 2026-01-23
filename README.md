# Oculum Security Scan

AI-native security scanner for LLM-powered applications. Detects prompt injection, hardcoded secrets, SQL injection, XSS, and more.

## Features

- **AI-Era Security**: Detects prompt injection, RAG vulnerabilities, unsafe AI tool execution
- **Traditional SAST**: Finds hardcoded secrets, SQL injection, XSS, command injection
- **Low False Positives**: AI-assisted validation reduces noise (requires API key)
- **GitHub Integration**: PR comments, inline annotations on diffs
- **Free Tier**: Pattern-matching scans run locally at no cost

## Quick Start

Add this workflow to `.github/workflows/security.yml`:

```yaml
name: Security Scan

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read
  pull-requests: write
  checks: write

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Oculum Security Scan
        uses: OculumDev/oculum-action@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          depth: cheap
          fail-on: high
```

## Usage Examples

### Free Tier (No API Key)

```yaml
- name: Run Oculum Security Scan
  uses: OculumDev/oculum-action@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    depth: cheap        # Fast pattern matching (free)
    fail-on: high       # Fail on critical/high issues
```

### With API Key (Validated Scan)

```yaml
- name: Run Oculum Security Scan
  uses: OculumDev/oculum-action@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    depth: validated    # AI-assisted validation
    fail-on: high
    oculum-api-key: ${{ secrets.OCULUM_API_KEY }}
```

### Scan Specific Directory

```yaml
- name: Run Oculum Security Scan
  uses: OculumDev/oculum-action@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    working-directory: ./src
    exclude-patterns: '**/*.test.ts,**/fixtures/**'
```

### Report Only (Never Fail)

```yaml
- name: Run Oculum Security Scan
  uses: OculumDev/oculum-action@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    depth: cheap
    fail-on: none       # Never fail the workflow
```

### Use Outputs in Subsequent Steps

```yaml
- name: Run Oculum Security Scan
  uses: OculumDev/oculum-action@v1
  id: scan
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    depth: cheap

- name: Check results
  run: |
    echo "Total findings: ${{ steps.scan.outputs.findings }}"
    echo "Blocking issues: ${{ steps.scan.outputs.blocking }}"
    echo "Status: ${{ steps.scan.outputs.status }}"
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `depth` | Scan depth: `cheap` (free), `validated` (AI validation) | No | `cheap` |
| `fail-on` | Fail threshold: `critical`, `high`, `medium`, `low`, `none` | No | `high` |
| `fail-on-categories` | Fail only on specific categories (e.g., `ai-*`, `secrets-*`) | No | - |
| `comment` | Post PR comment with results | No | `true` |
| `oculum-api-key` | API key for validated scans | No | - |
| `working-directory` | Directory to scan | No | `.` |
| `include-patterns` | Glob patterns for files to include | No | - |
| `exclude-patterns` | Glob patterns for files to exclude | No | - |

## Outputs

| Output | Description |
|--------|-------------|
| `findings` | Total number of findings |
| `blocking` | Number of blocking issues (critical + high) |
| `critical` | Number of critical findings |
| `high` | Number of high findings |
| `medium` | Number of medium findings |
| `low` | Number of low findings |
| `info` | Number of informational findings |
| `status` | `pass` or `fail` |
| `scan-duration` | Scan duration in milliseconds |
| `files-scanned` | Number of files scanned |

## Scan Depths

### Cheap (Free)

- **Cost**: Free, runs locally
- **Speed**: Fast (~1000 files/sec)
- **Method**: Pattern matching and heuristics
- **Best for**: Quick CI checks, open source projects

### Validated (Requires API Key)

- **Cost**: Requires API key
- **Speed**: Medium
- **Method**: Pattern matching + AI validation
- **Best for**: Production code, reducing false positives

## Permissions

The action requires these permissions:

```yaml
permissions:
  contents: read          # Read repository files
  pull-requests: write    # Post PR comments
  checks: write           # Create check run annotations
```

## Vulnerability Categories

### AI-Era Vulnerabilities

| Category | Description |
|----------|-------------|
| `ai_prompt_injection` | User input in AI prompts without sanitization |
| `ai_unsafe_execution` | AI output used in code execution or SQL |
| `ai_overpermissive_tool` | AI tools with excessive permissions |
| `ai_rag_exfiltration` | RAG queries exposing cross-tenant data |
| `ai_endpoint_unprotected` | AI endpoints without auth/rate limiting |

### Traditional Vulnerabilities

| Category | Description |
|----------|-------------|
| `hardcoded_secret` | API keys, passwords in source code |
| `sql_injection` | Unsanitized input in SQL queries |
| `xss` | Cross-site scripting vulnerabilities |
| `command_injection` | Shell command injection |
| `missing_auth` | Endpoints without authentication |

## Troubleshooting

### PR comments not appearing

Check that:
1. `GITHUB_TOKEN` is passed via `env`
2. `pull-requests: write` permission is granted
3. Workflow is triggered on `pull_request` event

### "Scan depth requires API key" error

The `validated` depth requires an API key. Either:
1. Get an API key at [oculum.dev](https://oculum.dev)
2. Use `depth: cheap` for free scans

## Support

- [Documentation](https://oculum.dev/docs)
- [Report Issues](https://github.com/OculumDev/oculum-action/issues)

## License

MIT License - see [LICENSE](LICENSE) for details.
