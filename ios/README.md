# iOSアプリ プロジェクトテンプレート

## macOSテンプレートとの主な差分

| ファイル | 差分 |
|---------|------|
| `project.yml.template` | platform: iOS、deploymentTarget: iOS 17.0、署名方式（Distribution / MAS） |
| `Makefile.template` | シミュレータ経由のビルド・起動、ipa出力 |
| `signing/` | `embedded.mobileprovision` / `ExportOptions.plist` |

---

## 使い方

1. このディレクトリの内容を新しいプロジェクトにコピー
2. `project.yml.template` を `project.yml` にリネームし、プレースホルダーを置換
3. `Makefile.template` を `Makefile` にリネームし、プレースホルダーを置換
4. `make setup` でXcodeプロジェクトを生成

## プレースホルダー一覧

| プレースホルダー | 説明 | 例 |
|-----------------|------|-----|
| `{{PROJECT_NAME}}` | プロジェクト名 | `MyApp` |
| `{{BUNDLE_ID}}` | バンドルID | `com.example.myapp` |
| `{{TEAM_ID}}` | Apple Developer Team ID | `ABC123DEF4` |
| `{{PROFILE_NAME}}` | プロビジョニングプロファイル名 | `MyApp App Store` |
| `{{PROFILE_UUID}}` | プロファイルUUID | `d7c282a3-...` |
| `{{API_KEY_ID}}` | App Store Connect API Key ID | `5RJRDGVG38` |
| `{{API_ISSUER_ID}}` | App Store Connect API Issuer ID | `69a6de83-...` |

## 運用ガイド

詳細は `operations.md` を参照。
