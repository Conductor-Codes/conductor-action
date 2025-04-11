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
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0  # Get complete history for better context
      
      - name: Conductor Code Review
        uses: conductor-codes/conductor-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          conductor_llm_config: ${{ secrets.CONDUCTOR_LLM_CONFIG }}
          conductor_api_key: ${{ CONDUCTOR_API_KEY }}
```

## Configuration

The action requires the following inputs:

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `github_token` | GitHub token for API access | Yes | `${{ github.token }}` |
| `conductor_llm_config` | JSON object containing credentials to authenticate with your LLM | Yes | N/A |
| `conductor_api_key` | API Key to authenticate with Conductor | Yes | N/A |


## Setup

This GitHub Action is designed to work with minimal configuration needed:

1. Create a repository secret named `CONDUCTOR_LLM_CONFIG` and input a JSON object containing your LLM authentication properties. Here's an example:
```json
{
  "api_type": "gemini",
  "model": "gemini-2.5-pro-exp-03-25",
  "api_key": "...",
}
```
2. Create a repository secret named `CONDUCTOR_API_KEY` and enter in your Api Key for Conductor.
3. Done! No need to modify your codebase.


## How It Works

The action:
1. Runs on new PRs, reopened PRs, and when a PR changes from draft to ready
2. Also runs when someone comments with `@conductor review`
3. Analyzes the code changes in the PR for performance and accessibility
4. Posts a review comment with findings


## Example Review

When a PR is opened, Conductor will automatically analyze the code and post a comment like:

```
This is a code review.

Review by Conductor
```

## License

See [LICENSE](LICENSE) file for details.

## Support

For issues with this action, please open an issue on the GitHub repository.
