# Codex 全ファイルレビュー PR (`make review-full`)

## 概要

orphan ブランチを使い、全ファイルを diff として表示する PR を作成して OpenAI Codex に全体レビューさせる。
`@codex review for <指示>` でレビュー観点を自由に指定できる。

## 背景

- 差分レビュー (001) は直近の変更のみが対象
- プロジェクト全体の品質チェック・設計レビューには全ファイルレビューが必要
- push 済みかどうかに関係なく実行できる
- GitHub Copilot Code Review は精度が悪いため使用しない

## 前提条件

- `gh` CLI インストール済み
- OpenAI Codex GitHub 連携が有効（ChatGPT Plus/Pro/Business）

## Codex レビューの仕組み

- **トリガー**: PR コメントに `@codex review` を投稿
- **指示付きレビュー**: `@codex review for <指示>` で観点を指定可能
- **常設ルール**: リポジトリの `AGENTS.md` にレビューガイドラインを記載
- **出力**: 標準の GitHub Code Review（P0/P1 のみデフォルト表示）
- **P0/P1 以外を検出させる**: `AGENTS.md` で重要度を明示的に上げる必要あり

## レビュー観点の指定

第1引数でレビュー観点を自由に指定できる。**指定なしの場合は設計レビューがデフォルト。**

```bash
# デフォルト（設計レビュー）
make review-full
# → @codex review for アーキテクチャの妥当性、責務分離、依存方向の正しさ、God class の兆候

# コードスメル探索
make review-full "コードスメル、重複コード、過度な複雑性"

# セキュリティ監査
make review-full "security vulnerabilities, input validation, injection risks"

# パフォーマンス
make review-full "パフォーマンスボトルネック、不要な再描画、メモリリーク"

# Swift 特有
make review-full "Swift Concurrency safety, Sendable conformance, retain cycles"
```

引数は `@codex review for $1` としてそのまま PR コメントに投稿される。

### AGENTS.md との使い分け

| | AGENTS.md | FOCUS 引数 |
|--|-----------|-----------|
| **用途** | 常設のレビュールール | 都度の観点指定 |
| **永続性** | リポジトリに永続 | 一回限り |
| **例** | 「God class を P1 として指摘」 | 「設計の妥当性を重点的に」 |
| **併用** | 両方同時に適用される | AGENTS.md + FOCUS が合わさる |

## ワークフロー

```
make review-full ["レビュー観点"]
  │
  ├── 1. orphan ブランチ `review/base` 作成（空コミット 1 つ）
  │     ※ 既存なら再利用
  ├── 2. `review/full-YYYYMMDD-HHMMSS` ブランチを HEAD から作成
  ├── 3. review/full-* ブランチを push
  ├── 4. review-full ラベル確保
  ├── 5. PR 作成: review/full-* → review/base
  │     └── 全ファイルが「追加」として diff に出る
  └── 6. @codex review [for FOCUS] コメント投稿

Codex がレビュー（AGENTS.md + FOCUS に従う）

make review-fetch
  └── tmp/review-*.md にコメント書き出し

make review-close
  ├── PR クローズ
  └── review/* ブランチ削除 (review/base は保持)
```

## 特徴

- **push 状態に依存しない** — master が push 済みでも実行可能
- **全ファイルが diff に出る** — Codex がプロジェクト全体を見れる
- **レビュー観点を自由に指定** — FOCUS 引数で都度カスタマイズ
- **冪等** — 毎回クリーンな状態から PR を作る
- `review-fetch` / `review-close` は 001（差分レビュー）と共通スクリプト

## 実装対象

| ファイル | 内容 |
|---------|------|
| `bin/review-full` | 全ファイルレビュー PR 作成 & `@codex review` 投稿スクリプト（第1引数: レビュー観点） |
| `bin/review-fetch` | 共通: レビューコメント取得（001 と共有） |
| `bin/review-close` | 共通: PR クローズ & ブランチ削除（001 と共有） |
| `AGENTS.md` テンプレート | レビューガイドライン（001 と共有） |
| `Makefile.template` 追記 | `review-full` ターゲット |

## bin/review-full の仕様

```bash
#!/bin/bash
set -euo pipefail

# 引数: レビュー観点（省略時は設計レビュー）
DEFAULT_FOCUS="アーキテクチャの妥当性、責務分離、依存方向の正しさ、God class の兆候"
FOCUS="${1:-$DEFAULT_FOCUS}"

# 1. orphan base ブランチを確保（リモートに存在しなければ作成）
if ! git ls-remote --heads origin review/base | grep -q review/base; then
  CURRENT_BRANCH=$(git branch --show-current)
  git checkout --orphan review/base
  git rm -rf . 2>/dev/null || true
  git commit --allow-empty -m "empty base for review PRs"
  git push origin review/base
  git checkout "$CURRENT_BRANCH"
else
  echo "review/base already exists on remote"
fi

# 2. レビューブランチ作成 & push
BRANCH="review/full-$(date +%Y%m%d-%H%M%S)"
git branch "$BRANCH" HEAD
git push origin "$BRANCH"

# 3. ラベル確保
gh label create "review-full" \
  --description "Review-only PR (do not merge)" \
  --color "D4C5F9" 2>/dev/null || true

# 4. ファイル数を収集
FILE_COUNT=$(git ls-files | wc -l | tr -d ' ')
TITLE="Full Review: $(basename $(git rev-parse --show-toplevel)) ($FILE_COUNT files)"

# 5. PR 作成
PR_URL=$(gh pr create \
  --base "review/base" \
  --head "$BRANCH" \
  --title "$TITLE" \
  --label "review-full" \
  --body "$(cat <<EOF
## Full Review PR (do not merge)

Codex 全ファイルレビュー用。マージ不要。

- Files: $FILE_COUNT
- HEAD: $(git rev-parse --short HEAD)
- Date: $(date +%Y-%m-%d)
EOF
)")

echo "$PR_URL"

# 6. @codex review コメント投稿
PR_NUMBER=$(gh pr view "$BRANCH" --json number --jq '.number')
gh pr comment "$PR_NUMBER" --body "@codex review for $FOCUS"
```

## Makefile ターゲット

```makefile
# 第1引数でレビュー観点を指定（省略時: 設計レビュー）
# 使い方: make review-full
#         make review-full FOCUS="コードスメル、重複コード"
review-full:
	@bin/review-full "$(FOCUS)"
```

## 注意事項

- 大規模リポジトリでは diff が巨大になる → Codex のトークン上限に注意
- `review/base` ブランチはリポジトリに永続的に残る（削除不要）
- `review-close` で `review/base` は削除しない（再利用するため）
- Codex レビュー回数: ChatGPT Plus で週 10-25 回、Pro で週 100-250 回
- 全ファイルレビューは回数消費が多くなる可能性あり、頻度に注意

## 進捗

- [x] 設計
- [ ] bin/review-full 実装
- [ ] bin/review-close に review/base 保護ロジック追加
- [ ] AGENTS.md テンプレート作成（001 と共有）
- [ ] Makefile.template 更新
- [ ] 各リポジトリへ配布
- [ ] 動作確認
