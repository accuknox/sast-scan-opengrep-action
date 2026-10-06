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
| pipeline_id | GitHub Run ID | No | `${{ github.run_id }}` |
| job_url | GitHub Job URL | No | `${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}` |
| accuknox_endpoint | CSPM panel URL | Yes |  |
| accuknox_token | AccuKnox API Token | Yes |  |
| accuknox_label | Label for scan results | Yes |  |
| accuknox_ai_analysis | Enable AI analysis (CodeAssure). When `true`, provider, model, and an API key are required | No | `false` |
| codeassure_provider | Model provider when AI is on: `openai`, `openai-compatible`, `anthropic`, `google`, `gemini`. Env `CODEASSURE_PROVIDER` if empty. Use `anthropic` for Anthropic; `openai` or `openai-compatible` for OpenRouter | Required when AI is on |  |
| codeassure_model | Model name when AI is on, for example `claude-sonnet-4-6`. OpenRouter uses its model id, for example `anthropic/claude-sonnet-4.6`. Env `CODEASSURE_MODEL` if empty | Required when AI is on |  |
| codeassure_api_key | Model API key, exported as `CODEASSURE_API_KEY`. Env `CODEASSURE_API_KEY` if empty, then `anthropic_api_key` (provider `anthropic` only). For OpenRouter this is the OpenRouter key | Required when AI is on |  |
| codeassure_api_base | API host. Leave empty for Anthropic (`https://api.anthropic.com`). Set `https://openrouter.ai/api/v1` for OpenRouter, or another non-default endpoint. Env `CODEASSURE_API_BASE` if empty | No |  |
| anthropic_api_key | Anthropic API key, exported as `ANTHROPIC_API_KEY`. Used only when `codeassure_provider` is `anthropic` and `codeassure_api_key` and `CODEASSURE_API_KEY` are empty | No |  |
| soft_fail | Continue even if scan finds issues | No | `true` |
| severity | Comma-separated severities: MEDIUM, HIGH, CRITICAL | No | `HIGH` |
| scanner_version | aspm-scanner-cli GitHub release tag | No | `v0.15.1` |


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
          severity: "HIGH"
```

AI SAST writes a temporary CodeAssure config on the runner from these inputs. Leave `codeassure_api_base` empty for Anthropic. Set it when the model is reached through OpenRouter or another host.

```yaml
      - name: "Run Accuknox SAST: Opengrep"
        uses: accuknox/sast-scan-opengrep-action@latest
        with:
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
          accuknox_ai_analysis: "true"
          codeassure_provider: anthropic
          codeassure_model: claude-sonnet-4-6
          codeassure_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          soft_fail: "true"
          severity: "HIGH"
```
