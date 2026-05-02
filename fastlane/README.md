# Fastlane テンプレート

全アプリ共通の Fastlane 標準セットアップ。
このディレクトリをプロジェクトルートにコピーして使う。

## ディレクトリ設計

```
fastlane/                                  ← API Key と Fastfile/Deliverfile/Appfile のみ
releases/v$(MARKETING_VERSION)/metadata/   ← App Store Connect に upload する内容（バージョン直編集）
releases/v$(MARKETING_VERSION)/screenshots/ ← 同 スクリーンショット
```

`fastlane/metadata/`, `fastlane/screenshots/` は **使わない**（fastlane upstream の慣習とは異なる）。
バージョンディレクトリ自体が「このバージョンで App Store に出す内容」かつ「過去リリースの履歴」を兼ねる。

**運用フロー:**
1. 新バージョン準備時、前バージョンから初期化
   ```bash
   cp -r releases/v1.0.0/metadata releases/v1.0.1/metadata
   cp -r releases/v1.0.0/screenshots releases/v1.0.1/screenshots
   make bump-marketing V=1.0.1
   ```
2. `releases/v$(MARKETING_VERSION)/{metadata,screenshots}/` を編集
3. `make asc-metadata` / `make asc-screenshots` / `make asc-upload` で App Store Connect に反映

`project.yml` の `MARKETING_VERSION` を切り替えれば、Fastfile が自動的に対応するディレクトリを upload 元に解決する。スナップショットを別途取る運用は不要。

## 必須ルール

- **API Key 認証のみ使用**（Apple ID / パスワード認証は禁止）
- `api_key.json` と `*.p8` は Git で管理する（ポータビリティ重視）
- `releases/v$(MARKETING_VERSION)/metadata/` を直接編集する（中間ファイルは作らない）
- lane 名は下記「標準 lane 一覧」に従う（独自名は禁止）
- `bundle exec` **不使用**（Bundler は導入しない）

## Fastlane のインストール

**Homebrew を使うこと**（`gem install` / Bundler は使わない）。

```bash
brew install fastlane
```

> `gem install fastlane` は System Ruby との干渉リスクがある。
> Bundler（Gemfile 管理）は導入コストが高いため、このプロジェクト群では採用しない。
> `bundle exec` なしで直接 `fastlane` を実行する。

## セットアップ手順

### 1. テンプレートをコピー

```bash
# macOS アプリ
mkdir -p /path/to/your-app/fastlane
cp project-templates/fastlane/Fastfile.macos.template /path/to/your-app/fastlane/Fastfile
cp project-templates/fastlane/Deliverfile.macos.template /path/to/your-app/fastlane/Deliverfile

# iOS アプリ
mkdir -p /path/to/your-app/fastlane
cp project-templates/fastlane/Fastfile.ios.template /path/to/your-app/fastlane/Fastfile
cp project-templates/fastlane/Deliverfile.ios.template /path/to/your-app/fastlane/Deliverfile
```

### 2. Appfile のプレースホルダーを置換

```bash
cp project-templates/fastlane/Appfile.template /path/to/your-app/fastlane/Appfile
# エディタで {{BUNDLE_ID}}, {{APPLE_ID}}, {{TEAM_ID}}, {{ITC_TEAM_ID}} を置換
```

### 3. API Key を配置

```bash
cp project-templates/fastlane/api_key.json.template /path/to/your-app/fastlane/api_key.json
# api_key.json に実際の Key ID / Issuer ID / 秘密鍵を記入
# *.p8 ファイルを fastlane/ 直下に配置
```

### 4. .gitignore に追加

```bash
cat project-templates/fastlane/.gitignore >> /path/to/your-app/.gitignore
```

### 5. Makefile に fastlane ターゲットを追加

`macos/Makefile.template` または `ios/Makefile.template` の `asc-*` ターゲットを参照。
`snapshot-release` / `archive-release` 等のスナップショット系ターゲットは不要（バージョンディレクトリが履歴を兼ねるため）。

### 6. 初回バージョンのメタデータディレクトリを作成

`fastlane/metadata.template/` と `fastlane/screenshots.template/` がプレースホルダ。
これを初回リリースの `releases/v$(MARKETING_VERSION)/{metadata,screenshots}/` にコピーする。

```bash
cd /path/to/your-app
V=$(grep 'MARKETING_VERSION' project.yml | head -1 | sed 's/.*"\(.*\)"/\1/')
mkdir -p "releases/v${V}"
cp -r ../../project-templates/fastlane/metadata.template    "releases/v${V}/metadata"
cp -r ../../project-templates/fastlane/screenshots.template "releases/v${V}/screenshots"
```

その後、`releases/v${V}/metadata/{en-US,ja}/*.txt` を実値で埋める:

```
releases/v${V}/metadata/en-US/name.txt          # アプリ名（30文字以内）
releases/v${V}/metadata/en-US/subtitle.txt      # サブタイトル（30文字以内）
releases/v${V}/metadata/en-US/keywords.txt      # キーワード（100文字以内）
releases/v${V}/metadata/en-US/description.txt   # アプリ概要（4000文字以内）
releases/v${V}/metadata/en-US/release_notes.txt # リリースノート（4000文字以内）
```

## 標準コマンド（Makefile ターゲット）

| ターゲット | 内容 |
|-----------|------|
| `asc-metadata` | メタデータのみアップロード（スクショなし） |
| `asc-screenshots` | スクリーンショットのみアップロード |
| `asc-upload` | メタデータ + スクリーンショット |
| `asc-upload-build` | ビルドのみアップロード（審査提出なし） |
| `asc-upload-all` | メタデータ + スクショ + ビルド |
| `asc-submit` | 審査に提出 |
| `asc-availability-show` | ASC の現在の配信対象 territory を表示 |
| `asc-availability-apply` | `fastlane/availability.json` を ASC に反映 |

```bash
make asc-metadata       # メタデータアップロード（releases/v$(MARKETING_VERSION)/metadata から）
```

## 配信対象 territory の管理

App Store の配信国/地域（territory）は **app-level 設定**で version 非依存のため、
`releases/v$(MARKETING_VERSION)/` ではなく **`fastlane/availability.json`** が
単一マスター。

```json
{
  "_comment": "ISO 3166 alpha-3。fastlane mac availability_show で現状取得。",
  "territories": ["JPN", "USA", "GBR"]
}
```

**運用フロー（show は実装済み、apply は WIP）:**

```bash
# 現状を取得
make asc-availability-show
# → コピペで fastlane/availability.json を作成 / 更新

# JSON を編集（territory 追加 or 削除）→ Web UI で同じ変更を適用
$EDITOR fastlane/availability.json
# 当面は ASC Web UI で territory を編集する（apply は未実装）
```

Web 上で territory を変更したら、`asc-availability-show` で取り直して
`availability.json` を更新 → push し直すのが正規ルート（メタデータと同じ方針）。

**`availability_apply` は WIP（未実装）:**
- Apple ASC API の AppAvailability v2 は仕様変更が複数回入っており、
  Spaceship 側のメンテナンスが追いついていない
- 旧 `App#update(territory_ids:)` は `/v1/apps/{id}/availableTerritories`
  relationship を削除済みで 404
- 新 `POST /v2/appAvailabilities` は既存 AppAvailability があると 409
- `PATCH /v2/appAvailabilities/{id}` は Apple サンプルが乏しく、全
  territory resource を included に展開する必要があり要詰め切り
- 当面は **show のみ + Web UI で編集** の運用

## ディレクトリ構造（プロジェクトルートでの最終形）

```
fastlane/
├── Appfile                    # Bundle ID・認証情報（プレースホルダーを置換）
├── Deliverfile                # deliver のデフォルト設定
├── Fastfile                   # lane 定義（バージョンディレクトリを動的解決）
├── availability.json          # 配信対象 territory（ISO 3166 alpha-3、app-level）
├── api_key.json               # App Store Connect API Key
└── AuthKey_XXXXXXXXXX.p8      # 秘密鍵
releases/
├── v1.0.0/
│   ├── metadata/
│   │   ├── en-US/             # 英語メタデータ（必須）
│   │   │   ├── name.txt
│   │   │   ├── subtitle.txt
│   │   │   ├── keywords.txt
│   │   │   ├── description.txt
│   │   │   ├── release_notes.txt
│   │   │   ├── promotional_text.txt
│   │   │   ├── privacy_url.txt
│   │   │   ├── support_url.txt
│   │   │   └── marketing_url.txt
│   │   └── ja/                # 日本語メタデータ（任意）
│   │       └── （同上）
│   └── screenshots/
│       ├── en-US/
│       └── ja/
├── v1.0.1/                    # 次のリリース。前バージョンから cp -r で初期化
│   └── ...
└── v1.0.0.md                  # リリースノート
```

## メタデータ文字数制限

| ファイル | 上限 |
|---------|------|
| `name.txt` | 30文字 |
| `subtitle.txt` | 30文字 |
| `keywords.txt` | 100文字 |
| `promotional_text.txt` | 170文字 |
| `description.txt` | 4000文字 |
| `release_notes.txt` | 4000文字 |

## サブスクリプションアプリの注意

`description.txt` に EULA リンクを必ず含めること（Guideline 3.1.2）。
**新バージョン作成時にこのリンクが消えないよう注意。**

```
Terms of Use (EULA): https://www.apple.com/legal/internet-services/itunes/dev/stdeula/
Privacy Policy: https://{{YOUR_DOMAIN}}/privacy.html
```

## App Store Connect API Key の作成

1. [App Store Connect API](https://appstoreconnect.apple.com/access/integrations/api) でキーを生成
2. ロール: **App Manager** 以上
3. Key ID・Issuer ID・`.p8` ファイルを `api_key.json` に記入

## トラブルシューティング

### `Metadata directory not found: releases/vX.Y.Z/metadata`

`project.yml` の `MARKETING_VERSION` に対応するディレクトリが無い。前バージョンから初期化:

```bash
cp -r releases/vPREV/metadata releases/vCURRENT/metadata
cp -r releases/vPREV/screenshots releases/vCURRENT/screenshots
```

### subtitle が 30 文字を超えている

```
'subtitle' cannot be longer than '30' characters
```

→ `releases/v$(MARKETING_VERSION)/metadata/en-US/subtitle.txt` を 30 文字以内に修正。

### メタデータを App Store Connect からダウンロードしたい

```bash
fastlane deliver download_metadata --metadata-path releases/v$(MARKETING_VERSION)/metadata
```
