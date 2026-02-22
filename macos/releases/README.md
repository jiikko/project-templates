# リリースノート管理

## バージョニング規則

| 設定 | 役割 | 変更タイミング |
|------|------|------------|
| `MARKETING_VERSION` | App Store 公開バージョン（例: `1.0.5`） | App Store 新バージョン公開準備時のみ |
| `CURRENT_PROJECT_VERSION` | ビルド番号・整数（例: `7`） | TestFlight アップロードのたびに +1 |

**開発中は `MARKETING_VERSION` を固定したまま、ビルド番号だけ上げ続ける。**
新しい公開バージョンを App Store に出す準備をするときだけ `project.yml` の `MARKETING_VERSION` を手動で編集する。

### タグ形式
- TestFlight: `testflight/{MARKETING_VERSION}-{BUILD}` 例: `testflight/1.0.5-7`
- App Store リリース: `v{MARKETING_VERSION}` 例: `v1.0.5`

## ワークフロー

1. 新リリースサイクル開始時に `vX.Y.Z.md` を作成し、変更内容を随時記録
2. リリース時にリリース日とコミットIDを追記
3. 次のサイクルは次のバージョン番号で新しいファイルを作成

## カテゴリ

- **New Features**: 新機能
- **Changes**: 既存機能の変更
- **Fixes**: バグ修正
- **Performance**: パフォーマンス改善
- **Refactoring**: リファクタリング
