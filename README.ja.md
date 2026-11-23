# Macrunara クイックスタート & Workflow テンプレート集

**[English](README.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md)**

> 主要モバイル技術スタック向けのコピー＆ゴー Workflow テンプレート。
> すべてのテンプレートは [runs-on ラベル規約](docs/runs-on-labels.md) に従い
> 標準ラベル `runs-on: [macos, arm64, macrunara]` を使用し、
> 「チェックアウト → 環境情報 → ビルド → テスト → 成果物アップロード」の
> 完全なパイプラインを内蔵しています。
> **ノードにはモバイル CI 環境がすべてプリインストール済み（Xcode / Flutter /
> Android SDK / JDK / Node / CocoaPods）。テンプレートをコピーするだけで、
> インストール不要です。**

## 3 分で導入

1. **Macrunara** GitHub App を組織にインストール（GitHub Marketplace、
   または組織の Settings → GitHub Apps から）。
2. 下のテンプレート一覧から 1 つをリポジトリの `.github/workflows/` に
   コピーし、ファイル先頭のコメントに従って scheme / パスを調整。
3. `main` に push（または PR を作成）すると、Mac mini M4 でビルド開始。

## テンプレート一覧

| テンプレート | 適用 | コピー |
|------|------|----------|
| iOS ネイティブ（xcodebuild） | `.xcodeproj` / `.xcworkspace` プロジェクト | [templates/ja/ios-xcodebuild.yml](templates/ja/ios-xcodebuild.yml) |
| Android（Gradle） | Gradle Wrapper プロジェクト | [templates/ja/android-gradle.yml](templates/ja/android-gradle.yml) |
| Flutter | Flutter アプリ（iOS + Android） | [templates/ja/flutter.yml](templates/ja/flutter.yml) |
| React Native | RN アプリ | [templates/ja/react-native.yml](templates/ja/react-native.yml) |

他言語のコメント版：**[English comments](templates/)** · **[简体中文注释版](templates/zh-CN/)**

コピー方法：テンプレートを開く → 右上の **Copy raw file** →
`.github/workflows/` に貼り付け（ファイル名は自由）。

## プリインストール環境（macos-mobile v1.2）

| 環境 | バージョン | 別バージョンの固定 |
|------|------|--------------|
| Xcode | 26.x + Command Line Tools | — |
| Flutter SDK | 3.47.2 stable | `subosito/flutter-action@v2`（テンプレート内にコメント済みの例あり） |
| Android SDK | API 35 + build-tools 35.0.0 + NDK 27.0.12077973 | job 内で `sdkmanager` により追加 |
| JDK | temurin 17.0.20 | `actions/setup-java@v4` |
| Node.js | 20.18.0 LTS + yarn 1.22.22 | `actions/setup-node@v4` |
| CocoaPods | 1.17.0（bundler 2.4.22） | — |

完全なバージョン一覧とイメージ更新履歴は
[docs/preinstalled-software.md](docs/preinstalled-software.md) を参照。

## FAQ

- **Q: ジョブがエラーなく永久にキュー待ちになる。**
  A: runs-on ラベルが間違っています。正確に `[macos, arm64, macrunara]` と
  書いてください。`self-hosted` や `macos-latest` では永久に待ち続けます。
  詳しくは [runs-on ラベル規約](docs/runs-on-labels.md)。
- **Q: `flutter: command not found` / `node: command not found` で失敗する。**
  A: 旧 v1.1 イメージのノードに割り当てられています。プラットフォーム側に
  v1.2 へのアップグレードを依頼してください。暫定策：Flutter なら
  `subosito/flutter-action@v2`、RN なら `actions/setup-node@v4` を追加
  （コメント済みの例がテンプレート内にあります）。
- **Q: Android ビルドが SDK ライセンス未承認で失敗する。**
  A: Macrunara のノードイメージは全ライセンス承認済みです。自前のマシンでは
  `yes | sdkmanager --licenses` を実行してください。
- **Q: `pod install` が遅い。**
  A: ノードは社内プロキシ経由で外部アクセスするため、CocoaPods CDN の
  初回取得は遅くなります（正常です）。Podfile 先頭でミラーの `source` を
  指定すると高速化できます。

## 注意事項

- **runs-on ラベルは変更禁止**：正確に `[macos, arm64, macrunara]`。
  `self-hosted` / `macos-latest` はエラーなく永久にキュー待ちになります。
  詳しくは [runs-on ラベル規約](docs/runs-on-labels.md)。
- 成果物の保持期間：`actions/upload-artifact@v4` はデフォルト 90 日。
  `retention-days` で調整可能。
- テンプレートはコード署名を行いません（CI はビルドの疎通確認のみ）。
  TestFlight 等への配信は署名ステップを各自で追加してください。
