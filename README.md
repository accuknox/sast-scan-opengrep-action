# Accuknox SAST: Opengrep

This performs an Opengrep SAST scan on your repository and uploads the results to AccuKnox's CSPM panel. It helps in identifying security issues and integrates seamlessly with GitHub Actions workflows.

## Features
- Runs Opengrep to analyze the repository.
- Uploads scan results to AccuKnox CSPM panel.
- Supports artifact upload to GitHub.
- Allows soft failure for non-blocking scans.

## Inputs
| Name | Description | Required | Default |
|------|-------------|----------|---------|
| pipeline_id | GitHub Run ID | Yes | `${{ github.run_id }}` |
| job_url | GitHub Job URL | Yes | `${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}` |
| accuknox_endpoint | CSPM panel URL | Yes | `cspm.demo.accuknox.com` |
| accuknox_tenant | AccuKnox Tenant ID | Yes |  |
| accuknox_token | AccuKnox API Token | Yes |  |
| accuknox_label | Label for scan results | Yes |  |
| input_soft_fail | Continue even if scan fails | No | `false` |
| upload_artifact | Upload scan results as artifact | No | `true` |

## Usage Example
```yaml
name: Accuknox SAST: Opengrep

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
        uses: accuknox/sast-scan-opengrep-action@1.0.0
        with:
          accuknox_endpoint: "cspm.stage.accuknox.com"
          accuknox_tenant: "${{ secrets.STAGE_TENANT_ID }}"
          accuknox_token: "${{ secrets.STAGE_TOKEN }}"
          accuknox_label: "SPOC"
          input_soft_fail: "true"
          upload_artifact: false
```