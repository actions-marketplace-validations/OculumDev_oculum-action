# Oculum Security Scanner - GitHub Action

🛡️ AI-native security scanner for LLM-powered applications. Detects prompt injection, RAG vulnerabilities, overpermissive AI tools, and traditional security issues.

## Features

- **AI-Era Security**: Detects vulnerabilities specific to LLM applications (prompt injection, RAG exfiltration, unsafe tool execution)
- **Traditional SAST**: Finds hardcoded secrets, SQL injection, XSS, and other common vulnerabilities
- **Low False Positives**: AI-assisted validation reduces noise (Pro tier)
- **GitHub Integration**: PR comments, check annotations, and SARIF upload to Code Scanning
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
  security-events: write

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Oculum Security Scan
        uses: oculum-security/oculum@v1
        with:
          depth: cheap
          fail-on: high
```

## Usage Examples

### Free Tier (No API Key)

```yaml
- uses: oculum-security/oculum@v1
  with:
    depth: cheap        # Fast pattern matching (free)
    fail-on: high       # Fail on critical/high issues
    comment: true       # Post PR comment
    sarif: true         # Upload to Code Scanning
```

### Pro Tier (With API Key)

```yaml
- uses: oculum-security/oculum@v1
  with:
    depth: validated    # AI-assisted validation
    fail-on: high
    oculum-api-key: ${{ secrets.OCULUM_API_KEY }}
```

### Scan Specific Directory

```yaml
- uses: oculum-security/oculum@v1
  with:
    working-directory: ./src
    depth: cheap
```

### Report Only (Never Fail)

```yaml
- uses: oculum-security/oculum@v1
  with:
    depth: cheap
    fail-on: none       # Never fail the workflow
```

### Use Outputs in Subsequent Steps

```yaml
- uses: oculum-security/oculum@v1
  id: scan
  with:
    depth: cheap

- name: Check results
  run: |
    echo "Total findings: ${{ steps.scan.outputs.findings }}"
    echo "Blocking issues: ${{ steps.scan.outputs.blocking }}"
    echo "Status: ${{ steps.scan.outputs.status }}"
```

### Matrix Build with Multiple Depths

```yaml
jobs:
  scan:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        depth: [cheap, validated]
    steps:
      - uses: actions/checkout@v4
      - uses: oculum-security/oculum@v1
        with:
          depth: ${{ matrix.depth }}
          oculum-api-key: ${{ matrix.depth != 'cheap' && secrets.OCULUM_API_KEY || '' }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `depth` | Scan depth: `cheap` (free), `validated` (AI validation), `deep` (full semantic) | No | `cheap` |
| `fail-on` | Fail threshold: `critical`, `high`, `medium`, `low`, `none` | No | `high` |
| `comment` | Post PR comment with results | No | `true` |
| `sarif` | Upload SARIF to Code Scanning | No | `true` |
| `oculum-api-key` | API key for Pro features | No | - |
| `working-directory` | Directory to scan | No | `.` |

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
| `tier` | Authenticated tier (`free`, `pro`, `enterprise`) |
| `sarif-file` | Path to SARIF file |
| `scan-duration` | Scan duration in milliseconds |
| `files-scanned` | Number of files scanned |

## Scan Depths

### Cheap (Free)

- **Cost**: Free, runs locally
- **Speed**: Fast (~1000 files/sec)
- **Method**: Pattern matching and heuristics
- **Best for**: Quick CI checks, open source projects

### Validated (Pro)

- **Cost**: Requires API key
- **Speed**: Medium
- **Method**: Pattern matching + AI validation
- **Best for**: Production code, reducing false positives

### Deep (Pro)

- **Cost**: Requires API key
- **Speed**: Slower
- **Method**: Full semantic analysis
- **Best for**: Security audits, detailed reports

## Permissions

The action requires these permissions:

```yaml
permissions:
  contents: read          # Read repository files
  pull-requests: write    # Post PR comments
  security-events: write  # Upload SARIF (optional)
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
| `ai_schema_mismatch` | AI output used without validation |

### Traditional Vulnerabilities

| Category | Description |
|----------|-------------|
| `hardcoded_secret` | API keys, passwords in source code |
| `sql_injection` | Unsanitized input in SQL queries |
| `xss` | Cross-site scripting vulnerabilities |
| `command_injection` | Shell command injection |
| `missing_auth` | Endpoints without authentication |
| `data_exposure` | Sensitive data in logs or responses |

## Troubleshooting

### Action fails with "GITHUB_TOKEN is required"

Add the `contents: read` permission to your workflow:

```yaml
permissions:
  contents: read
```

### SARIF upload fails

Ensure you have the `security-events: write` permission:

```yaml
permissions:
  security-events: write
```

Or disable SARIF upload:

```yaml
- uses: oculum-security/oculum@v1
  with:
    sarif: false
```

### "Scan depth requires API key" warning

The `validated` and `deep` depths require a Pro subscription. Either:

1. Get an API key at [oculum.dev/pricing](https://oculum.dev/pricing)
2. Use `depth: cheap` for free scans

### No findings in PR comment

Check that:
1. The workflow is triggered on `pull_request` event
2. `comment: true` is set (default)
3. `pull-requests: write` permission is granted

## Support

- 📖 [Documentation](https://oculum.dev/docs)
- 🐛 [Report Issues](https://github.com/oculum-security/oculum/issues)
- 💬 [Discord Community](https://discord.gg/oculum)
- 📧 [Email Support](mailto:support@oculum.dev)

## License

MIT License - see [LICENSE](LICENSE) for details.
