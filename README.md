# Canary GitHub Action

Scan your dependencies for vulnerabilities in your CI/CD pipeline.

## Usage

```yaml
name: Security Scan

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Scan dependencies
        uses: roselabs-io/canary-action@v1
        with:
          api-key: ${{ secrets.CANARY_API_KEY }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `api-key` | Canary API key (`sk_canary_...`) | Yes | - |
| `api-url` | Canary API URL | No | `https://canary-api.roselabs.io` |
| `fail-on` | Fail on severity: `critical`, `high`, `medium`, `low`, or `none` | No | `high` |
| `lockfiles` | Comma-separated lockfile paths (auto-detected if empty) | No | Auto-detect |

## Outputs

| Output | Description |
|--------|-------------|
| `vulnerabilities` | Total number of vulnerabilities found |
| `critical` | Number of critical severity vulnerabilities |
| `high` | Number of high severity vulnerabilities |
| `medium` | Number of medium severity vulnerabilities |
| `low` | Number of low severity vulnerabilities |
| `scan-failed` | Whether scan failed due to vulnerabilities (`true`/`false`) |

## Supported Lockfiles

The action auto-detects these lockfiles in your repository root:

| Ecosystem | Lockfiles |
|-----------|-----------|
| npm | `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` |
| Python | `requirements.txt`, `Pipfile.lock`, `poetry.lock`, `uv.lock` |
| Go | `go.sum` |
| Rust | `Cargo.lock` |

## Examples

### Fail only on critical vulnerabilities

```yaml
- uses: roselabs-io/canary-action@v1
  with:
    api-key: ${{ secrets.CANARY_API_KEY }}
    fail-on: critical
```

### Never fail (report only)

```yaml
- uses: roselabs-io/canary-action@v1
  with:
    api-key: ${{ secrets.CANARY_API_KEY }}
    fail-on: none
```

### Scan specific lockfiles

```yaml
- uses: roselabs-io/canary-action@v1
  with:
    api-key: ${{ secrets.CANARY_API_KEY }}
    lockfiles: 'backend/requirements.txt, frontend/package-lock.json'
```

### Use outputs in subsequent steps

```yaml
- uses: roselabs-io/canary-action@v1
  id: canary
  with:
    api-key: ${{ secrets.CANARY_API_KEY }}
    fail-on: none

- name: Comment on PR
  if: steps.canary.outputs.vulnerabilities > 0
  run: |
    echo "Found ${{ steps.canary.outputs.vulnerabilities }} vulnerabilities"
    echo "Critical: ${{ steps.canary.outputs.critical }}"
    echo "High: ${{ steps.canary.outputs.high }}"
```

## Graceful API Failures

If the Canary API is unavailable or returns an error, the action will:
- Log a warning
- Skip that lockfile
- Continue scanning other lockfiles
- **Not fail the CI pipeline** (unless vulnerabilities were found in successfully scanned files)

This ensures your CI doesn't break due to temporary API issues.

## Links

- [Canary Dashboard](https://canary.roselabs.io)
- [Get API Key](https://canary.roselabs.io/settings/api-keys)
