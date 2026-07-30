# runs-on 标签约定

> Macrunara 平台的 `runs-on` 标签匹配规则、标准写法、禁止清单与排查 FAQ。
> 写错标签是"任务永远 queued"的最常见原因，提交 workflow 前请务必对照本文检查。

---

## 1. 标准写法

Macrunara 节点上报的标准 runner 标签为 **`macos`、`arm64`、`macrunara`**。
客户 workflow 的 `runs-on` 推荐使用完整三标签写法：

```yaml
# 数组形式（推荐，可读性好）
runs-on: [macos, arm64, macrunara]
```

```yaml
# 单行形式（YAML 数组的另一种写法，等价）
runs-on:
  - macos
  - arm64
  - macrunara
```

```yaml
# 单标签简写（合法，只要它是节点标签的子集即可；但推荐写全三个，避免歧义）
runs-on: macrunara
```

## 2. 匹配语义

- **AND 逻辑**：`runs-on` 中列出的**每一个**标签，目标节点都必须具备，job 才会被调度到该节点。
- **子集规则**：`runs-on` 必须是节点上报标签集合的**子集**。多写一个节点没有的标签，匹配结果即为空集——没有任何节点能接单。
- **节点标签构成**：
  - **系统标签**（节点注册时自动上报）：`platform:macos`、`region:*`（如 `region:zhuji`）、`mode:*`（如 `mode:app`）
  - **配置标签**（节点 `agent.yaml` 的 `runner.labels`）：`macos`、`arm64`、`macrunara`

匹配示例：

| workflow 的 runs-on | 是否匹配 | 说明 |
|:---|:---|:---|
| `[macos, arm64, macrunara]` | ✅ | 三标签均为节点上报标签（标准写法） |
| `[macos, macrunara]` | ✅ | 是节点标签的子集 |
| `macrunara` | ✅ | 单子集，合法 |
| `[macos, arm64, macrunara, self-hosted]` | ❌ | `self-hosted` 节点未上报 → 永远排队 |
| `macos-latest` | ❌ | GitHub 托管 Runner 标签，节点未上报 → 永远排队 |
| `[MacOS, ARM64, Macrunara]` | ❌ | 标签**区分大小写** → 永远排队 |

## 3. 禁止标签清单

以下标签 **Macrunara 节点不会上报**，写入 `runs-on` 将导致任务**永远排队**（不会有任何报错提示，job 状态停留在 queued）：

| 禁止标签 | 原因 |
|:---|:---|
| `self-hosted` | 平台 JIT ephemeral runner 不上报此通用标签 |
| `macos-latest` / `macos-14` / `macos-15` | GitHub 托管 Runner 专用标签 |
| `ubuntu-latest` / `windows-latest` 等 | 非 macOS 平台标签 |
| `x64` / `amd64` | 节点为 Apple Silicon（arm64），不上报 x64 |
| 任何自定义拼写（如 `mac-runara`、`macruna`） | 拼写错误等同未上报标签 |

> **后果说明**：GitHub Actions 对无匹配标签的 job 不做即时失败处理，会一直等待"理论上可能出现的 Runner"。Macrunara 平台侧同样无法为其调度节点。请按第 6 节 FAQ 排查。

## 4. 预留标签（未来精细化调度）

以下标签为平台**预留**，当前节点尚未上报，仅供文档说明，**现阶段写入 runs-on 同样会导致排队**：

| 预留标签 | 规划用途 |
|:---|:---|
| `xcode:26.x` | 按节点 Xcode 大版本精细化调度（如 `xcode:26.0`、`xcode:26.1`） |
| `region:*` | 按机房区域直接指定调度目标（如 `region:zhuji`、`region:fuyang`）；现阶段请通过 setup-action 的 `region` 输入或平台默认调度表达区域偏好 |

平台开放这些标签后会另行公告，届时本文将同步更新。

## 5. 完整示例 workflow 片段

```yaml
name: macOS CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    # 标准三标签写法：AND 匹配，且是节点上报标签的子集
    runs-on: [macos, arm64, macrunara]

    steps:
      - uses: actions/checkout@v4

      # 可选步骤：品牌横幅 + 参数校验 + 环境信息导出（非必需）
      - uses: macrunara/setup-action@v1
        with:
          installation-id: ${{ secrets.MACRUNARA_INSTALLATION_ID }}
          region: auto

      - name: Build
        run: xcodebuild -scheme MyApp -destination 'platform=macOS'

      - name: Test
        run: swift test
```

更多完整模板（iOS / Android / Flutter / React Native）见 [templates/ 模板库](https://github.com/macrunara/quickstart#模板索引)。

## 6. 排查 FAQ

### Q: 任务一直停在 queued，没有任何节点接单，怎么办？

按以下顺序排查：

1. **检查标签拼写**：逐字符核对 `runs-on`，常见错误：`macruna`、`mac-runara`、`macrunara-app`（旧写法，已废弃）。
2. **检查大小写**：标签**区分大小写**，必须全小写：`macos` ✅ / `MacOS` ❌。
3. **检查是否子集**：`runs-on` 中的每个标签都必须在节点上报集合内。逐一删掉可疑标签（尤其是 `self-hosted`、`macos-latest`）再触发验证。
4. **检查 YAML 写法**：数组形式 `runs-on: [macos, arm64, macrunara]` 与多行形式等价；不要写成逗号拼接的单个字符串 `runs-on: "macos, arm64, macrunara"`（这会被当成**一个**标签）。
5. **确认队列无历史残留**：GitHub 队列为 FIFO，若队列中有带错误标签的旧 job 占位，请先取消（Actions 页面 → Cancel workflow）。
6. **确认订阅与配额**：登录 Macrunara 管理后台确认订阅有效、配额未满（排队超过 5 分钟时的常规检查项）。

### Q: 为什么不像 GitHub 托管 Runner 那样写 `macos-latest`？

Macrunara 节点是**自托管性质的 ephemeral runner**，标签集合由节点注册时上报决定，与 GitHub 托管 Runner 的标签体系无关。平台选择 `macos` / `arm64` / `macrunara` 三标签，分别表达"操作系统"、"CPU 架构"、"平台归属"三个维度。

### Q: 能否只写 `runs-on: macos`？

可以（是子集，合法），但不推荐：仅写 `macos` 无法区分其他可能上报 `macos` 标签的 Runner。推荐始终使用 `[macos, arm64, macrunara]` 完整写法。

### Q: setup-action 会帮我修改 Runner 标签吗？

**不会，也不能**。Runner 标签在注册时即固定，job 内无法动态修改（GitHub Actions 不存在此类机制）。`macrunara/setup-action@v1` 的作用是品牌横幅、参数校验与环境信息导出，属于可选步骤。标签只能由你在 workflow 的 `runs-on` 字段中正确声明。
