# Accuknox SAST

This performs an SAST scan on your repository and uploads the results to AccuKnox's CSPM panel. It helps in identifying security issues and integrates seamlessly with GitHub Actions workflows.

## Features
- Runs Opengrep to analyze the repository.
- Uploads scan results to AccuKnox CSPM panel.
- Allows soft failure for non-blocking scans.
- Enables AI analysis for intelligent security insights.
- Quality gates: the policy configured in AccuKnox passes, warns or fails the build.

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
| soft_fail | Continue even if scan finds issues. With `quality_gate`: continue even if the policy fails | No | `true` |
| severity | Comma-separated severities: MEDIUM, HIGH, CRITICAL. With `quality_gate`, the policy decides pass/fail instead | No | `HIGH` |
| aiscan_severity | Comma-separated severities to send for AI analysis, e.g. `HIGH,CRITICAL`. Used only when AI is on. Empty = all findings | No |  |
| quality_gate | Let the AccuKnox quality-gate policy pass, warn or fail the build. See [Quality gate](#quality-gate) | No | `false` |
| gate_scanners | Every scan type uploaded to this gate run: `SG` (SAST), `SQ` (SonarQube SAST), `TR` (SCA/container), `IAC`, `TruffleHog` or `DS` (secrets), `SBOM`. Case-sensitive. `SG` is always included. Empty = env `ACCUKNOX_GATE_SCANNERS`, else `SG` | No |  |
| gate_wait | Wait for the verdict in this step. Set `false` when a later step or job waits with `gate_only` | No | `true` |
| gate_only | Skip the scan and only wait for the verdict of scans already uploaded to the gate run. `accuknox_label` is not needed | No | `false` |
| gate_cli_id | Gate run id. A UUID is used as is (share it across jobs); any other text, such as `${{ matrix.os }}`, gives a separate run id for this workflow run attempt. Empty = env `ACCUKNOX_CLI_ID`, else derived from the workflow run | No |  |
| gate_token | User token for the quality-gate API. Empty = `accuknox_token` | No |  |
| gate_timeout | Seconds to wait for the verdict | No | `300` |
| gate_on_timeout | `fail` or `pass` when the verdict does not arrive in time | No | `fail` |
| scanner_version | aspm-scanner-cli GitHub release tag | No | `v0.16.5` |

## Outputs
| Name | Description |
|------|-------------|
| cli_id | Gate run id the scan was uploaded under (or, with `gate_only`, waited on). Pass it to a later `gate_only` job as `gate_cli_id` |
| gate_status | Set when the step waited: `passed`, `warning`, `failed`, `error`, `timeout`, `unsupported` or `rejected` |
| gate_json_file | Set when the step waited: path to the full quality gate response (JSON) |


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
          aiscan_severity: "HIGH,CRITICAL"
          soft_fail: "true"
          severity: "HIGH"
```

`severity` decides which findings fail the scan; `aiscan_severity` decides which findings are sent for AI analysis (empty = all). Values are not case-sensitive.

For other providers, set the provider, model and API key for that provider. Set `codeassure_api_base` only when the model is reached through a non-default host:

```yaml
          accuknox_ai_analysis: "true"
          codeassure_provider: <provider>      # anthropic, openai, openai-compatible, google, gemini
          codeassure_model: <model-id>
          codeassure_api_base: <api-url>       # optional
          codeassure_api_key: ${{ secrets.CODEASSURE_API_KEY }}
```

## Quality gate

With `quality_gate: "true"` the scan no longer fails on `severity`. It uploads its results as part of a quality-gate run, waits for AccuKnox to evaluate them against the quality-gate policies configured in AccuKnox, and passes, warns or fails the step from the verdict. The verdict, the findings counts and every breached threshold are printed in the log and written to the job summary; failures and warnings are also added as annotations.

```yaml
      - name: "Run Accuknox SAST: Opengrep"
        uses: accuknox/sast-scan-opengrep-action@latest
        with:
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
          quality_gate: "true"
          gate_token: ${{ secrets.ACCUKNOX_GATE_TOKEN }}
          soft_fail: "false"     # let a failed policy fail the build
```

| AccuKnox verdict | Step result | `gate_status` |
|---|---|---|
| All scans pass the policy | passes | `passed` |
| A `warn` policy is breached | passes, with a warning | `warning` |
| A `block` policy is breached | fails (exit 1). With `soft_fail: "true"`, passes with a warning | `failed` |
| A scan could not be evaluated (unparsable results, policy service down) | fails (exit 2), even with `soft_fail` | `error` |
| No verdict within `gate_timeout` | fails (exit 2). With `gate_on_timeout: pass`, passes | `timeout` |
| The token is not accepted by the gate API | fails (exit 2) | `rejected` |
| The run is unknown to AccuKnox | fails (exit 2) | `unsupported` |
| The scan itself fails (scanner or AI analysis error, upload error) | fails without waiting for a verdict | not set |

### Before you turn it on

- **A policy must cover the repository and branch.** AccuKnox matches policy scopes against `owner/name:branch` with case-sensitive glob patterns, for example `acme/web:*` or `acme/*:main`. When no policy applies, the scan passes without being checked; the action then prints a warning naming the `owner/name:branch` it reported. The branch is:

  | Event | Branch reported |
  |---|---|
  | `push` to a branch | the branch, e.g. `main` |
  | `pull_request` | the PR source branch, e.g. `feature/login` (not the target branch) |
  | `push` of a tag | the tag, e.g. `v1.4.0` |
  | `merge_group` | the merge-queue ref, e.g. `gh-readonly-queue/main/pr-12-…` |
  | `schedule`, `workflow_dispatch` | the branch the workflow ran on |

  To gate pull requests into `main`, scope the policy to every branch of the repository (`acme/web:*`), not to `acme/web:main`.
- **Use a user token for the gate.** The gate API identifies a user, which a label upload token may not. If the step reports `rejected`, set `gate_token` to an AccuKnox user token.
- **`soft_fail` defaults to `true`**, which reports a failed policy without failing the build. Set `soft_fail: "false"` to block on it.
- **Fork and Dependabot pull requests** do not receive repository secrets, so the step fails on the missing endpoint and token. Skip the step for them, for example with `if: github.event_name != 'pull_request' || (github.event.pull_request.head.repo.full_name == github.repository && github.actor != 'dependabot[bot]')`.
- **The first run for a tenant can be slow** while AccuKnox prepares reference data for a scan type. Raise `gate_timeout` if the first runs time out.

### How runs are identified

Every upload and every wait belongs to a gate run, identified by a run id (`cli_id`). By default it is derived from the workflow run (repository, run id and run attempt), so every AccuKnox step and job of one workflow run shares it without any configuration, and a re-run of the whole workflow starts a new gate run. Each upload declares the scan types the run waits for (`gate_scanners`); AccuKnox gives a verdict once every declared type has been uploaded and evaluated.

Two rules follow, and the scenarios below apply them:

1. **Wait once, after all scans of the run have uploaded.** A wait that starts before another scan of the same run uploads either gets a verdict without that scan, or waits for it until `gate_timeout`.
2. **Upload each scan type once per run.** A second upload of the same type is merged into the first (findings counts are added together), and a wait that starts before it is evaluated can return the earlier verdict.

### Scenario: only this SAST scan

Use the example above. The step uploads, waits and decides; nothing else is needed.

### Scenario: several AccuKnox scans in one job

List every scan type in `gate_scanners` on every upload, upload without waiting, and wait once at the end with `gate_only`. The other AccuKnox actions do not take part in the quality gate yet, so run the other scan types with the AccuKnox CLI, which this action leaves at `$RUNNER_TEMP/accuknox/accuknox-aspm-scanner`.

```yaml
    steps:
      - uses: actions/checkout@v4

      - name: "AccuKnox SAST (upload only)"
        uses: accuknox/sast-scan-opengrep-action@latest
        with:
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
          quality_gate: "true"
          gate_scanners: "SG,IAC"
          gate_wait: "false"

      - name: "AccuKnox IaC (upload only)"
        env:
          ACCUKNOX_ENDPOINT: ${{ secrets.ACCUKNOX_ENDPOINT }}
          ACCUKNOX_TOKEN: ${{ secrets.ACCUKNOX_TOKEN }}
          ACCUKNOX_LABEL: ${{ secrets.ACCUKNOX_LABEL }}
        run: |
          SCANNER="$RUNNER_TEMP/accuknox/accuknox-aspm-scanner"
          "$SCANNER" tool install --type iac
          "$SCANNER" scan --quality-gate --gate-scanners SG,IAC iac --command "-d ."

      - name: "AccuKnox quality gate"
        uses: accuknox/sast-scan-opengrep-action@latest
        with:
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          gate_only: "true"
          gate_token: ${{ secrets.ACCUKNOX_GATE_TOKEN }}
          soft_fail: "false"
```

Only list scan types that always upload a result. A secret scan with no findings uploads nothing, so listing it leaves the gate waiting until `gate_timeout`.

### Scenario: scans in separate jobs

Run the gate in its own job that `needs` every scan job. Create the run id once in a first job and pass it to every job: GitHub keeps a successful job's outputs when you use **Re-run failed jobs**, so re-run jobs keep uploading to and waiting on the same gate run.

```yaml
jobs:
  gate-id:
    runs-on: ubuntu-latest
    outputs:
      cli_id: ${{ steps.id.outputs.cli_id }}
    steps:
      - id: id
        run: echo "cli_id=$(cat /proc/sys/kernel/random/uuid)" >> "$GITHUB_OUTPUT"

  sast:
    needs: gate-id
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: accuknox/sast-scan-opengrep-action@latest
        with:
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
          quality_gate: "true"
          gate_scanners: "SG,IAC"
          gate_wait: "false"
          gate_cli_id: ${{ needs.gate-id.outputs.cli_id }}

  # iac:  the IaC job uploads with --quality-gate --gate-scanners SG,IAC and ACCUKNOX_CLI_ID set to the same id

  quality-gate:
    needs: [gate-id, sast, iac]
    runs-on: ubuntu-latest
    steps:
      - uses: accuknox/sast-scan-opengrep-action@latest
        with:
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          gate_only: "true"
          gate_cli_id: ${{ needs.gate-id.outputs.cli_id }}
          gate_token: ${{ secrets.ACCUKNOX_GATE_TOKEN }}
          soft_fail: "false"
```

- Do not run the gate job with `if: always()`. When a scan job fails before it uploads, the gate would wait for it until `gate_timeout`.
- Without a shared id, the jobs still share the derived run id, but only within one run attempt: after **Re-run failed jobs** the re-run jobs get a new id, and the gate reports that it is waiting on a run with no uploads.

### Scenario: re-runs

| What you re-run | Derived run id (no `gate_cli_id`) | Id from a `gate-id` job |
|---|---|---|
| All jobs | Works: new gate run | Works: the re-run `gate-id` job creates a new id |
| Failed jobs, scan and wait in one step | Works: new gate run | Works |
| Failed jobs, scans and gate in separate jobs | Fails: the re-run jobs get a new id and cannot see the earlier uploads (the action says so) | Works, as long as the failed job had not uploaded before it failed |
| Only the gate job | Fails: the earlier attempt's run is not found | Returns the same verdict again; re-run the scans to re-evaluate |

Never hard-code one UUID as `gate_cli_id` for every workflow run: all runs would merge into one gate run.

### Scenario: matrix builds

Every leg of a matrix shares the derived run id, so their uploads of the same scan type are merged and each leg can see a different verdict. Either gate one leg only:

```yaml
          quality_gate: ${{ matrix.os == 'ubuntu-latest' }}
```

or give each leg a gate run of its own with a text `gate_cli_id`:

```yaml
          quality_gate: "true"
          gate_cli_id: ${{ matrix.os }}
```

The same applies when this action runs more than once in one workflow run: give each call its own `gate_cli_id`, or gate only one of them.

### Troubleshooting

| Message | Cause | Fix |
|---|---|---|
| `No quality-gate policy applies to owner/name:branch` | No policy scope matches the repository and branch (case-sensitive) | Widen the policy scope, e.g. `owner/name:*` |
| `rejected (HTTP 401/403 …)` | The gate API does not accept the token | Set `gate_token` to an AccuKnox user token |
| `did not resolve before the timeout` with `Waiting on: <types>` | A declared scan type never uploaded, or AccuKnox is still evaluating | Remove the type from `gate_scanners`, make sure its scan uploads, or raise `gate_timeout` |
| `The backend has no quality-gate run for this cli_id` | Nothing was uploaded with this run id, the gate waited on a different id (for example after **Re-run failed jobs**), or the AccuKnox environment does not support quality gates | Share the run id with `gate_cli_id` (see [Scenario: scans in separate jobs](#scenario-scans-in-separate-jobs)) |
| `Scan type(s) not covered by the quality gate` | `gate_scanners` has a name outside `SG, SQ, TR, IAC, TruffleHog, DS, SBOM` | Fix the name; names are case-sensitive |
| `The quality gate needs the repository and branch` | The repository or branch could not be detected | Set `ACCUKNOX_REPOSITORY` and `ACCUKNOX_BRANCH` in the step environment |
