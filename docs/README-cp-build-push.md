# cp-build-push Github Actions Workflow

Build and push an image to the Cloud Platform

## Required Repo Environment Variables/Secrets:

### Variables

The variables and secrets below are mandatory, but all are injected in to the environment automatically by the CP as part of repository configuration, they should not be created manually. See [here](https://user-guide.cloud-platform.service.justice.gov.uk/documentation/deploying-an-app/container-repositories/create.html) for details.

| Secret     | Meaning                       |
|------------|-------------------------------|
| ECR_REGION | AWS Region                    |
| ECR_REPOSITORY | ECR Repository to Push to |

### Secrets

| Secret             | Meaning                                               |
|--------------------|-------------------------------------------------------|
| ECR_ROLE_TO_ASSUME | AWS IAM Role to assume during pipeline authentication |

## Example Usage

### Create a new image based on release tag

```yaml
on:
  release:
    types:
      - created

jobs:
  build_and_push:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/mp-build-push.yml@main
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
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/mp-build-push.yml@main
    secrets: inherit
    with:
      app_name: my-app
```