# qb-gate-codex-egress

**QB Gate 的「Codex 出站与换出口」插件。** 一个独立运行的本地程序（Go），负责 Codex 的
代理 / 订阅 / 机场出站，以及实验性的「换出口凑 292」；由 QB Gate 的扩展中心接入（启动、停止、
打开面板），但**不属于 QB Gate 核心** —— 核心不内置代理、不换出口、不换账户。

本仓库是 [gylive/ccodex-sleep-state](https://github.com/gylive/ccodex-sleep-state) 的 **fork**
（GPL-3.0，见 `LICENSE` 与 `NOTICE.md`）。引擎、网页面板、订阅解析与出站适配都来自上游；
本仓库只增加 QB Gate 的接入约定（`qb-gate-plugin.json`）、本 README 与 `DISCLAIMER.md`。

## 它做什么（大白话）

- **turn-state「292 / 332」**：从你自己的请求响应里读 `X-Codex-Turn-State`，看它「几块、多长」，
  缺失时补一张再发。**长度只是经验筛选口径，不是模型质量或额度指标，也不增加额度。**
- **代理 / 订阅 / 机场出站**：填本地 HTTP / SOCKS5，或导入 Clash / Mihomo 订阅、逐行 URI、Base64 订阅；
  出站适配 SS / SSR / VMess / VLESS / Trojan / Hysteria / Hysteria2 / TUIC / AnyTLS（来自 Mihomo）。
- **换出口凑 292（实验）**：在候选出口之间发短探测，保存合格的 turn-state 并走对应出口。
  **探测会消耗你自己的额度**；默认每轮最多 6 个候选、两轮至少间隔 180 秒；401 / 403 / 429 即停。

## 与 QB Gate 的关系

| | QB Gate 核心 | 本插件 |
|---|---|---|
| 官方 Codex 识别（turn-state） | 有，**被动**采集、只在当前一条连接上、不换出口 | 有，主动探测、可换出口 |
| 代理 / 订阅 / 机场 | **没有**（合规边界） | 有（Mihomo） |
| 接管哪个 Codex | 独立环境目录 + 本机路由，不碰你的默认 `~/.codex` | **接管你真实的 `~/.codex/config.toml`**（退出时恢复） |
| 谁来起停 | — | QB Gate 扩展中心（或你自己双击 `start.cmd`） |

**两者互斥。** QB Gate 的官方识别线挂在本机路由上时，扩展中心会拒绝启动本插件（后端守卫）；
反过来本插件在跑时，别去点 QB Gate 的「接入并启动 Codex（识别模式）」。

## 安装（Windows）

1. 到本仓库 **Releases** 下载 `windows-amd64` 发布包（由 CI 构建），完整解压到
   `%LOCALAPPDATA%\Programs\ccodex-sleep-state`（QB Gate 默认在这里找；放别处就在插件页指定
   `ccodex-sleep-state.exe` 的位置）。
2. 打开 QB Gate → 扩展中心 → 「Codex 出站与换出口」→ **启动插件**。它会新开一个控制台窗口、
   备份并接管 Codex 配置、起服务，并打开自己的网页面板（默认 `http://127.0.0.1:17841/admin/`）。
3. **重启 Codex、新建会话**（旧会话不会热切换）。在它的面板里配置订阅 / 代理 / 注入开关。
4. 退出：在 QB Gate 点 **停止并恢复 Codex 配置**（结束进程 + 跑本程序的 `restore`），
   或到它的窗口按 **Ctrl+C**（走它自己的恢复流程）。**不要直接结束进程后不管** —— Codex 会留在
   一个指向死服务的配置上（症状是 503）；那时运行 `ccodex-sleep-state.exe restore`。

不装 QB Gate 也能用：解压后双击 `start.cmd`，用法见 `docs/UPSTREAM-README.md`（上游原文）。

## 自己构建

需要 Go（版本见 `go.mod`）。仓库根：

```powershell
go build -o ccodex-sleep-state.exe ./cmd/ccodex-sleep-state
```

`.github/workflows/release.yml` 在打 tag（`v*`）时构建四个平台的发布包并附上 SHA256SUMS。

## 边界与免责

见 `DISCLAIMER.md`。要点：只接受你自己配置的出站；不改系统代理、不刷新 Codex 登录、不使用重置卡；
不承诺任何服务端结果；是否使用、是否符合服务商条款与所在地法律，由你自行判断并承担后果。

## 许可证

GPL-3.0，见 `LICENSE`。上游归属与第三方依赖见 `NOTICE.md`、`THIRD_PARTY_NOTICES.md`。
本仓库与 OpenAI、Mihomo、QB Gate 的作者之间没有隶属关系。
