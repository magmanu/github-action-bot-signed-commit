# github-action-bot-signed-commit
Enable signed commits from bots

<!-- BEGIN_ACTION_DOCS -->

# github-actions-bot-signed-commit
Enable bots to sign commits in GitHub Actions

# inputs
| Title | Required | Type | Default| Description |
|-----|-----|-----|-----|-----|
| APP_ID | False | string |  | If signing commits using Github Apps, provide the App ID |
| APP_PRIVATE_KEY | False | string |  | If signing commits using Github Apps, provide the private key |
| DESTINATION_REF | False | string | `${{ github.event.pull_request.head.ref || github.ref_name || 'main' }}` | The branch where the signed commits will be pushed to |
| FILE_LIST | False | string |  | The path to any bash script that will be run to sign the commits. Must be in the origin ref. E.g.: my_dir/script.sh |
<!-- END_ACTION_DOCS -->
