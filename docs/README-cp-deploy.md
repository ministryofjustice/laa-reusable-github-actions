# cp-deploy Github Actions Workflow

Deploy an application to the Cloud Platform using the laa-generic-service Helm Chart (WIP)

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

### Deploy to CP on PR

```yaml
on:
  workflow_dispatch:
  pull_request:
    types:
      - opened
      - synchronize

jobs:
  helm_deploy:
    needs: build_and_push
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/cp-deploy.yml@main
    with:
      environment: dev #--Requires multiple Github Environments set up in CP
      image_registry: 123456789012.dkr.ecr.eu-west-2.amazonaws.com #--Change as appropriate
      image_repo: my-image-repo
      image_tag: 0.0.1
      k8s_service_account: cd-serviceaccount #--Requires CD service account created by CP
      helm_release_name: my-release-name
      helm_values_path: ./helm/values.yaml
    secrets: inherit
```

### Deploy to CP after building an image using the cp-build-push workflow

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

  helm_deploy:
    needs: build_and_push
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/cp-deploy.yml@main
    with:
      environment: dev #--Requires multiple Github Environments set up in CP
      image_repo: ${{ needs.build_and_push.outputs.image_repo }}
      image_registry: ${{ needs.build_and_push.outputs.image_registry }}
      image_tag: ${{ needs.build_and_push.outputs.image_tag }}
      k8s_service_account: cd-serviceaccount #--Requires CD service account created by CP
      helm_release_name: my-release-name
      helm_values_path: ./helm/values.yaml
    secrets: inherit
```
