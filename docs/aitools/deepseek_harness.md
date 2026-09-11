# DeepSeek Harness (dsh)

DeepSeek Harness（简称 `dsh`）是 DeepSeek 开源的 AI Agent 运行环境。它通过 Web UI 在项目目录中协助阅读和修改代码、执行命令、维护任务计划。

## 前置条件

请先完成 [npm 环境配置](npm.md)，并确认 Node.js 可用：

```bash
node --version
npx --version
```

## 启动 dsh

进入需要处理的项目目录，执行以下命令：

```bash
cd your-project
npx @deepseek-ai/dsh web
```

首次运行时，`npx` 会下载并启动 dsh。启动成功后，终端会输出本地访问地址；默认地址为 <http://127.0.0.1:3080>，并会尝试在浏览器中打开它。

### 使用 npm 镜像站

默认情况下，`npx` 从 npm 官方源下载 dsh。如果下载较慢，可以临时使用镜像站：

```bash
npx --registry=https://registry.npmmirror.com @deepseek-ai/dsh web
```

也可以按照 [npm 环境配置](npm.md#可选使用-npm-镜像站) 将镜像站设为默认源，此后仍使用普通命令即可：

```bash
npx @deepseek-ai/dsh web
```

若只希望启动服务、不自动打开浏览器，可以使用：

```bash
npx @deepseek-ai/dsh web --no-open
```

## 首次配置

1. 在浏览器中打开终端输出的 dsh 地址。
2. 打开 **Settings → Models**，填写 DeepSeek API key 并保存（可在 DeepSeek 官网购买）。
3. 选择 **Choose workspace**，添加并选中当前项目目录。
4. 创建会话后输入任务，例如：

   ```text
   阅读这个仓库，说明各目录的作用，并找出当前未通过的测试。
   ```

配置完成后，dsh 可以立即使用模型，无需重新启动服务。执行过程中请阅读其提出的命令和文件修改请求，再决定是否批准。

## 常用操作

### 指定端口

默认端口被占用时，指定其他端口：

```bash
npx @deepseek-ai/dsh web --port 8080
```

然后在浏览器中打开终端输出的完整地址。

### 查看配置

当页面无法启动或插件配置出现问题时，可输出当前配置进行排查：

```bash
npx @deepseek-ai/dsh web --dump-config
```

### 从源码运行

需要修改 dsh 本身或参与开发时，使用源码方式运行：

```bash
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
pnpm install
pnpm run build
pnpm dsh web
```

此方式额外需要安装 `git` 和 `pnpm`。普通使用不需要克隆源码。

## 官方资料

- [DeepSeek Harness 官方仓库](https://github.com/deepseek-ai/deepseek-harness)
- [DeepSeek Harness 官方文档](https://deepseek-harness.github.io/deepseek-harness/)
- [Web UI 使用指南](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/index.md)
