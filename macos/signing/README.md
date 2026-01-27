# 署名関連ファイル

## 必要ファイル

| ファイル | 用途 | 取得方法 |
|---------|------|---------|
| `embedded.provisionprofile` | プロビジョニングプロファイル | Apple Developer Portal |
| `entitlements.mas.plist` | MAS用エンタイトルメント | 手動作成 |
| `entitlements.mas.inherit.plist` | 継承用エンタイトルメント | 手動作成 |
| `ExportOptions-MAS.plist` | MASエクスポート設定 | 手動作成 |
| `MAS.xcconfig` | MAS用Xcode設定 | 手動作成 |

## entitlements.mas.plist 例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.security.app-sandbox</key>
    <true/>
    <key>com.apple.security.files.user-selected.read-write</key>
    <true/>
    <key>com.apple.security.network.client</key>
    <true/>
</dict>
</plist>
```

## ExportOptions-MAS.plist 例

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
</dict>
</plist>
```

## プロファイルのインストール

```bash
make install-profile
```
