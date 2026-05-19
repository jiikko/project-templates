# Localization テンプレート (macOS)

`Localizable.xcstrings` (String Catalog) + Swift Testing 製の整合性テスト
+ test scheme の言語固定をまとめて新規 app に持ち込むためのテンプレート。

## このディレクトリの中身

| ファイル | 何のため |
|---------|---------|
| [`Localizable.xcstrings`](Localizable.xcstrings) | 空 catalog の skeleton (`sourceLanguage: "en"`、`strings: {}`)。app の `Resources/` 配下にコピーして start |
| [`LocalizableCatalogTests.swift.template`](LocalizableCatalogTests.swift.template) | catalog の **整合性検証 test** (Swift Testing)。app の `<Name>Tests/` にコピーし、`{{PROJECT_NAME}}` を実プロジェクト名に置換 |
| [`postBuildScript.yml.snippet`](postBuildScript.yml.snippet) | `project.yml` の test target に追加する postBuildScript (生 JSON を test bundle に同梱する shell + inputFiles / outputFiles) |

## このテンプレートが解く問題

1. **ローカライズ漏れ regression が test で catch されない**
   - 新規 literal を View に書いて catalog 登録忘れ → ja 環境で en のまま出る
   - 動的 string interpolation を `String(localized:)` に書いてしまう (`String(localized: "Failed: \(error.localizedDescription)")`) → catalog miss が静かに発生
   - en 環境で日本語 literal が直書きされている → ja 環境前提の文言が en でも出る
2. **test の runtime 言語依存**
   - `String(localized:)` / `LocalizedStringResource` は OS の `preferredLocalizations` を引く
   - ViewInspector で render 後 Text を assert する test が「CI / ローカルが ja か en か」で結果が変わる脆い状態になる

## 採用 (one-shot 手順)

### 1. catalog skeleton をコピー

```bash
# 新規 app のリポジトリで
APP=MyApp   # project の実名
cp project-templates/macos/localization/Localizable.xcstrings \
   "$APP/Resources/Localizable.xcstrings"
```

`project.yml` の app target が `sources: - path: $APP/Resources` を含む
ことを確認 (catalog は app bundle に自動で焼かれる)。

### 2. 整合性テストをコピー

```bash
# `.template` 拡張子を外し、{{PROJECT_NAME}} を実名に置換
sed "s/{{PROJECT_NAME}}/$APP/g" \
  project-templates/macos/localization/LocalizableCatalogTests.swift.template \
  > "${APP}Tests/LocalizableCatalogTests.swift"
```

### 3. `project.yml` の test target に postBuildScript を追加

`postBuildScript.yml.snippet` の中身を `<Name>Tests` target の
`postBuildScripts:` セクションに貼り付け、`{{PROJECT_NAME}}` を実名に置換。
`inputFiles` / `outputFiles` は **必須** (Xcode の incremental build に
入れないと毎回 always-run になり遅くなる)。

例 (Obaket の場合の最終形):

```yaml
targets:
  ObaketTests:
    type: bundle.unit-test
    platform: macOS
    sources:
      - path: ObaketTests
    # ... 省略 ...
    postBuildScripts:
      - name: Copy Localizable.xcstrings into test bundle for LocalizableCatalogTests
        script: |
          set -euo pipefail
          unset CDPATH
          SRC="${SRCROOT}/Obaket/Resources/Localizable.xcstrings"
          # ... snippet の中身 ...
        inputFiles:
          - ${SRCROOT}/Obaket/Resources/Localizable.xcstrings
        outputFiles:
          - ${BUILT_PRODUCTS_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}/LocalizableSource.json
```

### 4. `project.yml` の scheme に言語固定を入れる

```yaml
schemes:
  MyApp:
    # ...
    test:
      config: Debug
      commandLineArguments:
        "-AppleLanguages (en)": true
        "-AppleLocale en_US": true
      targets:
        - MyAppTests
```

これで `String(localized:)` が test 実行時に **常に en source を返す**
ようになり、ViewInspector / `Text` assert が runtime 言語に依存しない。

### 5. `project.yml` の options に developmentLanguage を明示

```yaml
options:
  developmentLanguage: en
```

### 6. 再生成 + test 実行

```bash
make setup
make build
make test
```

`Suite "Localizable catalog integrity" passed` が出れば成功。catalog が
空 (`strings: {}`) でも 5 つの test は全 pass する (空集合の検証)。

## 採用後の運用

| 操作 | やり方 |
|------|-------|
| 新規 UI literal の追加 | View に `Text("New Label")` 等を書く → `Localizable.xcstrings` に `{"New Label": { "extractionState": "manual", "localizations": { "ja": { "stringUnit": { "state": "translated", "value": "新しいラベル" } } } }}` を追加 |
| 動的文字列 | **`String(localized:)` の引数に interpolation を入れない**。`String(localized: "Failed to save") + ": \(reason)"` のように連結する (test #5 が catch する) |
| 翻訳途中 | `extractionState: "stale"` ではなく未登録のまま残す。stale は **削除すべき key の signal** に予約 |
| 固有名詞 (商標、フォント名等) で日本語が key 自身に残る場合 | `LocalizableCatalogTests.proverbAllowlist` に追加 |

## 個別アプリでの carve-out

「整合性テストの一部を skip したい」「ja 以外の言語を追加したい」「stale を
許容したい」場合は、コピー後の `LocalizableCatalogTests.swift` を **派生
ファイルとして直接編集** してよい。`fastlane/Fastfile.macos.template` と
違って **template の正本性は厳しく守らない** 方針:

- このテンプレートの目的は **「初期構成を 5 分で揃える」** こと
- 後の app 固有調整 (proverbAllowlist の拡張、新言語追加、forbiddenSubstrings
  の app 固有追加) はその app の事情を反映するべきで、template に逆流
  させる必要はない

## 関連

- 採用事例: `apps/ThumbnailThumb/ThumbnailThumbTests/LocalizableCatalogTests.swift` (XCTest 版、原型)
- 採用事例: `apps/obaket/macOS/ObaketTests/LocalizableCatalogTests.swift` (Swift Testing 版、このテンプレート起点)
- 関連ルール: `.claude/rules/standard-app-components.md` (Toast / Settings 共通化)
- Apple 公式: WWDC23 "Discover String Catalogs"、"Testing localizations when running your app"
