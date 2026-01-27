# iOSアプリ プロジェクトテンプレート

## macOSテンプレートとの関係

**ベース: `macos/`**

macOSアプリテンプレートをベースとしており、運用フローは共通。
差分は `project.yml` のプラットフォーム設定のみ:

```yaml
# macos
platform: macOS
deploymentTarget:
  macOS: "14.0"

# ios
platform: iOS
deploymentTarget:
  iOS: "17.0"
```

今後iOS固有の運用が必要になった場合はこのディレクトリに追加する。

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

## 運用ガイド

詳細は `operations.md` を参照。
