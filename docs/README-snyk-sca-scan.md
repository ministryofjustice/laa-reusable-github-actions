# Snyk Dependency (SCA) Scan — Reusable Workflow

A reusable GitHub Actions workflow that runs a plain Snyk open-source/
dependency (SCA) scan against a repo's build manifest. Results are
uploaded to GitHub's Security tab as SARIF reports.

This complements [`sast.yml`](../.github/workflows/sast.yml), which
covers container image, Infrastructure as Code, and application code
scanning, but does **not** scan declared dependencies.

## Why this exists (read before adopting)

This is an **additive** scan, not a policy requirement. LAA's
Guardrail 2 (Secure Pipelines) and Guardrail 8 (Tooling Catalogue)
name **Dependabot/Renovate**, not Snyk, as the default
dependency-scanning tool. Dependabot alone already satisfies Pipeline
Maturity Level 2 ("SAST + Dependency scanning"). This workflow exists
because Dependabot isn't yet being triaged and actioned consistently
across our repos, so it's a deliberate, interim, defence-in-depth
measure; not a replacement for Dependabot, and not something every
repo is required to adopt. Expect to deprecate it once Dependabot is
properly adopted and acted on across the team's repos.

## Usage

```yaml
jobs:
  sca-scan:
    name: Scan dependencies with Snyk
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/snyk-sca-scan.yml@<insert latest sha here>
    permissions:
      contents: read
      security-events: write
      actions: read
    secrets:
      SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

| Name              | Permission | Reason                                                                                                                                           |
|-------------------|------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| `contents`        | `read`     | Allows `actions/checkout` to clone the repository.                                                                                               |
| `security-events` | `write`    | Allows `codeql-action/upload-sarif` to publish scan results to the GitHub Security tab.                                                          |
| `actions`         | `read`     | Required by `codeql-action/upload-sarif` to read workflow run metadata so it can correctly associate SARIF results with the triggering workflow. |

## Inputs

| Name                | Required | Default  | Description                                                                                                                                                                                                                                                                                                                                                                                                |
|---------------------|----------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `build_tool`        | No       | `gradle` | Build tool used to resolve dependencies before scanning. One of `gradle`, `maven`. Determines only which manifest file is checked for; `snyk test` auto-detects the manifest itself.                                                                                                                                                                                                                       |
| `working_directory` | No       | `.`      | Directory containing the manifest (`build.gradle`/`build.gradle.kts`/`pom.xml`) to scan.                                                                                                                                                                                                                                                                                                                   |
| `minimum_threshold` | No       | `high`   | Minimum severity level to fail on. One of: `low`, `medium`, `high`, `critical`.                                                                                                                                                                                                                                                                                                                            |
| `scan_all_projects` | No       | `false`  | Set `true` to scan all sub-projects/modules in a monorepo. **Required for multi-module Gradle/Maven repos** — without it Snyk can scan an empty root build and return a false-clean result. Internally mapped to `--all-sub-projects` for `gradle`, `--maven-aggregate-project` for `maven`; kept build-tool-agnostic in name so it can map to a different Snyk flag if other build tools are added later. |
| `policy_path`       | No       | `.snyk`  | Path to a `.snyk` policy file, relative to `working_directory`. Leave blank if none exists — the workflow only passes `--policy-path` when the file is present.                                                                                                                                                                                                                                            |

## Secrets

Same authentication options as `sast.yml`:

*Option 1: OAuth (recommended)*

| Name                 | Description              |
|----------------------|--------------------------|
| `SNYK_CLIENT_ID`     | Snyk OAuth client ID     |
| `SNYK_CLIENT_SECRET` | Snyk OAuth client secret |

*Option 2: Static token*

| Name         | Description                                                              |
|--------------|--------------------------------------------------------------------------|
| `SNYK_TOKEN` | Snyk API token. If used instead of OAuth, rotate at least every 90 days. |

## What it scans

`snyk test` runs against the manifest in `working_directory`, gated on
`minimum_threshold`. There is no `continue-on-error` and no exit-code
suppression. A finding at or above the threshold fails the job the
normal GitHub Actions way (`snyk test` exits non-zero, and
`continue-on-error: false` is set explicitly on the step). Same
gating behaviour as `sast.yml`'s scans, which rely on the same
(implicit) GHA default.

## Results

SARIF output is uploaded to GitHub's "Security & Quality" tab, scoped
to the repository, under **Security & Quality → Code scanning
alerts**. As with `sast.yml`, SARIF files are sanitised before upload
to replace any `null`/`undefined` severity values, which would
otherwise cause the upload to fail.

## Examples

### Single-module Gradle repo

```yaml
jobs:
  sca-scan:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/snyk-sca-scan.yml@<insert latest sha here>
    permissions:
      contents: read
      security-events: write
      actions: read
    secrets:
      SNYK_CLIENT_ID: ${{ secrets.SNYK_CLIENT_ID }}
      SNYK_CLIENT_SECRET: ${{ secrets.SNYK_CLIENT_SECRET }}
```

### Multi-module Gradle repo

```yaml
jobs:
  sca-scan:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/snyk-sca-scan.yml@<insert latest sha here>
    permissions:
      contents: read
      security-events: write
      actions: read
    with:
      scan_all_projects: true
    secrets:
      SNYK_CLIENT_ID: ${{ secrets.SNYK_CLIENT_ID }}
      SNYK_CLIENT_SECRET: ${{ secrets.SNYK_CLIENT_SECRET }}
```

### Maven repo in a subdirectory, lower threshold

```yaml
jobs:
  sca-scan:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/snyk-sca-scan.yml@<insert latest sha here>
    permissions:
      contents: read
      security-events: write
      actions: read
    with:
      build_tool: maven
      working_directory: services/api
      minimum_threshold: medium
    secrets:
      SNYK_CLIENT_ID: ${{ secrets.SNYK_CLIENT_ID }}
      SNYK_CLIENT_SECRET: ${{ secrets.SNYK_CLIENT_SECRET }}
```

## Further reading

See the **Security Guardrails for Secure Pipelines** documentation for
policy context and guidance on when this workflow is required.
