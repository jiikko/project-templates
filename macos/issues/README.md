# Issue追跡

## ファイル命名規則

番号付きIssueには種類を示すプレフィックスを付与:

| プレフィックス | 種類 | 説明 | 例 |
|---------------|------|------|-----|
| `feat-` | 機能追加 | 新機能、機能拡張 | `001-feat-add-dark-mode.md` |
| `bug-` | バグ修正 | 不具合、クラッシュ、エラー | `002-bug-crash-on-launch.md` |
| `refactor-` | リファクタリング | コード品質改善、整理 | `003-refactor-cleanup-viewmodel.md` |
| `perf-` | パフォーマンス | 速度改善、最適化 | `004-perf-reduce-memory-usage.md` |
| `ux-` | UX改善 | 操作性、使い勝手の改善 | `005-ux-improve-onboarding.md` |
| `human-` | 人間タスク | 動作確認・目視レビュー・判断待ち（`期限:` 必須） | `006-human-verify-export.md` |
| `chore-` | 雑務 | CI 設定、ツール整備 | `007-chore-fix-ci-runner.md` |
| `risk-` | リスク記録 | 未確認リスク・観測ポイント | `008-risk-undo-race.md` |
| `task-` | 引き継ぎ | 移行・引き継ぎ等の作業トラッカー | `009-task-cdn-migration.md` |

### その他のルール

- `NNN-{prefix}-description.md`: 番号 + プレフィックス + 内容（必須形式）
- `*.md`: 番号なしIssue（タスクリスト、改善提案、ドキュメント）
- `done/`: 完了したIssue
- `pending/`: 着手未定・保留中のIssue
- `epic/<NNN>/`: 大きな issue (#NNN) を分割した子 issue の置き場（番号付き issue を含むため**採番の走査対象**）
- `next/`: 次に着手する issue の置き場（同上）

### Lint

```bash
make lint-issues  # 命名規則チェック
```

## ディレクトリ構造

```
issues/
├── README.md          # このファイル（ルール・テンプレート）
├── INDEX.md           # Issue索引（アプリ固有・要更新）
├── 001-feat-first-feature.md
├── 002-bug-some-bug.md
├── done/
│   └── 001-feat-completed-feature.md
└── pending/
    └── 099-refactor-deferred-topic.md
```

## 運用

- `issues/` 直下: 現在追跡中のIssue
- `issues/done/`: 完了したIssue
- `issues/pending/`: 着手未定・保留中のIssue
- `issues/epic/<NNN>/`: epic の子 issue
- `issues/next/`: 次に着手する issue
- **human タスク**（人間しかできない作業: 動作確認・目視レビュー・外部サービス操作・判断待ち）は
  `issues/` **直下**に `NNN-human-<スラッグ>.md` で起票し、本文に `期限: YYYY-MM-DD` を書く（必須）。
  済んだら `issues/done/` へ移動する（既読ヘッダーは使わない — 位置がステータスの正本）。
  チャットで動作確認を依頼して流すのではなく、ここに起こす。期限切れはセッション開始時に
  dotfiles の `human-tasks-due.sh` hook が注入する（hook の走査対象が issues/ 直下 + pending/
  のため、サブディレクトリには置かない）

番号付き issue はどのサブディレクトリにも置かれうるため、採番時は必ず全サブディレクトリを走査すること。

- 新規Issueは原則 `issues/` 直下に作成する
- 完了したら `issues/done/` に移動する
- 保留にしたものだけ `issues/pending/` に移動する

## Issue索引

Issue一覧・優先度・ステータスはアプリ固有の `INDEX.md` で管理する。

> **新しいissueを追加・完了・ステータス変更したら `INDEX.md` を必ず更新すること。**

`INDEX.md` の運用原則:
- **トークン消費の抑制**: ファイルが肥大化しないよう、各 Issue の詳細な概要や解説は含めない（トークン節約のため）。
- **純粋なインデックス**: 番号、タイトル、ステータス、優先度のテーブルのみで構成する。
- **セクション分け**: Open Issues を種類別に整理する、Done 一覧は書かない。

`INDEX.md` には以下を記載する:
- カテゴリ別の Open Issues 一覧（番号・タイトル・優先度・状態）

## テンプレート

```markdown
# Issue Title

**作成日**: YYYY-MM-DD

## 概要
問題や機能の簡潔な説明

## 詳細
- 詳細な説明
- 再現手順（バグの場合）
- 期待される動作

## 対応方針
実装アプローチの説明

## 関連ファイル
- `path/to/file.swift`

## 進捗
- [ ] 調査
- [ ] 実装
- [ ] テスト
- [ ] レビュー
```

---

> このファイルは `project-templates` で統合管理されています。直接変更せず、`project-templates/macos/issues/README.md` を変更してください。
