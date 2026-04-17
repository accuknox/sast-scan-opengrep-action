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
| accuknox_ai_analysis | Enable AI analysis for scan results | No | `true` |
| upload_results | Upload scan results as GitHub artifact | No | `true` |
| soft_fail | Continue even if scan fails | No | `false` |

## Usage Example

```yaml
name: AccuKnox Opengrep SAST

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
    env:
      YOUR_API_KEY: ${{ secrets.YOUR_API_KEY }}  # Required only if using a custom AI model via codeassure.json
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: "Run AccuKnox SAST: Opengrep"
        uses: accuknox/sast-scan-opengrep-action@v1.0.3
        with:
          accuknox_endpoint: "${{ secrets.ACCUKNOX_ENDPOINT }}"
          accuknox_token: "${{ secrets.ACCUKNOX_TOKEN }}"
          accuknox_label: "${{ secrets.ACCUKNOX_LABEL }}"
          soft_fail: "true"
          upload_results: true  # (Optional) set true if you want to upload result to Github artefact
```

> **Note:** The `env` section is only required if you are using a custom AI model via `codeassure.json`. If no custom model is configured, the `env` block can be omitted.

## Custom AI Model Configuration (Optional)

To use a custom AI model for analysis, add a `codeassure.json` file to the root of your repository. Set `provider` based on your model's endpoint:

| Provider | Description |
|----------|-------------|
| `openai` | OpenAI API (e.g., GPT-4) |
| `anthropic` | Anthropic API (e.g., Claude) |
| `google` | Google AI API (e.g., Gemini) |
| `openai-compatible` | Any OpenAI-compatible endpoint (e.g., local models, Azure OpenAI) |

```json
{
  "model": {
    "provider": "openai",
    "name": "your-model-name",
    "api_base": "https://api.openai.com/v1",
    "api_key": "$YOUR_API_KEY"
  }
}
```

Set `api_key` to any environment variable name of your choice (e.g., `$YOUR_API_KEY`). Store the actual key as a GitHub secret and expose it in your workflow's `env` section as shown in the usage example above.
