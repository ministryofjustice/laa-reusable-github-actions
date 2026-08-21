# Snyk Test Reusable Workflow

Reusable GitHub Actions workflow for running `snyk test` against a repository and optionally uploading SARIF to GitHub Code Scanning.

Workflow path:

`.github/workflows/snyk-test.yml`

## Usage

```yaml
jobs:
  snyk_test:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/snyk-test.yml@<pin-to-sha>
    permissions:
      actions: read
      contents: read
      security-events: write
    with:
      snyk_args: --severity-threshold=high --all-projects
      fail_on_issues: true
      upload_sarif: true
    secrets:
      snyk_token: ${{ secrets.SNYK_TOKEN }}
```

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `snyk_args` | No | `--severity-threshold=high` | Extra arguments passed to `snyk test`. Use this to set severity threshold, project discovery flags, org, policy file, etc. |
| `fail_on_issues` | No | `true` | If `true`, the workflow fails when Snyk finds issues at or above your configured threshold. |
| `upload_sarif` | No | `true` | If `true`, uploads `snyk.sarif` to GitHub Code Scanning. |

## Secrets

| Name | Required | Description |
|---|---|---|
| `snyk_token` | Yes | Snyk API token used to authenticate CLI calls. |

## Common configurations

### Fail on high or critical (typical PR gate)

```yaml
with:
  snyk_args: --severity-threshold=high --all-projects
  fail_on_issues: true
```

### Fail only on critical

```yaml
with:
  snyk_args: --severity-threshold=critical --all-projects
  fail_on_issues: true
```

### Report findings but do not block merges

```yaml
with:
  snyk_args: --severity-threshold=high --all-projects
  fail_on_issues: false
```

## How to tailor this for your repo

Most teams only need to change `snyk_args` and `fail_on_issues`:

1. Start with `--severity-threshold=high` and `--all-projects`.
2. Tighten to `critical` if `high` creates too much noise.
3. Set `fail_on_issues: false` temporarily while you baseline existing findings.
4. Re-enable `fail_on_issues: true` once your backlog is under control.

You can pass any supported `snyk test` flags in `snyk_args`.

## Notes

- If `upload_sarif` is enabled, ensure the caller job permissions include `security-events: write`.
- Pin to a commit SHA in `uses:` for stability and supply-chain safety.
