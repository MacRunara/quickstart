# Macrunara Quickstart & Workflow Templates

**[English](README.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md)**

> Copy-and-go workflow templates for common mobile stacks. All templates use the
> standard label `runs-on: [macos, arm64, macrunara]` per the
> [runs-on label convention](docs/runs-on-labels.md), with a complete
> "checkout → env info → build → test → artifact upload" pipeline.
> **Nodes ship with the full mobile CI environment preinstalled (Xcode / Flutter /
> Android SDK / JDK / Node / CocoaPods) — copy a template and go, nothing to install.**

## 3-minute setup

1. Install the **Macrunara** GitHub App to your organization (GitHub Marketplace,
   or org Settings → GitHub Apps).
2. Copy a template below into your repo's `.github/workflows/` directory and
   adjust scheme / paths per the file header comments.
3. Push to `main` or open a PR — a Mac mini M4 build starts.

## Template index

| Template | Best for | Copy |
|------|------|----------|
| iOS (xcodebuild) | `.xcodeproj` / `.xcworkspace` projects; env preinstalled | [templates/ios-xcodebuild.yml](templates/ios-xcodebuild.yml) |
| Android (Gradle) | Gradle Wrapper projects; env preinstalled | [templates/android-gradle.yml](templates/android-gradle.yml) |
| Flutter | Flutter apps (iOS + Android) | [templates/flutter.yml](templates/flutter.yml) |
| React Native | RN apps | [templates/react-native.yml](templates/react-native.yml) |

Localized comments: **[简体中文注释版](templates/zh-CN/)** · **[日本語コメント版](templates/ja/)**

To copy: open the template file → **Copy raw file** (top right) → paste into
`.github/workflows/` (any filename works).

## Preinstalled environment (macos-mobile v1.2)

| Toolchain | Version | Pin another version |
|------|------|--------------|
| Xcode | 26.x + Command Line Tools | — |
| Flutter SDK | 3.47.2 stable | `subosito/flutter-action@v2` (commented-out example in the template) |
| Android SDK | API 35 + build-tools 35.0.0 + NDK 27.0.12077973 | install extras in-job via `sdkmanager` |
| JDK | temurin 17.0.20 | `actions/setup-java@v4` |
| Node.js | 20.18.0 LTS + yarn 1.22.22 | `actions/setup-node@v4` |
| CocoaPods | 1.17.0 (bundler 2.4.22) | — |

Full version list and image update history: [docs/preinstalled-software.md](docs/preinstalled-software.md).

## FAQ

- **Q: My job stays queued forever with no error.**
  A: The runs-on label is wrong. Use exactly `[macos, arm64, macrunara]`; labels like
  `self-hosted` or `macos-latest` will queue forever. See the
  [runs-on label convention](docs/runs-on-labels.md).
- **Q: Build fails with `flutter: command not found` / `node: command not found`.**
  A: Your job landed on a node running the older v1.1 image. Ask the platform team to
  upgrade to v1.2. Workaround: add `subosito/flutter-action@v2` for Flutter projects,
  or `actions/setup-node@v4` for RN projects (commented-out examples included).
- **Q: Android build fails on SDK licenses not accepted?**
  A: Macrunara node images have all licenses pre-accepted; on your own machine run
  `yes | sdkmanager --licenses`.
- **Q: `pod install` is slow?**
  A: Nodes egress via an internal proxy; the first CocoaPods CDN pull is slower —
  this is normal. You can pin a mirror `source` at the top of your Podfile.

## Notes

- **Don't improvise runs-on labels**: use exactly `[macos, arm64, macrunara]`;
  `self-hosted` / `macos-latest` queue forever with no error — see the
  [runs-on label convention](docs/runs-on-labels.md).
- Artifact retention: `actions/upload-artifact@v4` keeps 90 days by default;
  adjust with `retention-days`.
- Templates do not sign code (CI verifies the build pipeline only); extend with
  your own signing steps for TestFlight / other distribution.
