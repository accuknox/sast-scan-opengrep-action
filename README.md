# Accuknox SAST

This performs an SAST scan on your repository and uploads the results to AccuKnox's CSPM panel. It helps in identifying security issues and integrates seamlessly with GitHub Actions workflows.

## Features
- Runs Opengrep to analyze the repository.
- Uploads scan results to AccuKnox CSPM panel.
- Supports artifact upload to GitHub.
- Allows soft failure for non-blocking scans.
- Enables AI analysis for intelligent security insights.

## Inputs
| Name | Description | Required | Default |
|------|-------------|----------|---------|
| pipeline_id | GitHub Run ID | Yes | `${{ github.run_id }}` |
| job_url | GitHub Job URL | Yes | `${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}` |
| accuknox_endpoint | CSPM panel URL | Yes | `cspm.demo.accuknox.com` |
| accuknox_token | AccuKnox API Token | Yes |  |
| accuknox_label | Label for scan results | Yes |  |
| accuknox_ai_analysis | Enable AI analysis for scan results | No | `false` |
| anthropic_api_key | Anthropic API key for AI analysis | No |  |
| soft_fail | Continue even if scan fails | No | `false` |

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
        uses: actions/checkout@v3

      - name: "Run Accuknox SAST: Opengrep"
        uses: accuknox/sast-scan-opengrep-action@latest
        with:
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY}}
          accuknox_ai_analysis: "false"
          soft_fail: "true"
```
