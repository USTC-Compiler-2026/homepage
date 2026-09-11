# OpenCode

OpenCode 是开源的终端 AI 编程工具，可以在项目目录中阅读和修改代码、执行命令，并通过不同模型提供商的 API 完成任务。

## 安装

请先完成 [npm 环境配置](npm.md)。OpenCode 官方建议 Windows 用户优先在 WSL 中使用；也可直接在 Windows PowerShell 中通过 npm 安装。

### Windows（PowerShell）

```powershell
npm install -g opencode-ai
```

可用镜像站：

```powershell
npm install -g opencode-ai --registry=https://registry.npmmirror.com
```

### Linux（Ubuntu / Debian，Bash）

```bash
npm install -g opencode-ai
```

!!! note "Linux 全局安装权限"

    若命令提示 `EACCES` 或权限不足，在 npm 全局安装或更新命令前加 `sudo`，例如 `sudo npm install -g opencode-ai`。

!!! note "其他安装方式"

    OpenCode 也提供 Windows 的 Scoop、Chocolatey 安装方式，以及 macOS/Linux 的安装脚本和 Homebrew 安装方式。课程环境推荐使用 npm，便于与其他工具保持一致。

## 验证安装

```bash
opencode --version
```

如果系统提示找不到 `opencode` 命令，请关闭并重新打开终端，使新的 PATH 配置生效。

## 首次启动与模型配置

进入代码项目目录后启动 OpenCode：

```bash
cd your-project
opencode
```

在交互界面中输入以下命令配置模型提供商：

```text
/connect
```

按照界面提示选择提供商并填写对应的 API key。OpenCode 支持多个模型提供商；首次使用时也可以选择 OpenCode Zen 提供的模型。

若使用 DeepSeek 官方的 [V4.1 Flash](https://www.deepseek.com/news/deepseek-v4-1-flash/) API，可创建或修改全局配置文件 `$HOME/.config/opencode/opencode.jsonc`：

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "deepseek/deepseek-flash",
  "small_model": "deepseek/deepseek-flash",
  "provider": {
    "deepseek": {
      "options": {
        "apiKey": "你的 DeepSeek API Key"
      }
    }
  }
}
```

## 更新

使用 npm 安装时，可以执行：

```bash
npm install -g opencode-ai@latest
```

使用镜像站更新：

```bash
npm install -g opencode-ai@latest --registry=https://registry.npmmirror.com
```

## 官方资料

- [OpenCode 官方文档](https://opencode.ai/docs)
- [OpenCode 官方仓库](https://github.com/anomalyco/opencode)
- [OpenCode 模型提供商配置](https://opencode.ai/docs/providers)
