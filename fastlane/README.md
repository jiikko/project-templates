# Fastlane テンプレート

全アプリ共通の Fastlane 標準セットアップ。  
このディレクトリをプロジェクトルートにコピーして使う。

## ディレクトリ設計

```
fastlane/metadata/      ← マスターデータ（常に最新。ここを編集してアップロード）
releases/vX.X.X/metadata/  ← リリース時のスナップショット（履歴）
```

**運用フロー:**
1. `fastlane/metadata/` を編集
2. `make fastlane-metadata` でアップロード
3. `make snapshot-release` で `releases/vX.X.X/metadata/` にスナップショットを保存

## 必須ルール

- **API Key 認証のみ使用**（Apple ID / パスワード認証は禁止）
- `api_key.json` と `*.p8` は Git で管理する（ポータビリティ重視）
- `fastlane/metadata/` はマスターデータのみ置く（過去バージョンは `releases/` で管理）
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
cp -r project-templates/fastlane/ /path/to/your-app/fastlane/
cp project-templates/fastlane/Fastfile.macos.template /path/to/your-app/fastlane/Fastfile
cp project-templates/fastlane/Deliverfile.macos.template /path/to/your-app/fastlane/Deliverfile

# iOS アプリ
cp -r project-templates/fastlane/ /path/to/your-app/fastlane/
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
# api_key.json に実際の Key ID / Issuer ID / 秘密鍵パスを記入
# *.p8 ファイルを fastlane/ 直下に配置
```

### 4. .gitignore に追加

```bash
cat project-templates/fastlane/.gitignore >> /path/to/your-app/.gitignore
```

### 5. Makefile に fastlane ターゲットを追加

`macos/Makefile.template` または `ios/Makefile.template` の `fastlane-*` ターゲットを参照。

### 6. メタデータ初期値を入力

```
fastlane/metadata/en-US/name.txt          # アプリ名
fastlane/metadata/en-US/subtitle.txt      # サブタイトル（30文字以内）
fastlane/metadata/en-US/keywords.txt      # キーワード（100文字以内）
fastlane/metadata/en-US/description.txt   # アプリ概要
fastlane/metadata/en-US/release_notes.txt # リリースノート
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
| `snapshot-release` | `fastlane/metadata/` を `releases/vX.X.X/` にコピー |

```bash
make asc-metadata       # メタデータアップロード
make snapshot-release   # リリース後にスナップショット保存
```

## ディレクトリ構造

```
fastlane/
├── Appfile                    # Bundle ID・認証情報（プレースホルダーを置換）
├── Deliverfile                # deliver のデフォルト設定
├── Fastfile                   # lane 定義
├── api_key.json               # App Store Connect API Key ⚠️ gitignore 必須
├── AuthKey_XXXXXXXXXX.p8      # 秘密鍵 ⚠️ gitignore 必須
├── metadata/
│   ├── en-US/                 # 英語メタデータ（必須）
│   │   ├── name.txt
│   │   ├── subtitle.txt
│   │   ├── keywords.txt
│   │   ├── description.txt
│   │   ├── release_notes.txt
│   │   ├── promotional_text.txt
│   │   ├── privacy_url.txt
│   │   ├── support_url.txt
│   │   └── marketing_url.txt
│   └── ja/                    # 日本語メタデータ（任意）
│       └── （同上）
└── screenshots/
    ├── en-US/
    └── ja/
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

### subtitle が 30 文字を超えている

```
'subtitle' cannot be longer than '30' characters
```

→ `metadata/en-US/subtitle.txt` を 30 文字以内に修正。

### メタデータを App Store Connect からダウンロードしたい

```bash
fastlane deliver download_metadata
```
