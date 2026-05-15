# cp-build-push Github Actions Workflow

Build and push an image to the Cloud Platform, then run [sast workflow](./README-sast.md) and image vulnerability scanning via Snyk.

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
      docker_additional_args: --secret id=github_token,env=GITHUB_TOKEN
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
      docker_additional_args: --secret id=github_token,env=GITHUB_TOKEN
```
