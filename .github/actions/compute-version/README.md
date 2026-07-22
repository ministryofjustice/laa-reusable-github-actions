# Compute version

Computes a semantic version from Git tags and Conventional Commits. Checkout full history first.

```yaml
- uses: actions/checkout@9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0 # v7.0.0
  with:
    fetch-depth: 0
    fetch-tags: true
- id: version
  uses: ministryofjustice/laa-reusable-github-actions/.github/actions/compute-version@<immutable-sha>
  with:
    bump: ${{ inputs.release_type || '' }}
```

`bump` accepts `major`, `minor`, `patch`, `none`, or empty auto-detection. The action understands
`v1.2.3`, legacy `repository-1.2.3`, and bare `1.2.3` tags.

Outputs: `prev_tag`, `bump_type`, `major`, `minor`, `patch`, and `next_version`.
