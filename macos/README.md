# macOSアプリ プロジェクトテンプレート

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
| `{{PROFILE_NAME}}` | プロビジョニングプロファイル名 | `MyApp MAS Distribution` |
| `{{PROFILE_UUID}}` | プロファイルUUID | `d7c282a3-...` |
| `{{API_KEY_ID}}` | App Store Connect API Key ID | `5RJRDGVG38` |

## ディレクトリ構造

```
{{PROJECT_NAME}}/
├── {{PROJECT_NAME}}/
│   └── Sources/
│       ├── App/           # アプリエントリーポイント
│       ├── Models/        # データモデル
│       ├── ViewModels/    # ビジネスロジック
│       ├── Views/         # SwiftUIビュー
│       ├── Services/      # サービス層
│       └── Extensions/    # Swift拡張
├── {{PROJECT_NAME}}Tests/
│   └── ...
├── docs/                  # ドキュメント
├── issues/                # Issue追跡
│   └── done/
├── releases/              # リリースノート
├── signing/               # 署名関連
├── .github/workflows/     # CI/CD
├── project.yml            # XcodeGen設定
├── Makefile
└── .swiftlint.yml
```

## 運用ガイド

詳細は `operations.md` を参照。
