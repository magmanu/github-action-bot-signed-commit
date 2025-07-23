<!-- BEGIN_ACTION_DOCS -->

# github-actions-bot-signed-commit
Sign commits using either a GitHub App or GITHUB_TOKEN.  It's particularly helpful for repos/orgs that enforce signed commits.  At the moment, Github doesn't natively provide a verified badge for ` github.token` or Github Apps, only for users (humans/service accounts).

# inputs
| Title | Required | Type | Default| Description |
|-----|-----|-----|-----|-----|
| TOKEN | False | string | `${{ github.token }}` | If signing commits with Github Apps, provide the App token. Else, it defults to GITHUB_TOKEN. |
| TARGET_OWNER | False | string | `${{ github.repository_owner }}` | The repository owner (user/org) |
| TARGET_REPO | False | string | `${{ github.event.repository.name }}` | The repository where the signed commits will be pushed to. |
| TARGET_REF | False | string |  | The branch where the signed commits will be pushed to. |
| FILE_LIST | False | string |  | Path to a text file containing the list of file paths to be committed. E.g.: subdir/file_paths.txt |
| WORKING_DIR | False | string | `${{ github.workspace }}` | The working directory where the action will run. |
| IS_DRY_RUN | False | boolean |  | If set to true, the action will push the commits and leave them orphans (the head won't move.) |

# outputs
| Title | Description | Value |
|-----|-----|-----|
|sha | SHA of the verified commit |  `${{ steps.sign_and_push.outputs.sha }}` | 
<!-- END_ACTION_DOCS -->

# Permissions

| Type           | Scope                               |
| -------------- | ----------------------------------- |
| `GITHUB_TOKEN` | `contents: write`                   |
| Github App     | `contents: write`, `metadata: read` |

# Features

- Sign commits with a bot identity (`GITHUB_TOKEN` or Github App)
- Push to the same repository or to a different one, to an existing branch of your choice
- Specify a list of files to be committed
- Handle large binary or text files
- Dry run mode test the commit signing push without changing the target branch head

# Usage

## Basic: `GITHUB_TOKEN` - push to the same repository

```yaml
jobs:
  sign-commit:
    runs-on: ubuntu-latest
    name: Bot verified commit
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.ref }}
          fetch-depth: 0 # required if you want to generate a list of files to be committed by comparing branches
          token: ${{ github.token }}

      - name: Create list of files to be signed
        run: |
          git fetch origin develop
          echo "$(git diff --name-only origin/develop...HEAD)" > file_list.txt

      - name: Sign commits and push to develop branch
        uses: magmanu/github-actions-bot-signed-commit@<sha> 
        with:
          TARGET_REF: develop
          FILE_LIST: file_list.txt
          IS_DRY_RUN: true # the commit will be pushed, but the branch head won't move
```

## Advanced: Github App - push to a different repository


```yaml
jobs:
  deploy-to-gh-pages:
    runs-on: ubuntu-latest
    name: Bot verified commit
    steps:
      - uses: actions/create-github-app-token@v2
        id: app-token
        with:
          app-id: ${{ secrets.GH_APP_ID }}
          private-key: ${{ secrets.GH_APP_PEM }}
          owner: ${{ github.repository_owner }}

      - name: Checkout repo1
        uses: actions/checkout@v4
        with:
          repository: owner/source
          token: ${{ steps.app-token.outputs.token }}
          ref: ${{ github.ref }}
          fetch-depth: 2
          path: source

      - name: Checkout repo2
        uses: actions/checkout@v4
        with:
          repository: owner/deployment
          token: ${{ steps.app-token.outputs.token }}
          ref: gh-pages
          fetch-depth: 2
          path: deployment

      - name: Build
        env:
          MKDOCS_GIT_COMMITTERS_APIKEY: ${{ github.token }}
        run: |
          #!/usr/bin/env bash

          set -euo pipefail

          pip install -r source/requirements.txt

          # Make a few changes in files from repo1
          cd source 

          mkdocs build
          rm -rf .cache
          echo "Clear content of repo2 and move build files there"
          
          cd ../deployment
          git rm -rf .
          mv ../source/site/* .

          # Create the list of files to be committed
          echo $(git ls-files --others --exclude-standard) > .github/files.txt

      - name: Sign commits and push to develop branch
      # Depending on how many files you are committing, 
      # you will end up with multiple commits in your target branch.
      # In this case, your branch will point to the most recent commit 
      # once the process is finished.
        uses: magmanu/github-actions-bot-signed-commit@<sha> 
        with:
          APP_TOKEN: ${{ steps.app-token.outputs.token }}
          TARGET_REF: gh-pages
          FILE_LIST: .github/files.txt
          WORKING_DIR: deployment # the directory for "repo2"
```

