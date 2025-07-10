# LAA Reusable Github Actions

Following the publication of [this article](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows) by Github announcing the beta release of reusable workflows, this is an attempt to start a central repository of github jobs that can be called from remote repositories.

## Notices

This is a beta function from Github and my be changed/break at very short notice

## Currently available workflows

- brakeman
- format
- rspec
- rubocop
- snyk
- Build and Push Image (Cloud Platform)
- Build and Push Image (Modernisation Platform)
- Build, Push and Deploy via Helm (Cloud Platform). For use with a dedicated generic-service helm chart

## Invoking a workflow

In your repo's workflows you can invoke one of the workflows (available in `.github/workflows/`) with

```yaml
jobs:
  rubocop:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/rubocop.yml@main
```

This can be broken down as follows:

`ministryofjustice/laa-reusable-github-actions/.github/workflows/rubocop.yml@main`  
`{---------------- repo_name ----------------}/{-- path to workflow file --}@{version}`

Where version is the branch name  
So if you create a branch named `improve-rubocop` you could test the
workflow on your remote repo by calling `ministryofjustice/laa-reusable-github-actions/.github/workflows/rubocop.yml@improve-rubocop`

## Currently available actions

- authenticate_to_cluster (k8s auth)
- ecr-auth (bundled AWS and ECR auth)
- helm-deploy (helm upgrade, for use with a dedicated generic-service helm chart)
- helm-kube-score (kube-score via helm templating)
- image-scan (trivy)
- secret-detection (trufflehog)

## Invoking an action

In your repo's workflows you can invoke one of the github actions (available in `.github/actions/`) with

```yaml
# example staging deployment requiring authentication to the kubernetes cluster
  deploy-staging:
    runs-on: ubuntu-latest
    needs: build
    environment: staging

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Authenticate to cluster
        uses: ministryofjustice/laa-reusable-github-actions/.github/actions/authenticate_to_cluster@main
        with:
          kube-cert: ${{ secrets.KUBE_STAGING_CERT }}
          kube-token: ${{ secrets.KUBE_STAGING_TOKEN }}
          kube-cluster: ${{ secrets.KUBE_STAGING_CLUSTER }}
          kube-namespace: ${{ secrets.KUBE_STAGING_NAMESPACE }}

      - name: deployment
        ...

```

Note that you may want to use the action but pin it so changes to it do not impact your workflow. To achieve this you can use a commit shah after the `@`.

```yaml
# example using a pinned version of the action for security and stability
uses: ministryofjustice/laa-reusable-github-actions/.github/actions/authenticate_to_cluster@7666443ea3f635ec52544311e0f0ea51830468c0
```

### Warning

I have set the job name of the initial workflows as `scan` so that, when you use a header
in _your_ repo it will be reflected in the calling path inside your pull request
So if you have a pull request workflow file in your repo like so:

```yaml
name: Pull request
on: [pull_request]

jobs:
  rubocop:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/rubocop.yml@main

  something:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:10.11
        # <snip>
```

Your pull request checks will resemble

```text
Pull request / rubocop / scan (pull_request)
Pull request / something (pull_request)
```

So it doesn't duplicate the local job `rubocop` e.g.

```text
Pull request / rubocop / rubocop (pull_request)
Pull request / something (pull_request)
```
