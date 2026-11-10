# Fastlane テンプレート

全アプリ共通の Fastlane 標準セットアップ。
このディレクトリをプロジェクトルートにコピーして使う。

## ディレクトリ設計

```
fastlane/                                       ← API Key と Fastfile/Deliverfile/Appfile のみ
releases/current/metadata/<locale>/             ← 説明文・キーワード・URL 等（バージョン非依存）
releases/v$(MARKETING_VERSION)/release_notes/   ← リリースノート（<locale>.txt。バージョン別）
releases/v$(MARKETING_VERSION)/screenshots/     ← スクリーンショット（バージョン別）
```

`fastlane/metadata/`, `fastlane/screenshots/` は **使わない**（fastlane upstream の慣習とは異なる）。

App Store Connect は説明文・キーワード・サブタイトル等について「現在の値」しか持たない。
バージョンごとに複製すると **古いディレクトリを編集しても誰も気づかない**ので、共有の 1 組に寄せる。
過去に何を出したかは git のタグ (`v{MARKETING_VERSION}`) と履歴で辿る。

**運用フロー:**
1. 新バージョン準備時
   ```bash
   make bump-marketing V=1.0.1
   mkdir -p releases/v1.0.1/release_notes releases/v1.0.1/screenshots
   cp -r releases/v1.0.0/screenshots/. releases/v1.0.1/screenshots/   # 変わらないなら流用
   ```
2. `releases/current/metadata/` (説明文等) と `releases/v$(MARKETING_VERSION)/release_notes/<locale>.txt` を編集
3. `make asc-metadata` / `make asc-screenshots` / `make asc-upload` で App Store Connect に反映

**旧レイアウト (`releases/v$(MARKETING_VERSION)/metadata/` に全項目) も動く。**
Fastfile は `releases/current/metadata` が在ればそちらを、無ければ従来のバージョンディレクトリを使う。
リリースノートも `releases/v<version>/release_notes/` が無ければ metadata 配下の `release_notes.txt` を使う。

## 必須ルール

- **API Key 認証のみ使用**（Apple ID / パスワード認証は禁止）
- `api_key.json` と `*.p8` は Git で管理する（ポータビリティ重視）
- `releases/current/metadata/` と `releases/v$(MARKETING_VERSION)/release_notes/` を直接編集する（中間ファイルは作らない）
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
共有メタデータは `releases/current/metadata/`、スクリーンショットは初回リリースの
`releases/v$(MARKETING_VERSION)/screenshots/` にコピーする。

```bash
cd /path/to/your-app
V=$(grep 'MARKETING_VERSION' project.yml | head -1 | sed 's/.*"\(.*\)"/\1/')
mkdir -p "releases/current" "releases/v${V}/release_notes"
cp -r ../../project-templates/fastlane/metadata.template    "releases/current/metadata"
cp -r ../../project-templates/fastlane/screenshots.template "releases/v${V}/screenshots"
mv "releases/current/metadata"/*/release_notes.txt "releases/v${V}/release_notes/" 2>/dev/null || true
```

その後、`releases/current/metadata/{en-US,ja}/*.txt` を実値で埋める:

```
releases/current/metadata/en-US/name.txt          # アプリ名（30文字以内）
releases/current/metadata/en-US/subtitle.txt      # サブタイトル（30文字以内）
releases/current/metadata/en-US/keywords.txt      # キーワード（100文字以内）
releases/current/metadata/en-US/description.txt   # アプリ概要（4000文字以内）
releases/v${V}/release_notes/en-US.txt            # リリースノート（4000文字以内。バージョン別）
```

## 標準コマンド（Makefile ターゲット）

| ターゲット | 内容 |
|-----------|------|
| `asc-metadata` | メタデータのみアップロード（スクショなし） |
| `asc-screenshots` | スクリーンショットのみアップロード |
| `asc-upload` | メタデータ + スクリーンショット |
| `asc-upload-build` | ビルドのみアップロード（審査提出なし） |
| `asc-upload-all` | メタデータ + スクショ + ビルド |
| `asc-submit` | 審査に提出（メタデータと同じ場所に `submission_information.json` があれば、IDFA・暗号化・コンテンツ権利の回答として渡す） |
| `asc-availability-show` | ASC の現在の配信対象 territory を表示し、配信する設定だが販売できない territory (`contentStatuses` が `["AVAILABLE"]` 以外。DSA・暗号の申告など) があれば警告する |
| `asc-availability-diff` | `fastlane/available_territories.json` と ASC の配信対象の差分を表示（書き込まない） |
| `asc-availability-pull` | ASC の配信対象で `fastlane/available_territories.json` を書き直す（ASC は変えない。未 commit の編集があれば `FORCE=1` が要る） |
| `asc-availability-apply` | JSON との差分を表示する dry-run。`CONFIRM=1` で ASC に反映。配信を止める / 始める地域が 6 つ以上なら、dry-run が示す `ALLOW_DISABLE=<件数>` / `ALLOW_ENABLE=<件数>` も要る |

```bash
make asc-metadata       # メタデータアップロード（releases/current/metadata + v$(MARKETING_VERSION)/release_notes から）
```

## 配信対象 territory の管理

App Store の配信国/地域（territory）は **app-level 設定**で version 非依存のため、
`releases/v$(MARKETING_VERSION)/` ではなく **`fastlane/available_territories.json`** が
単一マスター。

```json
{
  "_comment": "ISO 3166 alpha-3。make asc-availability-pull で現状取得。",
  "territories": ["JPN", "USA", "GBR"]
}
```

**運用フロー:**

```bash
# 初めて / Web UI で変えた後: ASC の今の設定で JSON を書き直し、git diff で差分を見て commit
make asc-availability-pull              # 未 commit の編集があると止まる (FORCE=1 で捨てて書き直す)

# repo で変える: JSON を編集 → 差分を確かめる → 反映
$EDITOR fastlane/available_territories.json
make asc-availability-diff              # 書き込まない
make asc-availability-apply             # dry-run (反映する予定を表示するだけ)
make asc-availability-apply CONFIRM=1   # ASC に反映し、取り直して JSON と一致したかを確かめる
```

- 反映は `PATCH /v1/territoryAvailabilities/{id}` で 1 地域ずつ `available` を切り替える
  (旧 `/v1/apps/{id}/availableTerritories` は 404、`POST /v2/appAvailabilities` は既存の設定があると 409 で使えない)
- 壊れた JSON で配信を止めないよう、空・重複・ASC に無い code は拒否し、配信を止める / 始める地域がどちらかの向きで 6 つ以上なら、計画と同じ件数の許可 (`ALLOW_DISABLE=<件数>` / `ALLOW_ENABLE=<件数>`) を求める。件数が違えば通らないので、シェルの履歴から古い行を打ち直しても、壊れた JSON は反映されない
- `CONFIRM` / `ALLOW_DISABLE` / `ALLOW_ENABLE` / `FORCE` はコマンド行で渡したときだけ効く (シェルに export したまま残った値では書き込まない)
- apply は打った時点の ASC と JSON の差を反映する。dry-run の後に ASC 側で地域が増えると、その地域も止める対象に入る
- `availableInNewTerritories` (新しい地域にも配信) の設定自体は扱わない (Web UI で変える)。ただし JSON が正本なので、
  Apple が新しい地域を足した後に apply すると、JSON に無いその地域は止まる。apply の前に diff を見て、残すなら JSON に足す
- 配信する設定でも、DSA・暗号の申告などで販売できない地域は `contentStatuses` が `["AVAILABLE"]` 以外になる。show / diff / pull が警告する

## ディレクトリ構造（プロジェクトルートでの最終形）

```
fastlane/
├── Appfile                    # Bundle ID・認証情報（プレースホルダーを置換）
├── Deliverfile                # deliver のデフォルト設定
├── Fastfile                   # lane 定義（バージョンディレクトリを動的解決）
├── available_territories.json          # 配信対象 territory（ISO 3166 alpha-3、app-level）
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

### `Metadata directory not found: releases/current/metadata または releases/vX.Y.Z/metadata`

共有レイアウトなら `releases/current/metadata/` を、旧レイアウトなら `MARKETING_VERSION` に
対応するディレクトリを用意する:

```bash
# 共有レイアウト (推奨)
mkdir -p releases/vCURRENT/release_notes releases/vCURRENT/screenshots
# 旧レイアウト
cp -r releases/vPREV/metadata releases/vCURRENT/metadata
cp -r releases/vPREV/screenshots releases/vCURRENT/screenshots
```

### `Release notes not found: releases/vX.Y.Z/release_notes/<locale>.txt`

共有レイアウトで `release_notes/` ディレクトリは在るが `.txt` が無い。そのまま upload すると
ASC の「このバージョンの新機能」が前バージョンのまま残るので fail-fast する。

### subtitle が 30 文字を超えている

```
'subtitle' cannot be longer than '30' characters
```

→ `releases/current/metadata/en-US/subtitle.txt` を 30 文字以内に修正。

### メタデータを App Store Connect からダウンロードしたい

```bash
fastlane deliver download_metadata --metadata-path releases/current/metadata
```
