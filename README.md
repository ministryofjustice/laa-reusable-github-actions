# LAA Reusable Github Actions

Following the publication of [this article](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows) by Github
announcing the beta release of reusable workflows, this is an attempt to start a central repository 
of github jobs that can be called from remote repositories.

## Notices
This is a beta function from Github and my be changed/break at very short notice

## Currently available actions
* rubocop
* rspec

## Invoking an action

In your repo's workflows you can invoke one of these actions with 
```yaml
jobs:
  call-workflow:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/rubocop.yml@main
```
This can be broken down as follows:

`ministryofjustice/laa-reusable-github-actions/.github/workflows/rubocop.yml@main`  
`{---------------- repo_name ----------------}/{-- path to workflow file --}@{version}`

Where version is the branch name  
So if you create a branch named `improve-rubocop` you could test the 
workflow on your remote repo by calling `ministryofjustice/laa-reusable-github-actions/.github/workflows/rubocop.yml@improve-rubocop`

### Warning
The header you give your job in _your_ repo will be reflected in the calling path inside your pull request
So if you have a pull request workflow file in your repo like so:
```yaml
name: Pull request
on: [pull_request]

jobs:
  call-workflow:
    uses: ministryofjustice/laa-reusable-github-actions/.github/workflows/rubocop.yml@main

  rspec:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:10.11
        ports: [ "5432:5432" ]
        options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5
        # <snip>
```
Your pull request checks will resemble
```text
Pull request / call-workflow / rubocop (pull_request)
Pull request / rspec (pull_request)
``` 

So beware of naming your local job `rubocop` or your jobs will resemble
```text
Pull request / rubocop / rubocop (pull_request)
Pull request / rspec (pull_request)
```

