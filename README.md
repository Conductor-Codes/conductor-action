# Conductor Code Review GitHub Action

This GitHub Action automatically reviews Pull Requests using Conductor.

## Features

- Automatic code reviews on new PRs
- Skips draft PRs
- Runs on comments with `@conductor review`
- Avoids duplicate reviews
- No configuration needed in your codebase

## Usage

Add this GitHub Action to your repository by creating a file at `.github/workflows/conductor-review.yml` with the following content:

```yaml
name: Conductor Code Review

on:
  pull_request:
    types: [opened, reopened, ready_for_review]
  issue_comment:
    types: [created]

jobs:
  code-review:
    # Don't run on draft PRs
    # Only run on issue comments that include '@conductor review'
    if: github.event.pull_request.draft == false || ${{ github.event_name != 'issue_comment' || contains(github.event.comment.body, '@conductor review') }}
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0  # Get complete history for better context
      
      # Run Conductor code review
      - name: Conductor Code Review
        uses: conductor-codes/conductor-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

## Configuration

The action requires the following inputs:

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `github_token` | GitHub token for API access | Yes | `${{ github.token }}` |

## How It Works

The action:
1. Runs on new PRs, reopened PRs, and when a PR changes from draft to ready
2. Also runs when someone comments with `@conductor review`
3. Analyzes the code changes in the PR
4. Posts a review comment with findings

## Zero Configuration Setup

This GitHub Action is designed to work immediately with no configuration needed:

- No need to modify your codebase
- No configuration files to create
- Works out of the box with the standard GitHub token

## Example Review

When a PR is opened, Conductor will automatically analyze the code and post a comment like:

```
# Conductor Code Review

This is a code review.

## Diff Stats
Diff size: 1253 characters
```

## License

See [LICENSE](LICENSE) file for details.

## Support

For issues with this action, please open an issue on the GitHub repository.
