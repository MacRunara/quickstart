# Macrunara 快速开始与 Workflow 模板库

> 面向常见移动技术栈的一键复制 Workflow 模板。所有模板均按
> [runs-on 标签约定](docs/runs-on-labels.md) 使用标准标签
> `runs-on: [macos, arm64, macrunara]`，内置"检出 → 环境信息 → 构建 → 测试 → 产物上传"完整链路。

## 3 分钟接入

1. 在 GitHub Marketplace 安装 **Macrunara** App 到你的组织（或在组织 Settings → GitHub Apps 安装）。
2. 从下方模板索引复制一个模板到你仓库的 `.github/workflows/` 目录，按文件头注释改 scheme / 路径。
3. push 到 `main` 或提 PR，即触发 Mac mini M4 构建。

## 模板索引

| 模板 | 适用 | 一键复制 |
|------|------|----------|
| iOS 原生（xcodebuild） | .xcodeproj / .xcworkspace 工程 | [templates/ios-xcodebuild.yml](templates/ios-xcodebuild.yml) |
| Android（Gradle） | Gradle Wrapper 项目 | [templates/android-gradle.yml](templates/android-gradle.yml) |
| Flutter | Flutter 跨平台应用 | [templates/flutter.yml](templates/flutter.yml) |
| React Native | RN 应用 | [templates/react-native.yml](templates/react-native.yml) |

一键复制方式：打开模板文件 → 右上角 **Copy raw file** → 粘贴到你仓库 `.github/workflows/` 下（文件名可自定）。

## 配套 Action（可选步骤）

- [`macrunara/setup-action@v1`](https://github.com/macrunara/setup-action) — 品牌横幅 + 参数校验 + 环境信息导出。
  使用需在仓库 Secrets 配置 `MACRUNARA_INSTALLATION_ID`（组织 Settings → Installed GitHub Apps → Macrunara 页面 URL 中的数字）。
- [`macrunara/local-mac-runner-action@v1`](https://github.com/macrunara/local-mac-runner-action) — Xcode 构建 / 归档导出 IPA / 产物上传三合一。

## 注意事项

- **runs-on 标签不可随意修改**：标准写法 `[macos, arm64, macrunara]`；写 `self-hosted`、`macos-latest`
  会导致任务永远排队且无报错，排查见 [runs-on 标签约定](docs/runs-on-labels.md)。
- 产物保留期：`actions/upload-artifact@v4` 默认 90 天，可用 `retention-days` 调整。
- 模板默认不做代码签名（CI 仅验证编译链路）；TestFlight / 蒲公英分发请自行扩展签名步骤。
