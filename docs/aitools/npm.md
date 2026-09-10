# npm 环境配置

npm（Node Package Manager）是 Node.js 的包管理器，用于下载、安装和更新 JavaScript/Node.js 软件包。本组文档使用 npm 安装 Codex CLI、Claude Code 和 OpenCode，因此需要先准备 Node.js 与 npm 环境。

本文介绍 Node.js 和 npm 的安装，以及使用 npm 安装 AI 工具的方法。Codex、Claude Code 和 OpenCode 的具体说明分别见[Codex](codex.md)、[Claude Code](claude.md)和[OpenCode](opencode.md)。

## npm 包管理器的好处

- **跨平台**：Windows 和 Linux 使用相同的安装命令。
- **自动处理依赖**：无需手动下载和配置相关组件。
- **方便管理**：支持版本更新和镜像源配置。

例如，三个工具都可以直接使用 npm 安装：

```bash
npm install -g @openai/codex
npm install -g @anthropic-ai/claude-code
npm install -g opencode-ai
```

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

以下命令以 Bash 为例，需要具有管理员权限：

```bash
apt-get update
apt-get install -y curl ca-certificates
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt-get install -y nodejs
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

也可以将镜像站设置为默认源：

```bash
npm config set registry https://registry.npmmirror.com
npm config get registry
```

恢复 npm 官方源：

```bash
npm config set registry https://registry.npmjs.org
```

!!! warning "镜像源的作用范围"

    npm 镜像只影响安装和更新时的包下载。工具安装完成后，登录和运行仍需要能够访问 OpenAI 或 Anthropic 的服务。
