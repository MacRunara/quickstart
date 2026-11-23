# Macrunara 快速开始与 Workflow 模板库

> 面向常见移动技术栈的一键复制 Workflow 模板。所有模板均按
> [runs-on 标签约定](docs/runs-on-labels.md) 使用标准标签
> `runs-on: [macos, arm64, macrunara]`，内置"检出 → 环境信息 → 构建 → 测试 → 产物上传"完整链路。
> **节点已预装全套移动 CI 环境（Xcode / Flutter / Android SDK / JDK / Node / CocoaPods），模板复制即用，无需自行安装。**

## 3 分钟接入

1. 在 GitHub Marketplace 安装 **Macrunara** App 到你的组织（或在组织 Settings → GitHub Apps 安装）。
2. 从下方模板索引复制一个模板到你仓库的 `.github/workflows/` 目录，按文件头注释改 scheme / 路径——**四类模板任选，环境无需自行安装**。
3. push 到 `main` 或提 PR，即触发 Mac mini M4 构建。

## 模板索引

| 模板 | 适用 | 一键复制 |
|------|------|----------|
| iOS 原生（xcodebuild） | .xcodeproj / .xcworkspace 工程；环境已预装，复制即用 | [templates/ios-xcodebuild.yml](templates/ios-xcodebuild.yml) |
| Android（Gradle） | Gradle Wrapper 项目；环境已预装，复制即用 | [templates/android-gradle.yml](templates/android-gradle.yml) |
| Flutter | Flutter 跨平台应用；环境已预装，复制即用 | [templates/flutter.yml](templates/flutter.yml) |
| React Native | RN 应用；环境已预装，复制即用 | [templates/react-native.yml](templates/react-native.yml) |

一键复制方式：打开模板文件 → 右上角 **Copy raw file** → 粘贴到你仓库 `.github/workflows/` 下（文件名可自定）。

## 节点预装环境（macos-mobile v1.2）

| 环境 | 版本 | 锁定其他版本 |
|------|------|--------------|
| Xcode | 26.x + Command Line Tools | — |
| Flutter SDK | 3.47.2 stable | `subosito/flutter-action@v2`（模板里有注释好的可选步骤） |
| Android SDK | API 35 + build-tools 35.0.0 + NDK 27.0.12077973 | job 内 `sdkmanager` 补装 |
| JDK | temurin 17.0.20 | `actions/setup-java@v4` |
| Node.js | 20.18.0 LTS + yarn 1.22.22 | `actions/setup-node@v4` |
| CocoaPods | 1.17.0（bundler 2.4.22） | — |

完整版本清单与镜像更新记录见 [docs/preinstalled-software.md](docs/preinstalled-software.md)。

## FAQ

- **Q: 任务一直排队，没有任何报错？**
  A: runs-on 标签写错了。标准写法 `[macos, arm64, macrunara]`；写 `self-hosted`、`macos-latest` 会永远排队。详见 [runs-on 标签约定](docs/runs-on-labels.md)。
- **Q: 构建报 `flutter: command not found` / `node: command not found`？**
  A: 你的 job 落在了 v1.1 旧镜像节点上。联系平台方升级到 v1.2；临时方案：Flutter 项目加 `subosito/flutter-action@v2`，RN 项目加 `actions/setup-node@v4`（模板里有注释好的示例）。
- **Q: Android 构建报 SDK licenses 未接受？**
  A: Macrunara 平台节点镜像已接受全部 licenses；如果你用的是自备节点，执行 `yes | sdkmanager --licenses`。
- **Q: pod install 很慢？**
  A: 节点走内网代理出网，CocoaPods CDN 首次拉取较慢属正常；可在 Podfile 顶部指定 `source` 镜像加速。

## 注意事项

- **runs-on 标签不可随意修改**：标准写法 `[macos, arm64, macrunara]`；写 `self-hosted`、`macos-latest`
  会导致任务永远排队且无报错，排查见 [runs-on 标签约定](docs/runs-on-labels.md)。
- 产物保留期：`actions/upload-artifact@v4` 默认 90 天，可用 `retention-days` 调整。
- 模板默认不做代码签名（CI 仅验证编译链路）；TestFlight / 蒲公英分发请自行扩展签名步骤。
