# cp-build-push-deploy Github Actions Workflow

Build and push and Image to the Cloud Platform and then deploy it using using the laa-generic-service Helm Chart

## Required Repo Environment Variables/Secrets:

### Variables

| Secret     | Meaning    |
|------------|------------|
| AWS_REGION | AWS Region |

### Secrets

| Secret                | Meaning                                                    |
|-----------------------|------------------------------------------------------------|
| DEPLOYMENT_ACCOUNT_ID | AWS Account ID for the account being authenticated against |
| ECR_ACCOUNT_ID        | AWS Account ID containing the ECR Repo being pushed to     |

## Example Usage

### Build Push and Deploy to CP on new tagged release

```yaml
on:
  release:
    types:
      - created

jobs:
  cp_build_and_deploy:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/cp-build-push-deploy.yml@main
    with:
      environment: dev #--Requires multiple Github Environments set up in CP
      image_tag: ${{ github.event.release.tag_name }}
      helm_release_name: my-release-name
      helm_chart: ministryofjustice/laa-generic-service #--WIP
      helm_values_path: ./helm/values.yaml
      k8s_service_account: cd-serviceaccount #--Requires CD service account created by CP
    secrets: inherit
```
