# 署名関連ファイル (iOS)

## 必要ファイル

| ファイル | 用途 | 取得方法 |
|---------|------|---------|
| `embedded.mobileprovision` | プロビジョニングプロファイル | Apple Developer Portal |
| `ExportOptions.plist` | App Storeエクスポート設定 | 手動作成 |

## ExportOptions.plist 例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>destination</key>
    <string>export</string>
    <key>method</key>
    <string>app-store</string>
    <key>signingStyle</key>
    <string>manual</string>
    <key>teamID</key>
    <string>{{TEAM_ID}}</string>
    <key>provisioningProfiles</key>
    <dict>
        <key>{{BUNDLE_ID}}</key>
        <string>{{PROFILE_NAME}}</string>
    </dict>
    <key>uploadSymbols</key>
    <true/>
</dict>
</plist>
```

## プロファイルのインストール

```bash
make install-profile
```
