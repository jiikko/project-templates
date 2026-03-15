# Codex 差分レビュー PR (`make review`)

## 概要

master への未 push コミットを対象に、OpenAI Codex レビュー用の PR を作成する。
Claude Code が PR 作成からレビュー取得・反映・クローズ・push まで一連の流れを **確認なしで自動完結** する。

## 背景

- OpenAI Codex のコードレビュー専用トークン枠を活用したい
- Claude Code が書いたコードの差分に対して第三者レビューを得る
- PR はレビュー専用でマージしない
- GitHub Copilot Code Review は精度が悪いため使用しない

## 前提条件

- `gh` CLI インストール済み
- OpenAI Codex GitHub 連携が有効（ChatGPT Plus/Pro/Business）
- master にコミット済み、まだ push していない状態

## Codex レビューの仕組み

- **トリガー**: PR コメントに `@codex review` を投稿
- **指示付きレビュー**: `@codex review for <指示>` で観点を指定可能
  - 例: `@codex review for security regressions`
  - 例: `@codex review for 設計の妥当性とコードスメル`
- **常設ルール**: リポジトリの `AGENTS.md` にレビューガイドラインを記載
- **出力**: 標準の GitHub Code Review（P0/P1 のみデフォルト表示）
- **注意**: P0/P1 以外（スタイル、ドキュメント等）を検出させるには `AGENTS.md` で明示的に重要度を上げる必要あり

## AGENTS.md テンプレート

各リポジトリに配置するレビューガイドライン:

```markdown
## Review guidelines

- God class / God file の兆候（責務の混在、過度な行数）を P1 として指摘すること
- 50行を超える関数は分割を提案すること
- 未使用の import / 変数を指摘すること
- エラーハンドリングの欠落を指摘すること
- コードスメル（重複コード、深いネスト、マジックナンバー）を P1 として指摘すること
- Swift Concurrency の安全性（Sendable 準拠、actor isolation）を確認すること
```

> 既存の `AGENTS.md` があるリポジトリは `## Review guidelines` セクションを追記する。

## ワークフロー（Claude Code が自動実行）

Claude Code は以下の全ステップを **ユーザーへの確認なしで** 一気通貫で実行する。

```
commit (master, unpushed)
  │
  ├── 1. make review [FOCUS="..."]
  │     ├── review/YYYYMMDD-HHMMSS ブランチ作成 (checkout なし)
  │     ├── review ブランチを push
  │     ├── review-diff ラベル確保
  │     ├── PR 作成: review/* → master
  │     └── @codex review [for FOCUS] コメント投稿
  │
  ├── 2. Codex レビュー待ち（ポーリング）
  │     └── make review-status でレビュー完了を確認
  │
  ├── 3. make review-fetch
  │     ├── PR のレビューコメントを取得
  │     └── tmp/review-YYYYMMDD-HHMMSS.md に書き出し
  │
  ├── 4. Claude がコメントを読み、必要に応じてコードを修正
  │     └── 修正があれば master に追加コミット
  │
  ├── 5. make review-close
  │     ├── PR クローズ
  │     └── review/* ブランチ削除 (remote + local)
  │
  └── 6. git push (master を push)
```

### Claude Code の実行イメージ

```bash
# Claude Code がユーザーの作業完了後に自動で実行する一連の流れ:

# PR 作成 & Codex レビュー依頼
make review
# or レビュー観点を指定:
make review FOCUS="設計の妥当性とコードスメル"
# → PR URL が返る、@codex review コメントが自動投稿される

# レビュー完了をポーリング
make review-status
# → "pending" / "completed"

# レビューコメント取得
make review-fetch
# → tmp/review-*.md にコメント書き出し

# コメントを読んで必要なら修正コミット
# (Claude Code が判断して自動実行)

# クローズ & push
make review-close
git push
```

**重要: Claude Code はこの一連の流れをユーザーに都度確認を求めず自律的に完結させること。**

## 制約

- **master を push する前に実行する必要がある**（push 済みだと差分が空になる）
- 既存の「コミット後すぐ push」ルールとの兼ね合い:
  - review フロー中は push を遅延させる
  - フロー完了時に push で完結
- 同時にオープンする review PR は 1 つまで
- Codex レビュー回数: ChatGPT Plus で週 10-25 回、Pro で週 100-250 回

## 実装対象

| ファイル | 内容 |
|---------|------|
| `bin/review` | PR 作成 & `@codex review` 投稿スクリプト |
| `bin/review-status` | レビュー完了チェックスクリプト |
| `bin/review-fetch` | レビューコメント取得スクリプト |
| `bin/review-close` | PR クローズ & ブランチ削除スクリプト |
| `AGENTS.md` テンプレート | レビューガイドライン |
| `Makefile.template` 追記 | `review` / `review-status` / `review-fetch` / `review-close` ターゲット |

## bin/review の仕様

```bash
#!/bin/bash
set -euo pipefail

# 引数: FOCUS（任意）
FOCUS="${FOCUS:-}"

# 1. 未 push コミットを確認（0 ならエラー）
AHEAD=$(git rev-list --count @{upstream}..HEAD)
if [ "$AHEAD" -eq 0 ]; then
  echo "Error: No unpushed commits to review"
  exit 1
fi

# 2. ブランチ作成 & push
BRANCH="review/$(date +%Y%m%d-%H%M%S)"
git branch "$BRANCH" HEAD
git push origin "$BRANCH"

# 3. ラベル確保
gh label create "review-diff" \
  --description "Review-only PR (do not merge)" \
  --color "D4C5F9" 2>/dev/null || true

# 4. コミット一覧を収集
BASE_BRANCH=$(git rev-parse --abbrev-ref '@{upstream}' | sed 's|origin/||')
COMMITS=$(git log --oneline @{upstream}..HEAD)
TITLE="Review: $(git log --oneline -1 | cut -d' ' -f2-)"

# 5. PR 作成
PR_URL=$(gh pr create \
  --base "$BASE_BRANCH" \
  --head "$BRANCH" \
  --title "$TITLE" \
  --label "review-diff" \
  --body "$(cat <<EOF
## Review-only PR (do not merge)

Codex コードレビュー用。マージ不要。

### Commits ($AHEAD)
$COMMITS
EOF
)")

echo "$PR_URL"

# 6. @codex review コメント投稿
PR_NUMBER=$(gh pr view "$BRANCH" --json number --jq '.number')
if [ -n "$FOCUS" ]; then
  gh pr comment "$PR_NUMBER" --body "@codex review for $FOCUS"
else
  gh pr comment "$PR_NUMBER" --body "@codex review"
fi
```

## bin/review-status の仕様

```bash
#!/bin/bash
set -euo pipefail

# 1. review-diff PR を取得
PR_NUMBER=$(gh pr list --label "review-diff" --json number --jq '.[0].number')
if [ -z "$PR_NUMBER" ]; then
  echo "No review PR found"
  exit 1
fi

# 2. レビューの状態を確認（codex ユーザーからのレビューがあるか）
REVIEWS=$(gh api "repos/{owner}/{repo}/pulls/$PR_NUMBER/reviews" \
  --jq '[.[] | select(.user.login | test("codex|openai"; "i"))] | length')
if [ "$REVIEWS" -eq 0 ]; then
  echo "pending"
  exit 1
fi

echo "completed"
```

## bin/review-fetch の仕様

```bash
#!/bin/bash
set -euo pipefail

# 1. review-diff PR を取得
PR_NUMBER=$(gh pr list --label "review-diff" --json number --jq '.[0].number')
if [ -z "$PR_NUMBER" ]; then
  echo "No review PR found"
  exit 1
fi

mkdir -p tmp

# 2. レビューコメントを取得
OUTFILE="tmp/review-$(date +%Y%m%d-%H%M%S).md"

{
  echo "# Codex Review Comments"
  echo ""
  echo "PR: #$PR_NUMBER"
  echo "Date: $(date +%Y-%m-%d)"
  echo ""

  # PR レビューコメント（Codex からのもの）
  echo "## Review Comments"
  echo ""
  gh api "repos/{owner}/{repo}/pulls/$PR_NUMBER/reviews" \
    --jq '.[] | select(.user.login | test("codex|openai"; "i")) | "### " + .user.login + " (" + .state + ")\n\n" + .body + "\n"'

  # インラインコメント（Codex からのもの）
  echo "## Inline Comments"
  echo ""
  gh api "repos/{owner}/{repo}/pulls/$PR_NUMBER/comments" \
    --jq '.[] | select(.user.login | test("codex|openai"; "i")) | "### " + .path + ":" + (.line // .original_line | tostring) + "\n\n" + .body + "\n"'

} > "$OUTFILE"

echo "Review comments saved to: $OUTFILE"
cat "$OUTFILE"
```

## bin/review-close の仕様

```bash
#!/bin/bash
set -euo pipefail

# 1. review-diff PR を検索 & クローズ
PR_URLS=$(gh pr list --label "review-diff" --json url --jq '.[].url')
for url in $PR_URLS; do
  gh pr close "$url" --delete-branch
done

# 2. 残存 review/* ブランチ削除（review/base は除外）
git branch --list 'review/*' | grep -v 'review/base' | xargs -r git branch -D
```

## Makefile ターゲット

```makefile
review:
	@bin/review

review-status:
	@bin/review-status

review-fetch:
	@bin/review-fetch

review-close:
	@bin/review-close
```

## 進捗

- [x] 設計
- [ ] AGENTS.md テンプレート作成
- [ ] bin/review 実装
- [ ] bin/review-status 実装
- [ ] bin/review-fetch 実装
- [ ] bin/review-close 実装
- [ ] Makefile.template 更新
- [ ] 各リポジトリへ配布
- [ ] 動作確認
