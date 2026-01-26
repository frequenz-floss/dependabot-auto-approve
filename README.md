# Dependabot Auto Manage Action

Automatically approves and merges Dependabot Pull Requests for production dependencies.

## Features

- ✅ Automatic approval of Dependabot PRs
- ✅ Automatic merge with configurable method (squash, merge, rebase)
- ✅ Filter by dependency type (production, development, all)
- ✅ Add labels to approved PRs
- ✅ Support for auto-merge (requires branch protection rules)
- ✅ Safe PR author verification

## Usage

### Basic usage

```yaml
name: Dependabot Auto Manage
on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: github.event.pull_request.user.login == 'dependabot[bot]' || github.event.pull_request.user.login == 'app/dependabot'
    steps:
      - name: Dependabot auto-manage
        uses: ad/dependabot-auto-approve@v1
```

### Advanced configuration

```yaml
name: Dependabot Auto Manage
on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: github.event.pull_request.user.login == 'dependabot[bot]' || github.event.pull_request.user.login == 'app/dependabot'
    steps:
      - name: Dependabot auto-manage
        uses: ad/dependabot-auto-approve@v1
        with:
          github-token: ${{ secrets.PAT_TOKEN }}    # Required for PR approval
          dependency-type: 'direct:production'      # or 'direct:development', 'all'
          merge-method: 'squash'                    # or 'merge', 'rebase'
          auto-merge: 'false'                       # 'true' for auto-merge (requires branch protection)
          add-label: 'dependabot-approved'          # label for approved PRs
```

## Input parameters

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `github-token` | GitHub token for API calls (use PAT for approval) | No | `github.token` |
| `dependency-type` | Type of dependencies to auto-merge | No | `direct:production` |
| `merge-method` | Merge method | No | `squash` |
| `auto-merge` | Enable auto-merge | No | `false` |
| `add-label` | Label for approved PRs | No | `dependabot-approved` |
| `ignore-regex-pr-title` | Regex patterns to ignore PRs (comma-separated) | No | |
| `pr-number` | PR number (for manual dispatch workflows) | No | |
| `pr-url` | PR URL (for manual dispatch workflows) | No | |

### Dependency types

- `direct:production` - production dependencies only
- `direct:development` - development dependencies only
- `all` - all dependencies

### Merge methods

- `squash` - squash commits into one
- `merge` - regular merge
- `rebase` - rebase and merge

### Ignore regex patterns

You can specify one or more regex patterns (comma-separated) to skip auto-approval for certain PRs. If any pattern matches the PR title, the PR will be ignored.

Examples:
- `ignore-regex-pr-title: '.*security.*,.*major.*'` - Skip PRs with "security" or "major" in the title
- `ignore-regex-pr-title: '^Bump.*from.*to.*'` - Skip PRs with specific version bump patterns
- `ignore-regex-pr-title: '.*breaking.*,.*deprecated.*'` - Skip PRs with breaking changes or deprecations

## Requirements

### Permissions

The workflow must have the following permissions:

```yaml
permissions:
  contents: write
  pull-requests: write
```

### GitHub Token

For **approval functionality**, you need a Personal Access Token (PAT) because `GITHUB_TOKEN` cannot approve PRs:

1. **Create a PAT** in GitHub Settings → Developer settings → Personal access tokens
2. **Grant `repo` scope** (for private repos) or `public_repo` scope (for public repos)
3. **Add it as a secret** in your repository: Settings → Secrets → Actions
4. **Use it in the workflow**:

```yaml
- uses: ad/dependabot-auto-approve@v1
  with:
    github-token: ${{ secrets.PAT_TOKEN }}  # Use your PAT secret name
```

> **Note**: The action will still work without a PAT, but won't be able to approve PRs. It will add labels and attempt to merge (if no approval is required).

### Auto-merge

To use `auto-merge: 'true'` you need to:

1. Configure branch protection rules in Settings → Branches
2. Enable "Allow auto-merge" for the repository
3. Set up required status checks

## Security

- ✅ Verifies that PR is created by Dependabot
- ✅ Supports both `dependabot[bot]` and `app/dependabot` formats
- ✅ Filters by dependency type
- ✅ Uses official `dependabot/fetch-metadata` action

## Troubleshooting

### "GitHub Actions is not permitted to approve pull requests"

This happens when using the default `GITHUB_TOKEN`. Solutions:

1. **Use a Personal Access Token**:
   ```yaml
   - uses: ad/dependabot-auto-approve@v1
     with:
       github-token: ${{ secrets.PAT_TOKEN }}
   ```

2. **Or disable approval and rely on auto-merge** (if branch protection allows):
   ```yaml
   - uses: ad/dependabot-auto-approve@v1
     with:
       auto-merge: 'true'  # Requires branch protection rules
   ```

### "Could not merge PR"

Common causes:
- Branch protection rules require approval
- Required status checks are failing
- Merge conflicts exist
- Repository doesn't allow auto-merge

### Action skips processing

Check that:
- PR is created by Dependabot (`dependabot[bot]` or `app/dependabot`)
- Dependency type matches your filter (`direct:production`, `direct:development`, or `all`)
- Workflow has proper permissions (`contents: write`, `pull-requests: write`)

## Workflow examples

### Production dependencies only

```yaml
name: Dependabot Auto Manage
on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: github.event.pull_request.user.login == 'dependabot[bot]' || github.event.pull_request.user.login == 'app/dependabot'
    steps:
      - uses: ad/dependabot-auto-approve@v1
        with:
          dependency-type: 'direct:production'
          merge-method: 'squash'
```

### All dependencies with auto-merge

```yaml
name: Dependabot Auto Manage
on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: github.event.pull_request.user.login == 'dependabot[bot]' || github.event.pull_request.user.login == 'app/dependabot'
    steps:
      - uses: ad/dependabot-auto-approve@v1
        with:
          dependency-type: 'all'
          auto-merge: 'true'
          add-label: 'auto-merged'
```

### Skip security and major updates

```yaml
name: Dependabot Auto Manage
on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: github.event.pull_request.user.login == 'dependabot[bot]' || github.event.pull_request.user.login == 'app/dependabot'
    steps:
      - uses: ad/dependabot-auto-approve@v1
        with:
          dependency-type: 'direct:production'
          ignore-regex-pr-title: '.*security.*,.*major.*'
          merge-method: 'squash'
```

### Manual trigger for specific PRs

```yaml
name: Dependabot Auto Manage
on: 
  pull_request:
  workflow_dispatch:
    inputs:
      pr_number:
        description: 'PR number to process'
        required: true
        type: number
      dependency_type:
        description: 'Dependency type to process'
        required: false
        default: 'direct:production'
        type: choice
        options:
          - 'direct:production'
          - 'direct:development'
          - 'all'

permissions:
  contents: write
  pull-requests: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: |
      (github.event_name == 'pull_request' && (github.event.pull_request.user.login == 'dependabot[bot]' || github.event.pull_request.user.login == 'app/dependabot')) ||
      (github.event_name == 'workflow_dispatch')
    steps:
      - name: Checkout (for manual dispatch)
        if: github.event_name == 'workflow_dispatch'
        uses: actions/checkout@v4
        
      - name: Check PR author (for manual dispatch)
        if: github.event_name == 'workflow_dispatch'
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          PR_AUTHOR=$(gh pr view ${{ github.event.inputs.pr_number }} --repo "${{ github.repository }}" --json author --jq '.author.login')
          if [[ "$PR_AUTHOR" != "dependabot[bot]" && "$PR_AUTHOR" != "app/dependabot" ]]; then
            echo "❌ PR #${{ github.event.inputs.pr_number }} is not from Dependabot (author: $PR_AUTHOR)"
            exit 1
          fi
          echo "✅ PR #${{ github.event.inputs.pr_number }} is from Dependabot"
          
      - name: Simulate PR event for manual dispatch
        if: github.event_name == 'workflow_dispatch'
        id: pr_info
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          # Get PR details and set outputs
          PR_DATA=$(gh pr view ${{ github.event.inputs.pr_number }} --repo "${{ github.repository }}" --json number,url,author)
          echo "pr_number=$(echo $PR_DATA | jq -r '.number')" >> $GITHUB_OUTPUT
          echo "pr_url=$(echo $PR_DATA | jq -r '.url')" >> $GITHUB_OUTPUT
          
      - uses: ad/dependabot-auto-approve@v1.3.2
        with:
          dependency-type: ${{ github.event_name == 'workflow_dispatch' && github.event.inputs.dependency_type || 'direct:production' }}
          merge-method: 'squash'
          add-label: ${{ github.event_name == 'workflow_dispatch' && 'manual-approval' || 'dependabot-approved' }}
          pr-number: ${{ github.event_name == 'workflow_dispatch' && steps.pr_info.outputs.pr_number || '' }}
          pr-url: ${{ github.event_name == 'workflow_dispatch' && steps.pr_info.outputs.pr_url || '' }}
```

## License

MIT License
