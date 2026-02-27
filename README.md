# プロジェクトテンプレート

iOS/macOSアプリの共通ディレクトリ構造と運用フローのテンプレート集。

## 構造

```
project-templates/
├── bin/                # 共通スクリプト（全プラットフォーム共有）
│   └── lint-issues     # Issue ファイル名の命名規則チェック
├── macos/              # macOSアプリ用テンプレート
│   ├── README.md
│   ├── operations.md   # 運用ガイド
│   ├── Makefile.template
│   ├── project.yml.template
│   ├── .swiftlint.yml
│   ├── .github/workflows/swiftlint.yml
│   ├── releases/
│   ├── signing/
│   └── issues/
└── ios/                # iOSアプリ用テンプレート
    ├── README.md
    ├── operations.md   # 運用ガイド
    ├── Makefile.template
    ├── project.yml.template
    ├── .swiftlint.yml
    ├── .github/workflows/swiftlint.yml
    ├── releases/
    ├── signing/
    └── issues/
```

## 使い方

### 新規プロジェクト作成時

1. 対象プラットフォームのディレクトリをコピー
   ```bash
   cp -r macos/ /path/to/new-project/
   # または
   cp -r ios/ /path/to/new-project/
   ```

2. 共通スクリプトをコピー
   ```bash
   cp -r ../bin/ /path/to/new-project/bin/
   chmod +x /path/to/new-project/bin/*
   ```

3. `.template` ファイルをリネームしてプレースホルダーを置換
   ```bash
   mv Makefile.template Makefile
   mv project.yml.template project.yml
   # エディタで {{PROJECT_NAME}} 等を置換
   ```

4. プロジェクト生成
   ```bash
   make setup
   ```

## テンプレート内容

### Makefile

| ターゲット | 用途 |
|-----------|------|
| `setup` | XcodeGen でプロジェクト生成 |
| `dev` | ビルド＆起動（バックグラウンド） |
| `dev-fg` | ビルド＆起動（フォアグラウンド） |
| `build` | Release ビルド |
| `test` | テスト実行 |
| `lint` | SwiftLint 実行 |
| `clean` | ビルド成果物削除 |
| `archive-mas` / `archive` | App Store アーカイブ作成 |
| `bump-build` | バージョン番号インクリメント |
| `lint-issues` | Issue ファイル名の命名規則チェック |
| `upload-testflight` | TestFlight アップロード |

### ディレクトリ

| ディレクトリ | 用途 |
|-------------|------|
| `docs/` | 設計・仕様ドキュメント |
| `issues/` | Issue追跡（完了分は `done/` へ） |
| `releases/` | リリースノート管理 |
| `signing/` | 署名関連ファイル |

### CI/CD

- PR時のSwiftLint検証（ubuntu-slim）

## 推奨 Lint

新規プロジェクトでは以下の Lint を必ず導入すること。スクリプトは `bin/` に格納されている。

| Lint | コマンド | 説明 |
|------|---------|------|
| SwiftLint | `make lint` | Swift コードの静的解析（Makefile に組み込み済み） |
| Issue ファイル名 | `make lint-issues` | `issues/` 内の番号付きファイルにプレフィックス（`feat-`, `bug-`, `refactor-`, `perf-`, `ux-`）を強制 |

## 運用ガイド

各プラットフォームの `operations.md` を参照。

