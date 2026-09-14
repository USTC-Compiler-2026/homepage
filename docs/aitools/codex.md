# Codex

Codex CLI 是 OpenAI 提供的终端编程工具，可以在项目目录中阅读代码、修改文件并运行命令。Windows 和 Linux 都可以通过 npm 安装。

## 安装

请先完成 [npm 环境配置](npm.md)。建议使用 Node.js LTS。

### Windows（PowerShell）

```powershell
npm install -g @openai/codex
```

可用镜像站：

```powershell
npm install -g @openai/codex --registry=https://registry.npmmirror.com
```

### Linux（Ubuntu / Debian，Bash）

```bash
# 镜像站用法和 Windows 系统相同
npm install -g @openai/codex
```

!!! note "Linux 全局安装权限"

    若命令提示 `EACCES` 或权限不足，在 npm 全局安装或更新命令前加 `sudo`，例如 `sudo npm install -g @openai/codex`。

## 验证安装

```bash
codex --version
```

如果系统提示找不到 `codex` 命令，请关闭并重新打开终端，使新的 PATH 配置生效。

## 首次登录

进入代码项目目录后启动 Codex：

```bash
cd your-project
codex
```

首次启动时选择 **Sign in with ChatGPT**，然后按照浏览器中的提示完成登录。

也可以使用 API key。下面的设置只对当前终端会话生效：

=== "Windows PowerShell"

    ```powershell
    $env:OPENAI_API_KEY = "你的 OpenAI API Key"
    codex
    ```

=== "Linux Bash"

    ```bash
    export OPENAI_API_KEY='你的 OpenAI API Key'
    codex
    ```

!!! note "关于登录凭证"

    在 $HOME/.codex 目录下，有一个 auth.json 和 config.toml 文件，其中 auth.json 是登录凭证，将此文件复制到其他机器上即可实现登录；config.toml 文件是配置文件。

## 更新

使用 npm 安装时，可以执行：

```bash
npm install -g @openai/codex@latest 
```

Windows PowerShell 和 Linux Bash 使用相同的 npm 命令。

## 使用桌面应用（可视化界面）

OpenAI 提供的可视化桌面端是 ChatGPT 桌面应用，登录后可在应用中选择 Codex 使用。本节的安装方式不依赖 Node.js 或 npm。

### macOS

从 [ChatGPT 官方下载页](https://chatgpt.com/download/) 下载 macOS 安装包。打开下载的 `.dmg` 文件后，将 ChatGPT 拖入“应用程序”文件夹；随后打开 ChatGPT，使用 ChatGPT 账户登录，并在应用中选择 Codex。

### Windows

从 [ChatGPT 官方下载页](https://chatgpt.com/download/) 下载并安装，也可在 `Microsoft Store` 安装。

安装后打开 ChatGPT 桌面应用，使用 ChatGPT 账户登录。

### Linux（Ubuntu / Debian）

Linux 桌面应用目前为预览版。请在 [ChatGPT 官方下载页](https://chatgpt.com/download/) 下载与处理器架构匹配的 `.deb` 安装包；可用以下命令查看架构：

```bash
uname -m
```

下载完成后，在安装包所在目录执行：

```bash
sudo apt install ./chatgpt_amd64.deb
```

ARM64 设备请将文件名替换为 `chatgpt_arm64.deb`。安装后从应用程序菜单打开 ChatGPT，登录后选择 Codex。当前官方支持 Ubuntu 24.04/26.04 LTS 和 Debian 13 的桌面环境。

## 官方资料

- [OpenAI Codex CLI 文档](https://developers.openai.com/codex/cli/)
- [Codex 官方仓库](https://github.com/openai/codex)
