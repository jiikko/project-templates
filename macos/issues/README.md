# Issue追跡

## ファイル命名規則

- 番号付き: `001-feature-name.md`, `002-bug-description.md`
- カテゴリプレフィックス: `PERF-001-optimization.md`, `UI-002-layout-fix.md`

## ディレクトリ構造

```
issues/
├── 001-current-issue.md
├── 002-another-issue.md
└── done/
    ├── 001-completed-issue.md
    └── 002-another-completed.md
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
