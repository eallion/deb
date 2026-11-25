# eallion/deb

基于 [deb-get](https://github.com/wimpysworld/deb-get) 的个人第三方软件包扩展仓库（External Repository）。

本仓库仅提供软件包解析与抓取定义，不存储任何 `.deb` 实体文件。所有安装包均在客户端执行时通过 GitHub API 或官方节点动态解析并下载自官方 Release。

## 客户端接入方式

在 Ubuntu 客户端执行以下命令添加本仓库：

```bash
echo "https://raw.githubusercontent.com/eallion/deb/main/eallion" | sudo tee /etc/deb-get/eallion.repo
sudo deb-get update
```

## 基础使用（标准软件）

对于标准软件，直接执行对应的安装命令即可，后续通过 `sudo deb-get upgrade` 自动追踪更新。

以 `anyzone` 为例：

```bash
# 安装
sudo deb-get install anyzone

# 全量更新
sudo deb-get update
sudo deb-get upgrade
```

## 已收录软件清单

| 软件名 (APP) | 类型 / 说明 | 上游来源 | 基础安装命令 |
| :--- | :--- | :--- | :--- |
| `anytype` | Anytype 官方桌面客户端（默认正式稳定版） | [anyproto/anytype-ts](https://github.com/anyproto/anytype-ts) | `sudo deb-get install anytype` |
| `anyzone` | Anyzone 实用工具 | [eallion/anyzone](https://github.com/eallion/anyzone) | `sudo deb-get install anyzone` |
| `brows3` | 基于 Rust/Tauri 的 S3 桌面客户端 | [rgcsekaraa/brows3](https://github.com/rgcsekaraa/brows3) | `sudo deb-get install brows3` |
| `easytier` | 去中心化 Mesh VPN 核心服务（Core/CLI） | [EasyTier/Easytier](https://github.com/EasyTier/Easytier) | `sudo deb-get install easytier` |
| `easytier-gui` | EasyTier 官方图形界面桌面端（GUI） | [EasyTier/Easytier](https://github.com/EasyTier/Easytier) | `sudo deb-get install easytier-gui` |
| `flclash` | 基于 ClashMeta 的跨平台代理客户端 | [chen08209/FlClash](https://github.com/chen08209/FlClash) | `sudo deb-get install flclash` |
| `kubo` | IPFS 官方 Go 语言实现核心程序（提供 `ipfs` CLI） | [ipfs/kubo](https://github.com/ipfs/kubo) | `sudo deb-get install kubo` |
| `linuxqq` | 腾讯 QQ Linux 官方客户端 | [im.qq.com](https://im.qq.com/linuxqq/) | `sudo deb-get install linuxqq` |
| `misterdownloadmanager` | 现代化高速桌面下载器 | [anefzaoui/misterdownloadmanager](https://github.com/anefzaoui/misterdownloadmanager) | `sudo deb-get install misterdownloadmanager` |
| `moke` | 墨客 Talebook 个人书库客户端 | [talebook/moke](https://github.com/talebook/moke) | `sudo deb-get install moke` |
| `mygit` | 轻量级原生 Git 客户端工具 | [Binaryify/MyGit](https://github.com/Binaryify/MyGit) | `sudo deb-get install mygit` |
| `piclist` | 基于 PicGo 的云存储管理与图床工具 | [Kuingsmile/piclist](https://github.com/Kuingsmile/piclist) | `sudo deb-get install piclist` |
| `rustdesk-unattended-wayland` | RustDesk 远程桌面（Wayland 无人值守专用版） | [rustdesk/rustdesk](https://github.com/rustdesk/rustdesk) | `sudo deb-get install rustdesk-unattended-wayland` |
| `switch-hosts` | Hosts 文件管理与切换工具 | [oldj/switchHosts](https://github.com/oldj/switchHosts) | `sudo deb-get install switch-hosts` |
| `wechat` | 腾讯微信 Linux 官方客户端 | [linux.weixin.qq.com](https://linux.weixin.qq.com/) | `sudo deb-get install wechat` |

---

## 版本通道选择（正式版与测试版）

### 默认通道：锁定稳定正式版
所有 GitHub 托管的软件包默认均通过过滤规则自动排除 `alpha`、`beta`、`rc`、`preview`、`dev`、`nightly` 构建，确保默认安装和升级时仅拉取正式发布的稳定版本。

### 可选通道：体验预发布/测试版
如果需要安装或测试上游的最新 Alpha / Beta / Pre-release 版本，可以在执行命令时通过环境变量指定通道：

```bash
# 方式 1：指定 beta 通道
sudo DEBGET_CHANNEL=beta deb-get install <app>

# 方式 2：开启预发布标记
sudo DEBGET_PRERELEASE=1 deb-get install <app>
```

> **注意**：由于 `sudo` 默认会重置环境变量，若希望临时传入的环境变量生效，请确保已按照下方说明配置了 `/etc/sudoers.d/debget_token` 中的环境变量保留规则。

---

## GitHub API 请求配额与速率限制优化

### 背景与限流说明
GitHub API 对未提供认证 Token 的匿名 IP 请求限制为 **60 次/小时**。当收录的软件较多或频繁运行 `deb-get update` 时，容易触发 HTTP 403 速率限制（Rate Limit Exceeded）。

配置 GitHub Personal Access Token（经典 Token 仅需默认读取权限或无 scope 的细粒度 Token 即可）后，API 请求配额将直接提升至 **5000 次/小时**。

### 第一步：配置 Sudo 环境变量保留（必须执行）
Ubuntu 的 `sudo` 默认启用了安全策略 `env_reset`，**执行任何 `sudo` 命令时都会强制清空自定义环境变量**（包括从 `/etc/environment` 或用户 Shell 中导出的变量）。

因此，必须配置一行 Sudoers 白名单规则，允许 `sudo` 保留 `DEBGET_TOKEN` 及可选通道变量：

```bash
echo 'Defaults env_keep += "DEBGET_TOKEN DEBGET_CHANNEL DEBGET_PRERELEASE"' | sudo tee /etc/sudoers.d/debget_token
```

> 配置此白名单后，无论通过以下哪种方式设置变量，执行 `sudo deb-get update` 或 `sudo DEBGET_CHANNEL=beta deb-get install <app>` 时都能正常读取变量，无需每次手动添加 `-E` 参数。

---

### 第二步：选择环境变量注入方式

#### 方案 A：跨 Shell 统一标准方案（推荐）
直接将 Token 写入系统级环境配置文件 `/etc/environment`：

```bash
echo 'DEBGET_TOKEN="ghp_你的GitHubToken"' | sudo tee -a /etc/environment
```

* **优势**：
  * 符合 POSIX 与 Linux PAM 标准；
  * **完全跨 Shell 兼容**：对 Fish、Bash、Zsh 均原生通用，登录系统后自动全局生效。

#### 方案 B：各 Shell 独立配置方案
如果偏好在个人用户的 Shell 配置中独立维护：

* **Fish**（编辑 `~/.config/fish/config.fish`）：
  ```fish
  set -gx DEBGET_TOKEN "ghp_你的GitHubToken"
  ```
* **Bash**（编辑 `~/.bashrc`）：
  ```bash
  export DEBGET_TOKEN="ghp_你的GitHubToken"
  ```
* **Zsh**（编辑 `~/.zshrc`）：
  ```zsh
  export DEBGET_TOKEN="ghp_你的GitHubToken"
  ```

*(注：如果未执行上述第一步的 sudoers 白名单配置，则每次运行命令必须使用 `sudo -E deb-get update` 显式透传环境变量)*

---

## 特殊软件与版本选择说明

针对上游存在多版本、不同架构变体或非 GitHub 托管的特殊场景，本仓库作了如下区分与处理：

### 1. RustDesk：Wayland 无人值守专用版
* **版本说明**：
  * **标准版 (`rustdesk`)**：已由 `deb-get` 官方仓库（`01-main`）原生收录，执行 `sudo deb-get install rustdesk` 即可。
  * **Wayland 无人值守版 (`rustdesk-unattended-wayland`)**：官方源未提供该变体，由本仓库独立维护，针对现代 Linux 桌面的 Wayland 显示协议定制，支持无人值守远程连接，免去本地手动弹框确认。
* **安装方式**：
  ```bash
  # 安装 Wayland 无人值守版
  sudo deb-get install rustdesk-unattended-wayland
  ```

### 2. EasyTier：核心服务（CLI）与 图形界面（GUI）
* **版本区别**：
  * **`easytier`**：去中心化 Mesh VPN 核心服务（CLI / 无头环境 / 服务器）。上游目前主要提供 GUI 的 `.deb` 包及 CLI 脚本，仓库内已独立预留 CLI 匹配规则，并向下兼容保底。
  * **`easytier-gui`**：包含完整系统托盘与网络拓扑可视化界面的官方桌面端。
* **安装方式**：
  ```bash
  # 桌面端安装 GUI
  sudo deb-get install easytier-gui

  # 核心服务
  sudo deb-get install easytier
  ```

### 3. Anytype：稳定正式版保障
* **特性说明**：Anytype 上游常将带 `-alpha` 标识的预发布构建设为最新发布版本。
* **处理逻辑**：脚本默认过滤排除测试版，确保始终获取正式稳定版；在指定 `DEBGET_CHANNEL=beta` 时可体验 Alpha 测试版本。
* **安装方式**：
  ```bash
  sudo deb-get install anytype
  ```

### 4. Linux QQ 与 微信：腾讯官方动态直链与验签
* **特性说明**：两款软件非 GitHub 托管，由腾讯 CDN 官方维护。其中 Linux QQ 下载直链具有防盗链动态过期签名。
* **处理逻辑**：脚本内已内置自动化请求腾讯官方 RPC 验签接口（`UrlSign/GetSign`），确保拉取到的直链带有有效时效安全参数。
* **安装方式**：
  ```bash
  sudo deb-get install linuxqq
  sudo deb-get install wechat
  ```

---

## 维护规范

1. 新增软件脚本统一放置于 `eallion/packages/` 目录下，文件名为上游 `.deb` 的实际 `Package` 注册名；
2. 脚本无 Shebang，无 `.sh` 后缀，需确保可通过 `bash -n` 语法检测；
3. 将新增的软件名按字母升序追加至 `eallion/manifest` 文件中；
4. 提交代码时会自动触发 GitHub Actions 进行清单格式与脚本语法校验。
