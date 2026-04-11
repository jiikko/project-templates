# プロジェクトテンプレート

iOS/macOSアプリの共通ディレクトリ構造と運用フローのテンプレート集。

## 構造

```
project-templates/
├── bin/                # 共通スクリプト（全プラットフォーム共有）
│   └── lint-issues     # Issue ファイル名の命名規則チェック
├── macos/              # macOSアプリ用テンプレート
│   ├── README.md
│   ├── README.md       # 使い方・運用ガイド
│   ├── Makefile.template
│   ├── project.yml.template
│   ├── .swiftlint.yml
│   ├── .github/workflows/swiftlint.yml
│   ├── releases/
│   ├── signing/
│   └── issues/
└── ios/                # iOSアプリ用テンプレート
    ├── README.md
    ├── Makefile.template
    ├── project.yml.template
    ├── .swiftlint.yml
    ├── .github/workflows/swiftlint.yml
    ├── releases/
    ├── signing/
    └── issues/
```

## 使い方

### 新規プロジェクト作成時（単一プラットフォーム）

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

### マルチプラットフォームの場合

macOS + iOS など複数プラットフォームを1リポジトリで管理する場合は、プラットフォームごとにディレクトリを掘り、それぞれの中にテンプレートの運用ファイルを配置する。

```
MyApp/
├── macOS/
│   ├── Sources/
│   ├── Tests/
│   ├── Resources/
│   ├── releases/
│   ├── signing/
│   ├── appstore-reviews/
│   ├── issues/
│   ├── project.yml      # XcodeGen（macOS targets）
│   └── Makefile
├── iOS/
│   ├── Sources/
│   ├── Tests/
│   ├── Resources/
│   ├── releases/
│   ├── signing/
│   ├── appstore-reviews/
│   ├── issues/
│   ├── project.yml      # XcodeGen（iOS targets）
│   └── Makefile
└── Shared/              # 共通コード（Swift Package等）
```

## Fastlane テンプレート

[`fastlane/`](fastlane/) に全アプリ共通の Fastlane 標準セットアップが格納されている。

| ファイル | 用途 |
|---------|------|
| `Appfile.template` | Bundle ID・認証情報（プレースホルダー置換用） |
| `Deliverfile.macos.template` | macOS アプリ向け deliver 設定 |
| `Deliverfile.ios.template` | iOS アプリ向け deliver 設定 |
| `Fastfile.macos.template` | macOS アプリ向け lane 定義 |
| `Fastfile.ios.template` | iOS アプリ向け lane 定義 |
| `api_key.json.template` | API Key 設定サンプル |
| `.gitignore` | `api_key.json` / `*.p8` の除外ルール |
| `metadata/en-US/` | メタデータプレースホルダー |
| `screenshots/en-US/` | スクリーンショット格納先 |

### 必須ルール

- **API Key 認証のみ**（Apple ID / パスワード認証は禁止）
- `api_key.json` と `*.p8` は **`.gitignore` に必ず追加**
- `metadata/` と `screenshots/` は Git 管理する
- lane 名は `metadata` / `screenshots` / `upload_metadata` / `upload_build` / `upload_all` / `submit` で統一
- `bundle exec` **不使用**

詳細は [`fastlane/README.md`](fastlane/README.md) を参照。

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
| `bump-build` | ビルド番号（`CURRENT_PROJECT_VERSION`）インクリメント |
| `lint-issues` | Issue ファイル名の命名規則チェック |
| `upload-testflight` | TestFlight アップロード |

### ディレクトリ

| ディレクトリ | 用途 |
|-------------|------|
| `docs/` | 設計・仕様ドキュメント |
| `issues/` | Issue追跡（進行中は直下、完了は `done/`、保留は `pending/`） |
| `releases/` | リリースノート管理 |
| `appstore-reviews/` | App Store審査記録 |
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

各プラットフォームの `README.md` を参照。
