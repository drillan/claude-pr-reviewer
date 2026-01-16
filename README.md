# Claude PR Reviewer

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Claude%20PR%20Reviewer-blue?logo=github)](https://github.com/marketplace/actions/claude-pr-reviewer)
[![GitHub release](https://img.shields.io/github/v/release/drillan/claude-pr-reviewer)](https://github.com/drillan/claude-pr-reviewer/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

[日本語](README.ja.md)

A GitHub Action for automated PR reviews using Claude AI. It leverages the [pr-review-toolkit](https://github.com/anthropics/claude-code) plugin to provide comprehensive code reviews.

## Features

- **Code Quality Check**: Review coding style and best practices
- **Test Coverage Analysis**: Evaluate test completeness and quality
- **Error Handling Detection**: Identify silent errors and improper error handling
- **Type Design Analysis**: Assess type definition quality and encapsulation
- **Comment Accuracy Verification**: Check consistency between code comments and implementation
- **Interactive Review**: Mention `@claude` in PR comments to request additional reviews or ask questions

## Usage

### Basic Workflow (OAuth Token)

```yaml
name: Claude PR Review

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: read
  pull-requests: write
  issues: write
  id-token: write

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: drillan/claude-pr-reviewer@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

### Basic Workflow (API Key)

```yaml
name: Claude PR Review

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: read
  pull-requests: write
  issues: write

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: drillan/claude-pr-reviewer@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Interactive Review with @claude Mention

```yaml
name: Claude PR Review

on:
  pull_request:
    types: [opened, synchronize]
  issue_comment:
    types: [created]

permissions:
  contents: read
  pull-requests: write
  issues: write
  id-token: write

jobs:
  review:
    runs-on: ubuntu-latest
    if: |
      github.event_name == 'pull_request' ||
      (github.event_name == 'issue_comment' &&
       github.event.issue.pull_request &&
       contains(github.event.comment.body, '@claude') &&
       github.event.comment.user.type != 'Bot')
    steps:
      - uses: drillan/claude-pr-reviewer@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

## Input Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `claude_code_oauth_token` | No* | - | Claude Code OAuth token |
| `anthropic_api_key` | No* | - | Anthropic API key |
| `anthropic_model` | No | `claude-opus-4-5-20251101` | Claude model to use |
| `review_language` | No | `English` | Language for review comments |
| `custom_prompt` | No | - | Additional custom instructions |
| `allowed_tools` | No | (see below) | Tools allowed for Claude |

\* Either `claude_code_oauth_token` or `anthropic_api_key` is required. If both are provided, `claude_code_oauth_token` takes precedence.

### Available Models

| Model | Model ID | Characteristics |
|-------|----------|-----------------|
| Claude Opus 4.5 | `claude-opus-4-5-20251101` | Highest quality (default) |
| Claude Sonnet 4.5 | `claude-sonnet-4-5-20250929` | Fast and balanced |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | Fastest and lowest cost |

### Default allowed_tools

```
Bash(gh pr comment:*),Bash(gh pr review:*),Bash(gh pr diff:*),Bash(gh pr view:*),Read,Grep,Glob
```

## Authentication Setup

Two authentication methods are available:

### Option 1: Claude Code OAuth Token (Recommended)

1. Visit [Claude Code](https://claude.ai/claude-code)
2. Obtain an OAuth token
3. Add to GitHub Secrets:
   - Go to repository Settings > Secrets and variables > Actions
   - Click "New repository secret"
   - Name: `CLAUDE_CODE_OAUTH_TOKEN`
   - Value: Your OAuth token

```yaml
- uses: drillan/claude-pr-reviewer@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

### Option 2: Anthropic API Key

1. Visit [Anthropic Console](https://console.anthropic.com/)
2. Obtain an API key
3. Add to GitHub Secrets:
   - Go to repository Settings > Secrets and variables > Actions
   - Click "New repository secret"
   - Name: `ANTHROPIC_API_KEY`
   - Value: Your API key

```yaml
- uses: drillan/claude-pr-reviewer@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

## Customization Examples

### Review in Japanese

```yaml
- uses: drillan/claude-pr-reviewer@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    review_language: Japanese
```

### Use Faster Model

```yaml
- uses: drillan/claude-pr-reviewer@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    anthropic_model: claude-sonnet-4-5-20250929
```

### Add Custom Prompt

```yaml
- uses: drillan/claude-pr-reviewer@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    custom_prompt: |
      Pay special attention to security aspects.
      Also evaluate performance implications.
```

### Avoid Repeated Reviews

By default, reviews are triggered on every push (`synchronize` event). To review only on PR creation and re-review on demand via `@claude` mention:

```yaml
on:
  pull_request:
    types: [opened]  # Remove synchronize
  issue_comment:
    types: [created]
```

## Required Permissions

Add the following permissions to your workflow:

```yaml
permissions:
  contents: read        # Read repository contents
  pull-requests: write  # Post comments on PRs
  issues: write         # Issue comments (for @claude mentions)
  id-token: write       # OIDC authentication
```

## License

MIT License
