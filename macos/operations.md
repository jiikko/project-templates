# 運用ガイド

## バージョン管理

### バージョニング規則
- `MARKETING_VERSION` = `CURRENT_PROJECT_VERSION` を常に同期
- `BASE_VERSION`（例: 1.0）を固定し、パッチ番号のみインクリメント
- 例: 1.0.1 → 1.0.2 → 1.0.3

### Gitタグ
- TestFlightアップロード時: `testflight/X.Y.Z`
- リリース時: `vX.Y.Z`

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

## App Store提出フロー

### 事前準備
1. `make bump-build` でバージョン番号をインクリメント
2. `make test` で全テストがパスすることを確認
3. `make lint` でリント警告がないことを確認
4. `releases/unreleased.md` の内容を確認

### TestFlightアップロード
```bash
make upload-testflight
```

このコマンドで以下が実行される:
1. バージョン番号インクリメント
2. MASアーカイブ作成
3. Gitタグ作成・プッシュ
4. App Store Connectへアップロード

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

### クリーンビルド
```bash
make clean
make build
```
