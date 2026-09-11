# Claude Code

Claude Code 是 Anthropic 提供的终端编程工具，可以在项目目录中协助阅读代码、修改文件和运行命令。Windows 和 Linux 都可以通过 npm 安装。

## 安装

请先完成 [npm 环境配置](npm.md)。当前 Claude Code 的 npm 包要求 Node.js 22 或更高版本，建议直接安装 Node.js LTS。

### Windows（PowerShell）

```powershell
npm install -g @anthropic-ai/claude-code
```

可用镜像站：

```powershell
npm install -g @anthropic-ai/claude-code --registry=https://registry.npmmirror.com
```

如果已经将 npm 镜像设置为默认源，也可以省略 `--registry` 参数。

### Linux（Ubuntu / Debian，Bash）

```bash
npm install -g @anthropic-ai/claude-code
```

## 验证安装

```bash
claude --version
claude doctor
```

如果系统提示找不到 `claude` 命令，请关闭并重新打开终端，使新的 PATH 配置生效。

## 首次登录

进入代码项目目录后启动 Claude Code：

```bash
cd your-project
claude
```

首次启动时按照浏览器提示完成登录。可用的登录方式和账户要求以 Claude Code 当前官方说明为准。

也可以使用 API key。下面的设置只对当前终端会话生效：

=== "PowerShell"

    ```powershell
    $env:ANTHROPIC_API_KEY = "你的 Anthropic API Key"
    claude
    ```

=== "Bash"

    ```bash
    export ANTHROPIC_API_KEY='你的 Anthropic API Key'
    claude
    ```

## 其他 api 接入

Claude Code 支持接入 DeepSeek 等提供 Anthropic 兼容 API 的服务。以下以 [DeepSeek V4.1 Flash](https://www.deepseek.com/news/deepseek-v4-1-flash/) 的 API 写法为例，修改 `$HOME/.claude/settings.json`：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "你的 DeepSeek API Key",
    "ANTHROPIC_MODEL": "deepseek-flash",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek-flash",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-flash",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-flash",
    "CLAUDE_CODE_SUBAGENT_MODEL": "deepseek-flash",
    "CLAUDE_CODE_EFFORT_LEVEL": "max"
  }
}
```

`$HOME` 表示当前用户的主目录：Linux 中通常为 `/home/<用户名>`；PowerShell 中通常为 `C:\Users\<用户名>`。修改完成后重新启动 Claude Code，使配置生效。

## 更新

使用 npm 安装时，可以执行：

```bash
# 也可用镜像站
npm install -g @anthropic-ai/claude-code@latest
```

Windows PowerShell 和 Linux Bash 使用相同的 npm 命令。

## 官方资料

- [Claude Code 安装与认证](https://code.claude.com/docs/en/getting-started)
