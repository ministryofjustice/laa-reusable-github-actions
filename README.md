# LAA Reusable Github Actions

Following the publication of [this article](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows) by Github
announcing the beta release of reusable workflows, this is an attempt to start a central repository 
of github jobs that can be called from remote repositories.

## Notices
This is a beta function from Github and my be changed/break at very short notice

## Currently available actions
* rubocop
* rspec
* brakeman

## Invoking an action

In your repo's workflows you can invoke one of these actions with 
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

So it doesn't  duplicate the  local job `rubocop` e.g.
```text
Pull request / rubocop / rubocop (pull_request)
Pull request / something (pull_request)
```

