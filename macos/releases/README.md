# リリース管理ディレクトリ

このディレクトリはアプリのリリース履歴（changelog）を管理します。

## ファイル構成

```
releases/
├── README.md              # このファイル
├── v1.1.0.md              # 開発中（次回公開予定）
├── v1.0.3.md              # 過去のリリース履歴
├── vX.Y.Z/                # Fastlane 導入済みの場合: そのバージョンのメタデータ実体
│   ├── metadata/{ja,en-US}/   # App Store Connect に upload する内容
│   └── screenshots/{ja,en-US}/ # App Store スクリーンショット
└── ...
```

> **Fastlane 導入済みの場合:** `releases/vX.Y.Z/{metadata,screenshots}/` がそのバージョンの App Store 公開内容そのもの。
> Fastfile が `project.yml` の `MARKETING_VERSION` を読んで、対応するディレクトリをアップロード元に動的解決する。
> 「マスター」と「スナップショット」を分ける運用は採らない（バージョンディレクトリ自体が履歴）。

## バージョニング規則

| 設定 | 役割 | 変更タイミング |
|------|------|------------|
| `MARKETING_VERSION` | App Store 公開バージョン（例: `1.0.5`） | App Store 新バージョン公開準備時のみ |
| `CURRENT_PROJECT_VERSION` | ビルド番号・整数（例: `7`） | TestFlight アップロードのたびに +1 |

**開発中は `MARKETING_VERSION` を固定したまま、ビルド番号だけ上げ続ける。**
新しい公開バージョンを App Store に出す準備をするときだけ `MARKETING_VERSION` を更新する（`make bump-marketing V=x.y.z`）。

#### ビルド番号の不変条件

- **同一 `MARKETING_VERSION` 内では strictly increasing**。App Store Connect が同一バージョン内の重複ビルド番号を拒否する
- **`MARKETING_VERSION` を跨いだときの扱いはアプリごとの運用に委ねる**。`1` にリセットしても、そのまま増やし続けてもよい

跨いだときの挙動は各アプリの `make bump-marketing` の実装が正本になる。リセットする実装（`apps/fdup-macos`）と、
リセットせず increasing を維持する実装（`apps/SnapTrim` / `apps/ThumbnailThumb`）が現存するため、このテンプレートでは
どちらかに断定しない。新規アプリはどちらかを選び、`bump-marketing` の実装とアプリ側 README の記述を揃えること。

### タグ形式

- TestFlight: `testflight/{MARKETING_VERSION}-{BUILD}` 例: `testflight/1.0.5-7`
- App Store リリース: `v{MARKETING_VERSION}` 例: `v1.0.5`

### コマンド

```bash
# TestFlight アップロード（ビルド番号を自動インクリメント）
make upload-testflight

# ビルド番号だけ手動で +1 したい場合
make bump-build

# App Store 新バージョン準備時のみ（MARKETING_VERSION を更新）
make bump-marketing V=1.1.0
```

> **公開バージョン（`MARKETING_VERSION`）の変更は `make bump-marketing V=x.y.z` に一本化する。**
> `project.yml` の手編集は行わない。App Store 審査提出の準備時だけ変更すること。

## 運用フロー

### 1. 開発中

次回公開予定バージョンのファイル `releases/vX.Y.Z.md` を直接育てる。

`unreleased.md` は使わない。変更を加えたら `releases/vX.Y.Z.md` に随時追記する。

例: 現在の公開版が `1.0.5`、次回が `1.0.6` なら `releases/v1.0.6.md` に追記。

### 2. App Store 申請準備

`releases/vX.Y.Z.md` にリリース日・申請状態・App Store 用リリースノート素案を追記して仕上げる。

```bash
make bump-marketing V=1.0.6  # project.yml の MARKETING_VERSION を更新
```

> **Fastlane 導入済みの場合:** バージョンディレクトリを直接編集してアップロードする。
> 編集対象は `releases/v$(MARKETING_VERSION)/{metadata,screenshots}/`。Fastfile が
> `project.yml` の `MARKETING_VERSION` を読んで動的にパスを解決する。
> ```bash
> # 新バージョン準備時: 前バージョンから初期化
> cp -r releases/v1.0.5/metadata releases/v1.0.6/metadata
> cp -r releases/v1.0.5/screenshots releases/v1.0.6/screenshots
> make bump-marketing V=1.0.6
>
> # releases/v1.0.6/{metadata,screenshots}/ を編集したら:
> make asc-metadata         # メタデータのみ変更時
> make asc-screenshots      # スクリーンショットのみ変更時
> make asc-upload           # 両方変更時
> ```

### 3. リリース後・次サイクル開始

```bash
git tag v1.0.6 && git push origin v1.0.6
touch releases/v1.0.7.md     # 次バージョンのファイルを作成
```

## カテゴリガイドライン

変更内容は以下のカテゴリに分類する（Keep a Changelog 形式）。

- **🎉 新機能 (Added)**: 新しく追加された機能
- **🔄 変更 (Changed)**: 既存機能の変更・改善
- **🐛 修正 (Fixed)**: バグ修正
- **⚡ パフォーマンス (Performance)**: 速度・効率の改善
- **🗑️ 削除 (Removed)**: 削除された機能
- **🔧 リファクタリング (Refactoring)**: コード品質向上（ユーザー影響なし）

## リリースノートファイルの書き方

```markdown
# v1.0.6

リリース日: 2026-XX-XX
コミット: xxxxxxx
前回リリース: 2026-XX-XX (v1.0.5)

## 🎉 新機能 (Added)
- ○○機能を追加

## 🔄 変更 (Changed)
- ○○の動作を変更

## 🐛 修正 (Fixed)
- ○○のバグを修正

---

## App Store リリースノート素案

### en-US
• Added: ○○ feature
• Fixed: ○○ issue

### ja
• 新機能: ○○を追加
• 修正: ○○を修正

---

## リリース前動作確認

- [ ] 新機能が仕様通り動く
- [ ] 既存機能が壊れていない（回帰確認）
- [ ] TestFlight で実機確認済み
```

## 書き方のコツ

- **ユーザー視点で書く**: 技術的な詳細よりも、ユーザーにとっての影響を記述
- **簡潔に**: 1行で要点を伝える
- **行動ベース**: 「○○を追加」「○○を修正」のように動詞で始める
- **コミットIDを必ず記載**: リリースのトレーサビリティ確保

---

> このファイルは `project-templates` で統合管理されています。直接変更せず、`project-templates/macos/releases/README.md` を変更してください。
