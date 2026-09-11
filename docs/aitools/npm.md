# npm 环境配置

npm（Node Package Manager）是 Node.js 的包管理器，用于下载、安装和更新 JavaScript/Node.js 软件包。本组文档通过 npx 启动 DeepSeek Harness，并使用 npm 安装 Claude Code、Codex CLI 和 OpenCode，因此需要先准备 Node.js 与 npm 环境。

本文介绍 Node.js 和 npm 的安装，以及使用 npm 安装 AI 工具的方法。DeepSeek Harness、Claude Code、Codex 和 OpenCode 的具体说明分别见[DeepSeek Harness](deepseek_harness.md)、[Claude Code](claude.md)、[Codex](codex.md)和[OpenCode](opencode.md)。

## npm 包管理器的好处

- **跨平台**：Windows 和 Linux 使用相同的安装命令。
- **自动处理依赖**：无需手动下载和配置相关组件。
- **方便管理**：支持版本更新和镜像源配置。

例如，DeepSeek Harness 可使用 npx 直接启动；Claude Code、Codex 和 OpenCode 可使用 npm 全局安装：

```bash
npx @deepseek-ai/dsh web
npm install -g @anthropic-ai/claude-code
npm install -g @openai/codex
npm install -g opencode-ai
```

!!! note "Linux 全局安装权限"

    使用 apt 或 NodeSource 安装 Node.js 时，`npm install -g` 可能提示 `EACCES` 或权限不足。此时在 npm 全局安装或更新命令前加 `sudo`，例如 `sudo npm install -g @openai/codex`。Windows 不需要 `sudo`；使用 `npx` 启动 DeepSeek Harness 通常也不需要。

## 安装 Node.js LTS

### Windows

在普通 PowerShell 或 Windows Terminal 中执行：

```powershell
winget install -e --id OpenJS.NodeJS.LTS
```

安装完成后，关闭并重新打开终端，检查 Node.js 和 npm 是否安装成功：

```powershell
node --version
npm --version
```

如果系统没有 `winget`，可以从 [Node.js 官方下载页](https://nodejs.org/en/download)下载并安装标记为 **LTS** 的 Windows Installer。安装时请保留“Add to PATH”选项。

### Linux（Ubuntu / Debian）

以下命令以 Bash 为例，普通用户需要具有 `sudo` 权限：

```bash
sudo apt-get update
sudo apt-get install -y curl ca-certificates
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs
```

安装完成后检查版本：

```bash
node --version
npm --version
```

## 可选：使用 npm 镜像站

如果 npm 官方源下载较慢，可以临时使用镜像站。镜像站只负责下载 npm 包，不限制操作系统。

临时使用镜像站：

```bash
npm install -g <package-name> --registry=https://registry.npmmirror.com
```

!!! warning "镜像源的作用范围"

    npm 镜像只影响安装和更新时的包下载。工具安装完成后，登录和运行仍需要能够访问所选模型提供商的服务。
