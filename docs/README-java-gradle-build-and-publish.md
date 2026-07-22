# Java Gradle build and publish workflow

`java-gradle-build-and-publish.yml` is the standard entry point for Java repositories that build
and publish with Gradle. It checks out full Git history, computes a version, builds and tests,
optionally runs Semgrep and JaCoCo verification, publishes the artifact, creates a Git tag and
GitHub Release, and uploads reports.

Use an immutable commit SHA. Replace `<immutable-sha>` in the examples with the promoted commit
from this repository.

## Main-branch release

This combines the release patterns used by Data Access, Data Claims and Submit a Bulk Claim.
Downstream Pact, image and deployment jobs remain in the application repository and consume
`published_artifact_version`.

```yaml
name: Build main

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      release_type:
        type: choice
        default: patch
        options: [patch, minor, major]

jobs:
  release:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/java-gradle-build-and-publish.yml@<immutable-sha>
    permissions:
      contents: write
      packages: write
    with:
      java_version: '25'
      java_distribution: corretto
      build_command: ':service:assemble'
      integration_test_task: integrationTest
      release_type: ${{ inputs.release_type || '' }}
      semgrep_check: true
      jacoco_coverage_report: false
    secrets:
      gh_token: ${{ secrets.GITHUB_TOKEN }}

  next-job:
    needs: release
    runs-on: ubuntu-latest
    steps:
      - run: echo "Released ${{ needs.release.outputs.published_artifact_version }}"
```

An empty `release_type` on `main` derives major/minor/patch from Conventional Commits. Explicit
`patch`, `minor` or `major` values override detection. Use `none` for build and test without
publishing.

## Feature or pull-request snapshot

The workflow computes the snapshot version itself, so callers do not need a separate version job.

```yaml
name: Build feature

on:
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/java-gradle-build-and-publish.yml@<immutable-sha>
    permissions:
      contents: write
      packages: write
    with:
      java_version: '25'
      release_type: snapshot
      build_command: ':app:assemble'
      integration_test_task: integrationTest
      semgrep_check: true
      jacoco_coverage_report_path: app/build/reports/jacoco
      junit_results_path: app/build/test-results
      junit_report_path: app/build/reports/tests
    secrets:
      gh_token: ${{ secrets.GITHUB_TOKEN }}

  publish-image:
    needs: build-and-test
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/java-ecr-publish-image.yml@<immutable-sha>
    permissions:
      contents: read
      id-token: write
      security-events: write
    with:
      image_version: ${{ needs.build-and-test.outputs.published_artifact_version }}
      jar_subproject: app
    secrets: inherit
```

Supply `override_version` only when the repository has an established external preview-version
format that must be retained.

## Inputs

| Input | Default | Description |
|---|---|---|
| `java_version` | `25` | JDK version. |
| `java_distribution` | `temurin` | JDK distribution understood by `setup-java`. |
| `build_command` | `build` | Gradle task, including an optional subproject prefix. |
| `build_args` | empty | Additional whitespace-separated Gradle arguments. |
| `integration_test_task` | empty | Additional integration-test task. |
| `release_type` | empty | `major`, `minor`, `patch`, `snapshot`, `none`, or empty auto-detection. |
| `override_version` | empty | Explicit snapshot/output version. |
| `semgrep_check` | `false` | Run the central Semgrep action before building. |
| `jacoco_coverage_report` | `true` | Run JaCoCo verification and upload its report. |
| `junit_results` / `junit_report` / `checkstyle_report` | `true` | Enable corresponding artifact uploads. |
| `*_path` | Gradle defaults | Override report paths for multi-module builds. |
| `github_bot_username` | `github-actions-bot` | Identity used for annotated release tags. |

## Secrets and output

`gh_token` is required. `aws_region` is optional and exposes `AWS_REGION` to Gradle for services
whose build configuration requires it. Sonatype username/password and GPG key/passphrase are
optional; provide all four to publish through Sonatype. Otherwise the workflow runs
`./gradlew publish`.

The `published_artifact_version` output contains the release version without `v`, or the complete
snapshot version.
