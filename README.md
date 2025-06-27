<!-- BEGIN_ACTION_DOCS -->

# github-actions-bot-signed-commit
Enable bots to sign commits in GitHub Actions

# inputs
| Title | Required | Type | Default| Description |
|-----|-----|-----|-----|-----|
| TOKEN | False | string | `${{ github.token }}` | If signing commits for Github Apps, provide the App token. If not provided, the action will automatically use GITHUB_TOKEN. |
| TARGET_OWNER | False | string | `${{ github.repository_owner }}` | The repository owner (user or org) |
| TARGET_REPO | False | string | `${{ github.event.repository.name }}` | The repository name where the commits will be signed and pushed to. |
| TARGET_REF | False | string |  | The branch where the signed commits will be pushed to. |
| FILE_LIST | False | string |  | The path to a text file containing the list of file paths to be committed. E.g.: subdir/file_paths.txt |

# outputs
| Title | Description | Value |
|-----|-----|-----|
|sha | SHA of the verified commit |  `${{ steps.sign_and_push.outputs.sha }}` | 
<!-- END_ACTION_DOCS -->


# permissions

- GITHUB_TOKEN  
`contents: write`

- Github App  
`contents: write`, `metadata: read`

# Usage

## Basic (`GITHUB_TOKEN`)

```yaml
jobs:
  test-action:
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
```

## Basic (Github App)

```yaml
jobs:
  test-action:
    runs-on: ubuntu-latest
    name: Bot verified commit
    steps:
      - name: Get GitHub App Token
        uses: actions/create-github-app-token@v2
        id: app-token
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}
          owner: ${{ github.repository_owner }}
          
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.ref }}
          fetch-depth: 0 # required if you want to generate a list of files to be committed by comparing branches
          token: ${{ steps.app-token.outputs.token }}

      - name: Create list of files to be signed
        run: |
          git fetch origin develop
          echo "$(git diff --name-only origin/develop...HEAD)" > file_list.txt
          cat file_list.txt

      - name: Sign commits and push to develop branch
        uses: magmanu/github-actions-bot-signed-commit@<sha> 
        with:
          APP_TOKEN: ${{ steps.app-token.outputs.token }}
          TARGET_REF: develop
          FILE_LIST: file_list.txt
```
