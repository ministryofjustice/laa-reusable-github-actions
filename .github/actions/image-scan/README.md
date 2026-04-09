# scan-image

Scans a container image for vulnerabilities using Snyk. Runs on every call, uploads results to GitHub Code Scanning, and tracks the image in the Snyk dashboard.

> [!NOTE]
> This workflow should be considered deprecated and instead use [sast.yml](../../workflows/sast.yml)

## Usage

```yaml
jobs:
  scan:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/scan-image.yml@main
    with:
      image_uri: ${{ env.ECR_IMAGE }}
    secrets: inherit
```

## Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `image_uri` | yes | | Full URI of the image to scan |
| `dockerfile_path` | no | `Dockerfile` | Path to the Dockerfile relative to the repo root |

## Secrets

`SNYK_CLIENT_ID` and `SNYK_CLIENT_SECRET` must be available in the
calling repo. Set these at in the repository secrets and they'll be picked up
automatically via `secrets: inherit`.
