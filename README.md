# Accuknox SAST

This performs an SAST scan on your repository and uploads the results to AccuKnox's CSPM panel.

**Current action (new tags / `main`):** OpenGrep runs as a **local binary** (`tool install --type sast`). No Docker image, no `--container-mode`.

**Older action tags (e.g. `v1.0.6`):** still use the OpenGrep **container image**. Pin those tags if you need the old behavior.

## Inputs
| Name | Description | Required | Default |
|------|-------------|----------|---------|
| pipeline_id | GitHub Run ID | No | `${{ github.run_id }}` |
| job_url | GitHub Job URL | No | this Actions run URL |
| accuknox_endpoint | CSPM panel URL | Yes |  |
| accuknox_token | AccuKnox API Token | Yes |  |
| accuknox_label | Label for scan results | Yes |  |
| accuknox_ai_analysis | Enable AI analysis | No | `false` |
| anthropic_api_key | Exported as `ANTHROPIC_API_KEY` if set | No |  |
| soft_fail | Continue even if scan finds issues | No | `true` |
| severity | Severities that fail the job when `soft_fail` is false | No | `HIGH` |
| scanner_version | aspm-scanner-cli release tag | No | `v0.15.2-rc.3` |

## Usage Example
```yaml
name: Accuknox SAST

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  accuknox-cicd:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: "Run Accuknox SAST: Opengrep"
        uses: accuknox/sast-scan-opengrep-action@latest
        with:
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
          accuknox_ai_analysis: "false"
          soft_fail: "true"
          severity: "HIGH"
          # scanner_version: v0.15.1   # optional; default is v0.15.2-rc.3
```

Runners: **Linux x86_64** (`ubuntu-latest`) and **macOS arm64**.

The job **uploads** to AccuKnox (does not use `--skip-upload`).
