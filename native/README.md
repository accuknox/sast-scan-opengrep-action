# AccuKnox SAST (native OpenGrep)

Same SAST upload as the root action, but **OpenGrep runs as a local binary**. No Docker, no `--container-mode`.

Use this when the runner cannot pull `opengrep` images, or you do not want nested Docker.

## Difference from the root action

| | Root (`uses: accuknox/sast-scan-opengrep-action@…`) | This (`…/native@…`) |
|---|---|---|
| OpenGrep | `tool install --type sast` then local binary | same |
| Needs Docker | No | No |
| CLI download | `scanner_version` (default `v0.15.1`) | `scanner_version` (default `v0.15.1`) |

## Inputs

Same AccuKnox creds as the root action, plus:

| Name | Required | Default |
|------|----------|---------|
| `scanner_version` | No | `v0.15.1` |

## Usage

```yaml
- name: Checkout
  uses: actions/checkout@v4

- name: AccuKnox SAST (native OpenGrep)
  uses: accuknox/sast-scan-opengrep-action/native@latest
  with:
    accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
    accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
    accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
    soft_fail: "true"
    severity: "HIGH"
    # scanner_version: v0.15.1
```

Runners: **Linux x86_64** (`ubuntu-latest`) and **macOS arm64**. Windows is not wired here.

The job still **uploads** to AccuKnox (does not use `--skip-upload`).
