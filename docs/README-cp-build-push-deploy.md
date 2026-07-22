# cp-build-push-deploy Github Actions Workflow

Build and push and Image to the Cloud Platform and then deploy it using using the laa-generic-service Helm Chart (WIP)

The build uses a Dockerfile by default. Set `build_mode: gradle-buildpacks` to build a Spring Boot
image with `bootBuildImage` before running the same Snyk scan and Helm deployment.

## Build Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `dockerfile_path` | No | `Dockerfile` | Dockerfile used in `dockerfile` mode. |
| `build_mode` | No | `dockerfile` | `dockerfile` or `gradle-buildpacks`. |
| `java_version` | No | `25` | Java version used in `gradle-buildpacks` mode. |
| `java_distribution` | No | `temurin` | Java distribution used in `gradle-buildpacks` mode. |
| `jar_subproject` | No | `''` | Gradle subproject containing `bootBuildImage`. |
| `docker_build_args` | No | `''` | Additional arguments used in `dockerfile` mode. |
| `image_tag` | No | `${{ github.sha }}` | Primary image tag. |
| `additional_image_tags` | No | `[]` | JSON array of additional image tags. |
| `dockerfile_requires_git` | No | `false` | Provide Git credentials to the Docker build. |

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

### Build a Spring Boot image and deploy it

```yaml
jobs:
  cp_build_and_deploy:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/cp-build-push-deploy.yml@main
    with:
      environment: dev
      app_name: my-java-app
      build_mode: gradle-buildpacks
      java_version: '25'
      java_distribution: temurin
      jar_subproject: service
      helm_release_name: my-java-app
      helm_chart: ./laa-generic-helm-chart
      helm_values_path: ./helm/values.yaml
    secrets: inherit
```
