# Check for synced branches action
[![MIT license](https://img.shields.io/badge/License-MIT-blue.svg)](https://lbesson.mit-license.org/)
[![trello](https://img.shields.io/badge/Trello-UDE-blue])](https://trello.com/b/HmfuRY2K/untitleddesktop)
[![Discord](https://img.shields.io/discord/717037253292982315.svg?label=&logo=discord&logoColor=ffffff&color=7389D8&labelColor=6A7EC2)](https://discord.gg/4wgH8ZE)

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

### Setup
Before we begin Github Actions pull without any commit history, lfs objects, submodules and such, which might be useful to you. The important thing is that this action cannot run without them as it will report false issues. In order to allow this you need to modify your checkout action to look like this
```yaml
- name: checking out code
  uses: actions/checkout@v3
  with:
    ref: master
    token: ${{ secrets.GITHUB_TOKEN }}
    lfs: true
    submodules: true
    clean: false
    fetch-depth: 0
```
to briefly explain if you don't know what any of this does:
1. The `ref` field specifies the current branch to check out
1. The `token` field is the toke with which github gains access to the repo, this token is predefined so you can leave the setting as is
1. The `lfs` field enables [Git LFS](https://git-lfs.github.com/)
1. The `submodules` field checks out the [submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) in the repository if any
1. The `clean` field makes github not run `git clean -ffdx && git reset --hard HEAD`
1. The `fetch-depth` fields specifies how much of the commit history to download, at default it's set to `1` but when set to `0` it downloads the whole history instead of just 1 commit

Once you have all this done you can move to the next step!

### Inputs
The fields `first-branch` and `second-branch` are required because they store the names for the branches to compare.

The `upstream-url` field should contain the URL to a repository on github or anywhere else. If the field is not specified the action uses the `origin` remote

An additional field is the `both-upstream` bool, which when set to true will set all the remotes to the `upstream` remote added by specifying the `upstream-url` field

### Outputs
The action outputs the `ahead` and `behind` integers, which can be used in your action if statements to enable and disable event such as creating an issue or pull request. Here is an example of an action that creates an issue when the `origin/master` branch is behind `upstream/master`
```yaml
name: Create automatic issue
on:
  schedule:
    - cron:  "0 0 * * *"
jobs:
  Update-Fork:
    runs-on: ubuntu-latest
    steps:
      - name: checking out code
        uses: actions/checkout@v3
        with:
          ref: master
          token: ${{ secrets.GITHUB_TOKEN }}
          lfs: true
          submodules: true
          clean: false
          fetch-depth: 0
      - name: Extract commit data
        uses: rlespinasse/git-commit-data-action@v1.x
      - name: Check for synced branches action
        id: check
        uses: MadLadSquad/check-for-synced-branches-action@master
        with:
          upstream-url: "https://github.com/kcat/openal-soft"
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
