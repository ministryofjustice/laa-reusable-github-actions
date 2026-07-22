# cp-build-push Github Actions Workflow

Build and push an image to the Cloud Platform, then run [sast workflow](./README-sast.md) and image vulnerability scanning via Snyk.

The workflow builds from a Dockerfile by default. Set `build_mode: gradle-buildpacks` to build a
Spring Boot image with `bootBuildImage`; this mode supports an optional Gradle subproject and does
not require a Dockerfile.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `environment` | No | — | GitHub deployment environment. |
| `dockerfile_path` | No | `Dockerfile` | Dockerfile used in `dockerfile` mode. |
| `build_mode` | No | `dockerfile` | `dockerfile` or `gradle-buildpacks`. |
| `java_version` | No | `25` | Java version used in `gradle-buildpacks` mode. |
| `java_distribution` | No | `temurin` | Java distribution used in `gradle-buildpacks` mode. |
| `jar_subproject` | No | `''` | Gradle subproject containing `bootBuildImage`. |
| `docker_build_args` | No | `''` | Additional arguments used in `dockerfile` mode. |
| `app_name` | Yes | — | Application name used in AWS session names. |
| `image_tag` | No | `${{ github.sha }}` | Primary image tag. |
| `additional_image_tags` | No | `[]` | JSON array of additional image tags. |
| `dockerfile_requires_git` | No | `false` | Provide Git credentials to the Docker build. |

## Required Repo Environment Variables/Secrets:

### Variables

The variables below are mandatory, and are all are injected into the environment automatically by the CP as part of repository configuration, they should not be created manually. See [here](https://user-guide.cloud-platform.service.justice.gov.uk/documentation/deploying-an-app/container-repositories/create.html) for details.

| Variable       | Meaning        |
| -------------- | -------------- |
| ECR_REGION     | AWS Region     |
| ECR_REPOSITORY | ECR Repository |

### Secrets

The secrets below are mandatory `ECR_ROLE_TO_ASSUME` is injected into the environment automatically by the CP, see above.

| Secret             | Meaning                                               |
| ------------------ | ----------------------------------------------------- |
| ECR_ROLE_TO_ASSUME | AWS IAM Role to assume during pipeline authentication |
| SNYK_CLIENT_ID     | Snyk OAuth client ID for SAST and image scanning      |
| SNYK_CLIENT_SECRET | Snyk OAuth client secret for SAST and image scanning  |

## Example Usage

### Create a new image based on release tag

```yaml
on:
  release:
    types:
      - created

jobs:
  build_and_push:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/cp-build-push.yml@main
    secrets: inherit
    with:
      app_name: my-app
      image_tag: ${{ github.event.release.tag_name }}
```

### Create a new image on PR, tagged with the latest commit hash

```yaml
on:
  workflow_dispatch:
  pull_request:
    types:
      - opened
      - synchronize

jobs:
  build_and_push:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/cp-build-push.yml@main
    secrets: inherit
    with:
      app_name: my-app
```

### Build a Spring Boot image with Gradle buildpacks

```yaml
jobs:
  build_and_push:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/cp-build-push.yml@main
    secrets: inherit
    with:
      app_name: my-java-app
      build_mode: gradle-buildpacks
      java_version: '25'
      java_distribution: temurin
      jar_subproject: service
```
