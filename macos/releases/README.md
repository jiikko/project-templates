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

### 非推奨: `unreleased.md` 運用

`unreleased.md` に開発中の変更を溜めて、公開時に `v{MARKETING_VERSION}.md` へリネームする運用は非推奨。

理由:
- 次回公開予定の変更がどの公開バージョン向けか分かりにくい
- App Store 申請準備中の版と、その次の開発分が同じファイルに混ざりやすい
- リリースノートの整理タイミングが遅れやすい

### 推奨: 次回版の `v{MARKETING_VERSION}.md` を直接育てる

次回 App Store に出す予定の `v{MARKETING_VERSION}.md` を開発中から直接更新する。

例:
- 現在の公開版が `1.0.5`
- 次回公開予定が `1.0.6`
- 開発中の変更は `releases/v1.0.6.md` に追記する

公開準備が始まったら、そのファイルにリリース日、申請状態、動作確認結果、App Store 用文面を追記して仕上げる。
次の開発サイクルに入ったら、さらに次の版 `releases/v1.0.7.md` を作って記録を始める。

### 旧運用

1. 開発中の変更は `unreleased.md` に随時記録
2. App Store 公開版のリリース時に `v{MARKETING_VERSION}.md`（例: `v1.0.md`）にリネーム
3. リリース日とコミットIDを追記
4. 新しい `unreleased.md` を作成

## カテゴリ

- **New Features**: 新機能
- **Changes**: 既存機能の変更
- **Fixes**: バグ修正
- **Performance**: パフォーマンス改善
- **Refactoring**: リファクタリング
