# Java image build, scan and ECR publish workflow

`java-ecr-publish-image.yml` builds a Java service image, optionally scans it with Snyk, and
publishes it to ECR. Use `dockerfile_path` for a Dockerfile build; omit it for Spring Boot
`bootBuildImage` and optionally select a Gradle `jar_subproject`.

## Dockerfile image following a main release

This is the Data Access pattern and can be repeated for secondary images such as a mass generator.

```yaml
jobs:
  publish-image:
    needs: release
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/java-ecr-publish-image.yml@<immutable-sha>
    permissions:
      contents: read
      id-token: write
      security-events: write
    with:
      dockerfile_path: Dockerfile
      image_version: ${{ needs.release.outputs.published_artifact_version }}
      image_scan: true
    secrets:
      ecr_repository: ${{ vars.ECR_REPOSITORY }}
      ecr_region: ${{ vars.ECR_REGION }}
      ecr_role_to_assume: ${{ secrets.ECR_ROLE_TO_ASSUME }}
      snyk_token: ${{ secrets.SNYK_TOKEN }}
```

## Gradle buildpacks image for a multi-module service

This follows the Data Claims and Submit a Bulk Claim pattern. OAuth credentials are preferred for
Snyk authentication.

```yaml
jobs:
  publish-image:
    needs: build-and-test
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/java-ecr-publish-image.yml@<immutable-sha>
    permissions:
      contents: read
      id-token: write
      security-events: write
    with:
      java_version: '25'
      java_distribution: corretto
      jar_subproject: 'claims-data:service'
      image_version: ${{ format('{0}-{1}', vars.IMAGE_PREFIX, needs.build-and-test.outputs.published_artifact_version) }}
      tag_with_latest: false
    secrets:
      ecr_repository: ${{ vars.ECR_REPOSITORY }}
      ecr_region: ${{ vars.ECR_REGION }}
      ecr_role_to_assume: ${{ secrets.ECR_ROLE_TO_ASSUME }}
      snyk_client_id: ${{ secrets.SNYK_CLIENT_ID }}
      snyk_client_secret: ${{ secrets.SNYK_CLIENT_SECRET }}
```

The workflow builds without pushing, scans the local image, then publishes only after a successful
scan. Set `publish: false` to build and scan without ECR publication.

## Inputs

| Input | Default | Description |
|---|---|---|
| `java_version` | `25` | JDK version. |
| `java_distribution` | `temurin` | JDK distribution. |
| `image_version` | required | Primary image tag. |
| `dockerfile_path` | empty | Dockerfile path; empty selects Gradle buildpacks. |
| `docker_build_args` | empty | Additional Docker build arguments. |
| `jar_subproject` | empty | Gradle subproject containing `bootBuildImage`. |
| `image_scan` | `true` | Run Snyk container scanning. |
| `image_scan_severity` | `medium` | Minimum Snyk severity. |
| `image_scan_fail_on` | `upgradable` | Snyk `--fail-on` value. |
| `image_scan_policy_path` | `.snyk` | Snyk policy path. |
| `image_scan_upload_sarif` | `true` | Upload cleansed SARIF to code scanning. |
| `publish` | `true` | Authenticate and push to ECR. |
| `tag_with_latest` | `false` | Also publish `latest`. |

## Secrets and output

`ecr_region`, `ecr_repository` and `ecr_role_to_assume` are required. Scanning requires either
OAuth (`snyk_client_id` plus `snyk_client_secret`) or the deprecated static `snyk_token`.
`ecr_registry` supports a different registry account. Certificate bindings use
`root_certificate`, `binding_directory` and optional `tls_keystore_password`.

`published_image_version` is populated when `publish` is true and is suitable for downstream
deployment jobs.
