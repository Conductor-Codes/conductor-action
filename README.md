# Conductor Code Review GitHub Action

This GitHub Action automatically reviews Pull Requests using Conductor.

## Features

- Automatic code reviews on new PRs
- Skips draft PRs
- Runs on comments with `@conductor-codes review`
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
    if: ${{ github.event_name != 'issue_comment' || contains(github.event.comment.body, '@conductor-codes review') }}
    runs-on: ubuntu-latest
    
    steps:
      - name: Determine PR ref
        if: github.event_name == 'issue_comment'
        id: pr
        uses: actions/github-script@v6
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const pr = await github.rest.pulls.get({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.issue.number
            });
            core.setOutput('head_ref', pr.data.head.ref);

      - name: Checkout code
        uses: actions/checkout@v3
        with:
          ref: ${{ github.head_ref || steps.pr.outputs.head_ref }}
          fetch-depth: 0  # Get complete history for better context
      
      - name: Conductor Code Review
        uses: conductor-codes/conductor-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          conductor_llm_config: ${{ secrets.CONDUCTOR_LLM_CONFIG }}
          conductor_api_key: ${{ secrets.CONDUCTOR_API_KEY }}
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
  "api_key": "..."
}
```
2. Create a repository secret named `CONDUCTOR_API_KEY` and enter in your Api Key for Conductor.
3. Done! No need to modify your codebase.

_Note: If you're using AWS Bedrock read the AWS Bedrock section below._


### AWS Bedrock Setup

If you're using AWS Bedrock to authenticate with your LLM you can do so in one of two ways:

1. Through an IAM role (recommended)
2. Through a user with a key & secret

#### IAM Role Configuration (recommended)

1. Create a trust policy for GitHub Actions using OIDC federation.
2. Create an IAM role with `bedrock:InvokeModel` and `bedrock:InvokeModelWithResponseStream` permissions for your chosen model.
3. Add the `aws-actions/configure-aws-credentials@v4` step to your GitHub Action created above (this should be added above the `Conductor Code Review` step):
```yaml
- name: Configure AWS credentials via OIDC
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::<account-id>:role/<role-name>
    aws-region: <region>
```
4. Set only the `api_type` and `model` in the `CONDUCTOR_LLM_CONFIG` repository secret:
```json
{
  "api_type": "bedrock",
  "model": "us.anthropic.claude-3-7-sonnet-20250219-v1:0"
}
```

#### Key & Secret Configuration

1. Set the AWS Access Key & AWS Secret Key in the `CONDUCTOR_LLM_CONFIG` repository secret.
```json
{
  "api_type": "bedrock",
  "model": "us.anthropic.claude-3-7-sonnet-20250219-v1:0",
  "aws_access_key": "...",
  "aws_secret_key": "...",
  "aws_region": "..."
}
```

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
