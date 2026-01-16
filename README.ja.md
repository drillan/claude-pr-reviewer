# Claude PR Reviewer

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Claude%20PR%20Reviewer-blue?logo=github)](https://github.com/marketplace/actions/claude-pr-reviewer)
[![GitHub release](https://img.shields.io/github/v/release/drillan/claude-pr-reviewer)](https://github.com/drillan/claude-pr-reviewer/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

[English](README.md)

Claude AI を使用した自動 PR レビュー GitHub Action です。[pr-review-toolkit](https://github.com/anthropics/claude-code) プラグインを活用し、包括的なコードレビューを提供します。

## 機能

- **コード品質チェック**: コーディングスタイル、ベストプラクティスの確認
- **テストカバレッジ分析**: テストの網羅性と品質の評価
- **エラーハンドリング検出**: サイレントエラーや不適切なエラー処理の発見
- **型設計分析**: 型定義の品質とカプセル化の評価
- **コメント正確性検証**: コードコメントと実装の整合性チェック
- **対話的レビュー**: PR コメントで `@claude` とメンションすると追加レビューや質問への回答を取得

## 使用方法

### 基本的なワークフロー（OAuth Token）

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

### 基本的なワークフロー（API Key）

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

### @claude メンションによる対話的レビュー

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

## 入力パラメータ

| パラメータ | 必須 | デフォルト | 説明 |
|-----------|------|-----------|------|
| `claude_code_oauth_token` | No* | - | Claude Code OAuth トークン |
| `anthropic_api_key` | No* | - | Anthropic API キー |
| `anthropic_model` | No | `claude-opus-4-5-20251101` | 使用する Claude モデル |
| `review_language` | No | `English` | レビューコメントの言語 |
| `custom_prompt` | No | - | カスタム指示（追加プロンプト） |
| `allowed_tools` | No | (後述) | Claude が使用可能なツール |

\* `claude_code_oauth_token` または `anthropic_api_key` のどちらか一方が必須です。両方が指定された場合は `claude_code_oauth_token` が優先されます。

### 利用可能なモデル

| モデル | モデルID | 特徴 |
|--------|----------|------|
| Claude Opus 4.5 | `claude-opus-4-5-20251101` | 最高品質（デフォルト） |
| Claude Sonnet 4.5 | `claude-sonnet-4-5-20250929` | 高速・バランス型 |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | 最速・低コスト |

### allowed_tools のデフォルト値

```
Bash(gh pr comment:*),Bash(gh pr review:*),Bash(gh pr diff:*),Bash(gh pr view:*),Read,Grep,Glob
```

## 認証設定

認証方法は2種類から選択できます：

### 方法 1: Claude Code OAuth Token（推奨）

1. [Claude Code](https://claude.ai/claude-code) にアクセス
2. OAuth トークンを取得
3. GitHub Secrets に登録：
   - リポジトリの Settings > Secrets and variables > Actions
   - New repository secret をクリック
   - Name: `CLAUDE_CODE_OAUTH_TOKEN`
   - Value: 取得したトークンを入力

```yaml
- uses: drillan/claude-pr-reviewer@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

### 方法 2: Anthropic API Key

1. [Anthropic Console](https://console.anthropic.com/) にアクセス
2. API キーを取得
3. GitHub Secrets に登録：
   - リポジトリの Settings > Secrets and variables > Actions
   - New repository secret をクリック
   - Name: `ANTHROPIC_API_KEY`
   - Value: 取得した API キーを入力

```yaml
- uses: drillan/claude-pr-reviewer@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

## カスタマイズ例

### 日本語でレビュー

```yaml
- uses: drillan/claude-pr-reviewer@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    review_language: Japanese
```

### 高速モデルを使用

```yaml
- uses: drillan/claude-pr-reviewer@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    anthropic_model: claude-sonnet-4-5-20250929
```

### カスタムプロンプトを追加

```yaml
- uses: drillan/claude-pr-reviewer@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    custom_prompt: |
      特にセキュリティ面を重点的にチェックしてください。
      パフォーマンスへの影響も評価してください。
```

## 必要な権限

ワークフローに以下の権限を設定してください：

```yaml
permissions:
  contents: read        # リポジトリの読み取り
  pull-requests: write  # PR へのコメント投稿
  issues: write         # Issue コメント（@claude 対応時）
  id-token: write       # OIDC 認証
```

## ライセンス

MIT License
