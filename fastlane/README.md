# Fastlane テンプレート

全アプリ共通の Fastlane 標準セットアップ。
このディレクトリをプロジェクトルートにコピーして使う。

## ディレクトリ設計

```
fastlane/                              ← API Key と Fastfile/Deliverfile/Appfile のみ
appstore-listing/
  metadata/<locale>/                   ← 説明文・キーワード・URL 等とリリースノート (release_notes.txt)
  screenshots/<locale>/                ← スクリーンショット
  release_notes_version.txt            ← リリースノートがどのバージョン用か (MARKETING_VERSION と同じ値)
  review_attachment.txt                ← (任意) 審査の添付ファイルのパス
```

`fastlane/metadata/`, `fastlane/screenshots/` は **使わない**（fastlane upstream の慣習とは異なる）。

App Store Connect は説明文・キーワード・サブタイトル等について「現在の値」しか持たない。
バージョンごとに複製すると **古いディレクトリを編集しても誰も気づかない**ので、バージョンを含まない 1 か所に置く。
過去に何を出したかは git のタグ (`v{MARKETING_VERSION}`) と履歴で辿る。

メタデータを送る lane は最初に、`release_notes_version.txt` が `project.yml` の `MARKETING_VERSION` と
違うとき、またはどれかの locale の `release_notes.txt` が無いか空のときに止まる (前のバージョンのノートを送らないため)。

**運用フロー:**
1. 新バージョン準備時 (ディレクトリの作成・名前の変更・スクリーンショットのコピーは要らない)
   ```bash
   make bump-marketing V=1.0.1
   # appstore-listing/metadata/<locale>/release_notes.txt を空にして、1.0.1 のノートを書く
   echo 1.0.1 > appstore-listing/release_notes_version.txt   # ノートを書いた人が印を新しいバージョンにする
   ```
2. 説明文等とスクリーンショットが変わるなら `appstore-listing/` を直接編集する
3. `make asc-listing-check` で置き場所・印・deliver の読み込みを確かめる (App Store Connect には接続しない)
4. `make asc-metadata` / `make asc-screenshots` / `make asc-upload` で App Store Connect に反映

**前の置き場所も読める。** `appstore-listing/metadata` が無い app は、共有レイアウト
(`releases/current/metadata/` + `releases/v$(MARKETING_VERSION)/{release_notes,screenshots}/`)、
それも無ければ旧来 (`releases/v$(MARKETING_VERSION)/{metadata,screenshots}/`) を読む。
新しい app と移す app は `appstore-listing/` にする。

**前の置き場所から移す** (共有レイアウトの例。`V` は今の `MARKETING_VERSION`):

```bash
mkdir -p appstore-listing/screenshots
git mv releases/current/metadata appstore-listing/metadata
git mv "releases/v${V}/screenshots/"* appstore-listing/screenshots/
for f in "releases/v${V}/release_notes/"*.txt; do
  l=$(basename "$f" .txt); git mv "$f" "appstore-listing/metadata/$l/release_notes.txt"
done
echo "$V" > appstore-listing/release_notes_version.txt
# releases/current/ に残ったもの (review_attachment.txt・review_attachments/・subscriptions/ など) も移す
for f in releases/current/*; do git mv "$f" appstore-listing/; done
# review_attachment.txt の中のパス (releases/current/review_attachments/...) を appstore-listing/review_attachments/... に直す
make asc-listing-check
```

> iOS のテンプレート (`Fastfile.ios.template`) は未対応で、前の置き場所 (`releases/v<version>/{metadata,screenshots}/`) のまま。

## 必須ルール

- **API Key 認証のみ使用**（Apple ID / パスワード認証は禁止）
- `api_key.json` と `*.p8` は Git で管理する（ポータビリティ重視）
- `appstore-listing/` を直接編集する（中間ファイルは作らない）
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
`snapshot-release` / `archive-release` 等のスナップショット系ターゲットは不要（過去に出した内容は git のタグで辿るため）。

### 6. 掲載素材のディレクトリを作成

`fastlane/metadata.template/` と `fastlane/screenshots.template/` がプレースホルダ。
`appstore-listing/metadata` と `appstore-listing/screenshots` にコピーし、`release_notes_version.txt` を作る。

```bash
cd /path/to/your-app
V=$(grep 'MARKETING_VERSION' project.yml | head -1 | sed 's/.*"\(.*\)"/\1/')
mkdir -p appstore-listing
cp -r ../../project-templates/fastlane/metadata.template    appstore-listing/metadata
cp -r ../../project-templates/fastlane/screenshots.template appstore-listing/screenshots
echo "$V" > appstore-listing/release_notes_version.txt
```

その後、`appstore-listing/metadata/{en-US,ja}/*.txt` を実値で埋める:

```
appstore-listing/metadata/en-US/name.txt           # アプリ名（30文字以内）
appstore-listing/metadata/en-US/subtitle.txt       # サブタイトル（30文字以内）
appstore-listing/metadata/en-US/keywords.txt       # キーワード（100文字以内）
appstore-listing/metadata/en-US/description.txt    # アプリ概要（4000文字以内）
appstore-listing/metadata/en-US/release_notes.txt  # リリースノート（4000文字以内。今のバージョン用）
```

## 標準コマンド（Makefile ターゲット）

| ターゲット | 内容 |
|-----------|------|
| `asc-listing-check` | App Store Connect に接続せず、置き場所・`release_notes_version.txt`・deliver の読み込みを確かめる |
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
make asc-metadata       # メタデータアップロード（appstore-listing/metadata から）
```

## 配信対象 territory の管理

App Store の配信国/地域（territory）は **app-level 設定**で version 非依存のため、
`appstore-listing/` ではなく **`fastlane/available_territories.json`** が
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
- `CONFIRM` / `ALLOW_DISABLE` / `ALLOW_ENABLE` / `FORCE` はコマンド行で渡したときだけ効く (シェルに export したまま残った値では書き込まない)。
  ただし環境変数の `MAKEFLAGS` に入れた代入は、make がコマンド行の値として扱うので通る (親の make から子へ引き継ぐ形。見分けられない)。`MAKEFLAGS` に入れたまま残さない
- テンプレートを入れた直後や長く触っていない app では、apply の前に `make asc-availability-pull` で JSON を ASC の今の値に揃える。
  仮置きの JSON (1 か国だけ等) のまま dry-run すると、ほとんどの国の配信を止める計画と、そのまま打てる `CONFIRM=1 ALLOW_DISABLE=<件数>` の行が出る
- apply は打った時点の ASC と JSON の差を反映する。dry-run の後に ASC 側で地域が増えると、その地域も止める対象に入る
- `availableInNewTerritories` (新しい地域にも配信) の設定自体は扱わない (Web UI で変える)。ただし JSON が正本なので、
  Apple が新しい地域を足した後に apply すると、JSON に無いその地域は止まる。apply の前に diff を見て、残すなら JSON に足す
- 配信する設定でも、DSA・暗号の申告などで販売できない地域は `contentStatuses` が `["AVAILABLE"]` 以外になる。show / diff / pull が警告する

## ディレクトリ構造（プロジェクトルートでの最終形）

```
fastlane/
├── Appfile                    # Bundle ID・認証情報（プレースホルダーを置換）
├── Deliverfile                # deliver のデフォルト設定
├── Fastfile                   # lane 定義（置き場所は appstore-listing/ を読む）
├── available_territories.json          # 配信対象 territory（ISO 3166 alpha-3、app-level）
├── api_key.json               # App Store Connect API Key
└── AuthKey_XXXXXXXXXX.p8      # 秘密鍵
appstore-listing/
├── metadata/
│   ├── en-US/                 # 英語メタデータ（必須）
│   │   ├── name.txt
│   │   ├── subtitle.txt
│   │   ├── keywords.txt
│   │   ├── description.txt
│   │   ├── release_notes.txt  # 今のバージョンのノート
│   │   ├── promotional_text.txt
│   │   ├── privacy_url.txt
│   │   ├── support_url.txt
│   │   └── marketing_url.txt
│   └── ja/                    # 日本語メタデータ（任意）
│       └── （同上）
├── screenshots/
│   ├── en-US/
│   └── ja/
├── release_notes_version.txt  # リリースノートがどのバージョン用か
└── review_attachment.txt      # (任意) 審査の添付
releases/
└── v1.0.0.md                  # changelog（App Store の素材は置かない）
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

## 審査の添付ファイル (App Review に関する情報)

バージョンの「App Review に関する情報」の添付 (審査の担当者だけが見る説明動画など) を使う app は、
`appstore-listing/review_attachment.txt` に、添付するファイルのパス (project のルートからの相対) を 1 行で書く。

```
appstore-listing/review_attachments/demo.mp4
```

- 🚨 deliver はメタデータを送るとき、添付を渡さないと **App Store Connect に既にある添付を全部消す** (fastlane 2.238.0 で確認)。
  `metadata` / `upload_metadata` / `upload_all` の lane は `review_attachment_options` で、この指定が在れば毎回そのファイルを添付し直す
- `appstore-listing/` に移していない app は `releases/current/review_attachment.txt` を読む
- 指定が無い app は今までどおり添付を渡さない (既にある添付は消える。lane がその旨を出す)。Web で手で添付しても、次にメタデータを送ると消える
- 指定したファイルが無いと lane は止まる (黙って添付を消さない)。添付をやめるなら `review_attachment.txt` を消す

## App Store Connect API Key の作成

1. [App Store Connect API](https://appstoreconnect.apple.com/access/integrations/api) でキーを生成
2. ロール: **App Manager** 以上
3. Key ID・Issuer ID・`.p8` ファイルを `api_key.json` に記入

## トラブルシューティング

### `Metadata directory not found: appstore-listing/metadata・releases/current/metadata・releases/v… のどれも無い`

`appstore-listing/metadata/<locale>/` を作る (セットアップ手順 6)。前の置き場所から移すなら上の「前の置き場所から移す」。

### `appstore-listing のリリースノートはバージョン X 用で、MARKETING_VERSION=Y と違う`

`appstore-listing/metadata/<locale>/release_notes.txt` を Y のノートに書き直してから、
`appstore-listing/release_notes_version.txt` を Y にする。前のバージョンのノートを送らないための停止。

### `release_notes_version.txt が無いか空`

`appstore-listing/release_notes_version.txt` に、リリースノートを書いたバージョン (`MARKETING_VERSION` と同じ値) を書く。

### `appstore-listing/metadata/<locale>/release_notes.txt が無いか空`

その locale のリリースノートを書く。そのまま送ると「このバージョンの新機能」が前のバージョンのまま残る。

### `Screenshots directory not found`

`appstore-listing/screenshots/<locale>/` を作ってスクリーンショットを置く (`fastlane/screenshots.template` をコピー)。

### subtitle が 30 文字を超えている

```
'subtitle' cannot be longer than '30' characters
```

→ `appstore-listing/metadata/en-US/subtitle.txt` を 30 文字以内に修正。

### メタデータを App Store Connect からダウンロードしたい

```bash
fastlane deliver download_metadata --metadata-path appstore-listing/metadata
```
