# Check for synced branches action
An action to check if 2 branches are synced and output their differences if any
## How to
The action can be called with
```yaml
- name: Check for synced branches action
  uses: MadLadSquad/check-for-synced-branches-action@v1.0.0.0
  with:
    upstream-url: "https://github.com/foo/foo"
    first-branch: master
    second-branch: development
```
### Inputs
The fields `first-branch` and `second-branch` are required because they store the names for the branches to compare.

The `upstream-url` field should contain the URL to a repository on github or anywhere else. If the field is not specified the action uses the `origin` remote

An additional field is the `both-upstream` bool, which when set to true will set all the remotes to the `upstream` remote added by specifying the `upstream-url` field
### Outputs
The action outputs the `ahead` and `behind` integers, which can be used in your action if statements to enable and disable event such as creating an issue or pull request. Here is an example of an action that creates an issue when the `origin/master` branch is behind `upstream/master`
```yaml
name: Create automatic issue
env:
  REPO_NAME: "kcat/openal-soft"
  
on:
  schedule:
    - cron:  "0 0 * * *"

jobs:
  Update-Fork:
    runs-on: ubuntu-latest
    steps:
      - name: checking out code
        uses: actions/checkout@v2
      - name: Extract commit data
        uses: rlespinasse/git-commit-data-action@v1.x
      - name: Check for synced branches action
        id: check
        uses: MadLadSquad/check-for-synced-branches-action@master
        with:
          upstream-url: "https://github.com/${{ env.REPO_NAME }}"
          first-branch: master
          second-branch: master
      - name: Issue
        if: steps.check.outputs.behind > 0
        uses: JasonEtco/create-an-issue@v2 # Creates an issue
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          filename: .github/auto-issue-template.md
          update_existing: true
          search_existing: all
```
