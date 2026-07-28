# Semgrep scan

Runs the repository-independent Semgrep `auto` ruleset against the caller's repository. The action
checks out the repository and runs Semgrep in the `returntocorp/semgrep` container, so callers do
not need to install Semgrep on their runner.

The scan exits unsuccessfully when Semgrep reports a finding. It retains the existing exclusion
for `python.django.security.django-no-csrf-token.django-no-csrf-token`.

```yaml
steps:
  - uses: ministryofjustice/laa-reusable-github-actions/.github/actions/semgrep-scan@<immutable-sha>
```

The action is language-agnostic and can be used by any repository whose runner provides Docker.
