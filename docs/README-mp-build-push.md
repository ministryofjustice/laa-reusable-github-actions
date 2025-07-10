# mp-build-push Github Actions Workflow

Build and push an image to the Modernisation Platform

## Required Repo Environment Variables/Secrets:

### Variables

| Secret     | Meaning    |
| ---------- | ---------- |
| AWS_REGION | AWS Region |

### Secrets

| Secret                | Meaning                                                    |
| --------------------- | ---------------------------------------------------------- |
| DEPLOYMENT_ACCOUNT_ID | AWS Account ID for the account being authenticated against |
| ECR_ACCOUNT_ID        | AWS Account ID containing the ECR Repo being pushed to     |

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
      image_repo: my-image-repo #--Repo name only, not full arn or registry/repo format
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
      image_repo: my-image-repo #--Repo name only, not full arn or registry/repo format
```
