# cp-build-push-deploy Github Actions Workflow

Build and push and Image to the Cloud Platform and then deploy it using using the laa-generic-service Helm Chart (WIP)

## Required Repo Environment Variables/Secrets:

### Variables

| Variable       | Meaning        |
| -------------- | -------------- |
| ECR_REGION     | AWS Region     |
| ECR_REPOSITORY | ECR Repository |

### Secrets

| Secret             | Meaning                                               |
| ------------------ | ----------------------------------------------------- |
| ECR_ROLE_TO_ASSUME | AWS IAM Role to assume during pipeline authentication |
| SNYK_CLIENT_ID     | Snyk OAuth client ID for SAST and image scanning      |
| SNYK_CLIENT_SECRET | Snyk OAuth client secret for SAST and image scanning  |
| KUBE_CLUSTER       | Kubernetes cluster endpoint                           |
| KUBE_NAMESPACE     | Kubernetes namespace to deploy to                     |
| KUBE_TOKEN         | Kubernetes service account token                      |
| KUBE_CERT          | Kubernetes cluster certificate authority              |

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
      app_name: my-app
      image_tag: ${{ github.event.release.tag_name }}
      helm_release_name: my-release-name
      helm_chart: ./laa-generic-helm-chart
      helm_values_path: ./helm/values.yaml
      k8s_service_account: cd-serviceaccount #--Requires CD service account created by CP
    secrets: inherit
```
