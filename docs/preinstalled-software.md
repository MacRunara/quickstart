# Macrunara 节点预装软件清单

> 对应节点镜像 `macos-mobile`（Harbor: `172.16.0.165:5000/macos-ci/macos-mobile`）。
> 每次镜像升版（v1.2 → v1.3 …）同步更新本页。

## 当前版本：v1.2（2026-09-08 发布，sha256:bb0e80c7…）

### 系统与 Xcode

| 软件 | 版本 | 说明 |
|------|------|------|
| macOS | 26.x | Apple Silicon（arm64） |
| Xcode | 26.x | 含 iOS Simulator runtime |
| Command Line Tools | 26.6 | 含 git / clang 等基础工具链 |
| Rosetta 2 | 已安装 | 部分旧工具链需要 |

### 移动 CI 环境（v1.2 新增/替换）

| 软件 | 版本 | 安装位置 | 备注 |
|------|------|----------|------|
| Flutter SDK | **3.47.2 stable** | `~/flutter` | v1.1 是 master 通道预览版，v1.2 整目录替换为 stable；已 `precache --ios --android`（不含 web/桌面端） |
| Node.js | **20.18.0 LTS** | `/usr/local/bin` | v1.2 新增；含 npm 10.8.2 |
| yarn | 1.22.22 | `/usr/local/bin` | v1.2 新增 |
| CocoaPods | **1.17.0** | 系统 ruby gem | v1.2 新增；ffi 1.17.4 |
| bundler | 2.4.22 | 系统 ruby gem | v1.2 新增 |

### 既有环境（v1.1 已有，v1.2 沿用）

| 软件 | 版本 | 安装位置 |
|------|------|----------|
| JDK（temurin） | 17.0.20 | `/Library/Java/JavaVirtualMachines` |
| Android SDK Platform | API 35（另有 36） | `~/Library/Android/sdk/platforms` |
| Android build-tools | 35.0.0 | `~/Library/Android/sdk/build-tools` |
| Android NDK | 27.0.12077973（另有 26.3） | `~/Library/Android/sdk/ndk` |
| Android platform-tools | 37.0.1 | `~/Library/Android/sdk/platform-tools` |
| Ruby | 2.6.10（系统自带） | `/usr/bin/ruby` |

### 环境变量

镜像内 `~/.zprofile` 已配置（登录 shell 生效）；CI job 内由各 `build-actions` 子动作自动写入 `$GITHUB_PATH` / `$GITHUB_ENV`，无需手动 export：

- `ANDROID_HOME=$HOME/Library/Android/sdk`
- `JAVA_HOME=$(/usr/libexec/java_home -v 17)`
- PATH 追加：`/usr/local/bin`、`~/flutter/bin`、`$ANDROID_HOME/cmdline-tools/latest/bin`、`$ANDROID_HOME/platform-tools`

### 刻意不预装

- **Android 模拟器系统镜像**（单个 10GB+，CI 只做编译+单测；需要的项目 job 内 `sdkmanager "system-images;..."` 自装）
- Flutter web / macOS 桌面端 precache
- Chrome（flutter doctor 的 web 检查项会报缺失，不影响 iOS/Android 构建）

### 版本核查命令（在节点 VM 内执行）

```bash
flutter --version          # Flutter 3.47.2 • channel stable
node --version             # v20.18.0
pod --version              # 1.17.0
java -version              # 17.0.20
sdkmanager --list_installed
xcodebuild -version
```

## 历史版本

| 镜像 | 发布日期 | 变更 |
|------|----------|------|
| v1.1 | 2026-08 | Xcode 26 + JDK 17 + Android SDK + Flutter master（预览版，CI 不可用） |
| v1.2 | 2026-09-08 | Flutter 换 stable 3.47.2；新增 Node 20 + yarn、CocoaPods 1.17.0、bundler；precache --ios --android |
