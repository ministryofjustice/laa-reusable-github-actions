# Snyk image scan

Composite action that scans an existing local or registry image with Snyk, cleanses the SARIF
report, optionally uploads it to GitHub code scanning, and fails after reporting vulnerabilities.

```yaml
- name: Scan image
  uses: ministryofjustice/laa-reusable-github-actions/.github/actions/image-scan@<immutable-sha>
  with:
    image_uri: ${{ steps.build.outputs.image_uri }}
    dockerfile_path: Dockerfile
    snyk_client_id: ${{ secrets.SNYK_CLIENT_ID }}
    snyk_client_secret: ${{ secrets.SNYK_CLIENT_SECRET }}
    severity: high
    fail_on: upgradable
```

OAuth credentials are preferred. `snyk_token` is retained as a deprecated fallback and produces a
migration warning. Set `dockerfile_path: ''` for buildpacks images without a Dockerfile.

| Input | Default | Description |
|---|---|---|
| `image_uri` | required | Image to scan. |
| `dockerfile_path` | `Dockerfile` | Dockerfile context, or empty. |
| `snyk_client_id` / `snyk_client_secret` | empty | OAuth credentials. |
| `snyk_token` | empty | Deprecated static token. |
| `severity` | `medium` | Minimum severity. |
| `fail_on` | `upgradable` | Snyk failure policy. |
| `policy_path` | `.snyk` | Policy file. |
| `upload_sarif` | `true` | Upload to GitHub code scanning. |

SARIF upload requires `security-events: write` in the calling job.
