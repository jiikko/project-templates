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

### その他のルール

- `NNN-{prefix}-description.md`: 番号 + プレフィックス + 内容（必須形式）
- `*.md`: 番号なしIssue（タスクリスト、改善提案、ドキュメント）
- `done/`: 完了したIssue

### Lint

```bash
make lint-issues  # 命名規則チェック
```

## ディレクトリ構造

```
issues/
├── README.md
├── 001-feat-first-feature.md
├── 002-bug-some-bug.md
└── done/
    └── 001-feat-completed-feature.md
```

## テンプレート

```markdown
# Issue Title

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
