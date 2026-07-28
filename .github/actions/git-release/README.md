# Git release

Idempotently creates an annotated Git tag and, by default, a GitHub Release.

```yaml
- id: release
  uses: ministryofjustice/laa-reusable-github-actions/.github/actions/git-release@<immutable-sha>
  with:
    tag: ${{ steps.version.outputs.next_version }}
    github_token: ${{ secrets.GITHUB_TOKEN }}
```

The caller needs `contents: write`; checkout must retain credentials capable of pushing the tag.
Inputs are `tag` (required), `github_bot_username`, `github_token`, and
`create_github_release` (default `true`). The `gtag` output contains the existing or created tag.

To publish an artifact between tagging and creating the GitHub Release, call once with
`create_github_release: 'false'`, publish using `gtag`, then call again with the same tag.
