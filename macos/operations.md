# 運用ガイド

## バージョン管理

### バージョニング規則

| 設定 | 役割 | 形式 | 変更タイミング |
|------|------|------|------------|
| `MARKETING_VERSION` | App Store に表示される公開バージョン | `1.0.5` | App Store 新バージョン公開準備時のみ |
| `CURRENT_PROJECT_VERSION` | ビルド番号（単調増加する整数） | `7` | TestFlight アップロードのたびに +1 |

**運用ポリシー:**
- **開発中はビルド番号だけを上げる** — `MARKETING_VERSION` は固定のまま、TestFlight に何度でも投げられる
- **App Store で新バージョンを公開する準備のときだけ `MARKETING_VERSION` を上げる**

```bash
# TestFlight アップロード時（ビルド番号だけ +1）
make upload-testflight

# ビルド番号だけ手動で +1
make bump-build
```

> **公開バージョン（`MARKETING_VERSION`）は `project.yml` を直接編集して変更する。**
> コマンドは用意しない。App Store 審査提出の準備時だけ手動で変更すること。

### Gitタグ
- TestFlightアップロード時: `testflight/{MARKETING_VERSION}-{CURRENT_PROJECT_VERSION}`
  - 例: `testflight/1.0.5-7`（同じ公開バージョンで複数ビルドを区別できる）
- リリース時: `v{MARKETING_VERSION}`

---

## リリースノート管理（releases/）

### ワークフロー
1. 開発中の変更は `releases/unreleased.md` に随時記録
2. リリース時に `releases/vX.Y.md` にリネーム
3. リリース日とコミットIDを追記
4. 新しい `unreleased.md` を作成

### カテゴリ
- **New Features**: 新機能
- **Changes**: 既存機能の変更
- **Fixes**: バグ修正
- **Performance**: パフォーマンス改善
- **Refactoring**: リファクタリング（ユーザーに影響なし）

---

## Issue追跡（issues/）

### ファイル命名規則
- 番号付き: `001-feature-name.md`, `002-bug-description.md`
- カテゴリプレフィックス: `PERF-001-optimization.md`, `UI-002-layout-fix.md`

### ディレクトリ構造
```
issues/
├── 001-current-issue.md
├── 002-another-issue.md
└── done/
    ├── 001-completed-issue.md
    └── 002-another-completed.md
```

### 完了時
- 完了したissueは `issues/done/` に移動

---

## 新規プロジェクト作成時のチェックリスト

- [ ] `Info.plist` に `ITSAppUsesNonExemptEncryption = false` を追加する
  - これがないと TestFlight アップロードのたびに輸出コンプライアンスの手動対応が必要になる
  - iCloud/HTTPS などの標準暗号化のみ使用している場合は `false` でよい

---

## App Store提出フロー

### TestFlight アップロード（日常的な開発ビルド）

```bash
make upload-testflight
```

このコマンドで以下が実行される:
1. `CURRENT_PROJECT_VERSION`（ビルド番号）のみインクリメント
2. MASアーカイブ作成
3. `testflight/{VERSION}-{BUILD}` タグ作成・プッシュ
4. App Store Connectへアップロード

`MARKETING_VERSION` は変更されない。同じ公開バージョンのまま何度でも投げられる。

### App Store 新バージョン公開の準備

新しい公開バージョンを審査に出す前に一度だけ実行する:

```bash
# 1. project.yml の MARKETING_VERSION を手動で編集
#    例: "1.0.5" -> "1.0.6"
vi project.yml

# 2. テスト・リントを確認
make test
make lint

# 3. releases/unreleased.md の内容を確認・整理

# 4. TestFlight へアップロード（ビルド番号も +1 される）
make upload-testflight
```

### App Store審査提出
1. App Store Connectでビルドを選択
2. スクリーンショット・メタデータを更新
3. 審査に提出

---

## 署名設定（signing/）

### 必要ファイル
| ファイル | 用途 |
|---------|------|
| `embedded.provisionprofile` | プロビジョニングプロファイル |
| `entitlements.mas.plist` | MAS用エンタイトルメント |
| `entitlements.mas.inherit.plist` | 継承用エンタイトルメント |
| `ExportOptions-MAS.plist` | MASエクスポート設定 |
| `MAS.xcconfig` | MAS用Xcode設定 |

### プロファイルのインストール
```bash
make install-profile
```

---

## CI/CD（GitHub Actions）

### SwiftLint PR検証
- PRがオープンされた時に`.swift`ファイルの変更を検証
- ubuntu-latestで実行（SwiftLint Linux版を使用）

### 検証内容
- SwiftLintルール違反チェック
- （オプション）ユニットテスト実行

---

## 日常開発

### 開発サーバー起動
```bash
make dev      # バックグラウンドで起動
make dev-fg   # フォアグラウンドでログ出力
```

### テスト実行
```bash
make test     # 全テスト
make lint     # リントのみ
```

### E2Eテスト

E2Eテストではアプリ内でローカルWebサーバーを起動し、REST APIやHTTPリクエストをテストする。

**依存ライブラリ例**:
- [Swifter](https://github.com/httpswift/swifter) - 軽量HTTPサーバー

**パターン**:
```swift
// テストのsetUp()でサーバー起動
let server = HttpServer()
server["/api/test"] = { request in
    return .ok(.json(["status": "ok"]))
}
try server.start(8080)

// tearDown()で停止
server.stop()
```

**注意点**:
- ポート競合を避けるため、テストごとに異なるポートを使うか、動的にポートを割り当てる
- サーバー起動/停止はsetUp/tearDownで確実に行う
- タイムアウト設定を適切に（サーバー起動待ちを考慮）

### クリーンビルド
```bash
make clean
make build
```
