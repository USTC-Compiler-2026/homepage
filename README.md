# 中国科学技术大学编译原理与技术课程

## README

本项目为中国科学技术大学 2025 秋季学期李诚班《编译原理与技术》课程在线文档，使用 Mkdocs 搭建并在 GitHub 上托管项目。

> 这里是[ MkDocs 文档](https://www.mkdocs.org/)，你可以访问官方文档以查询更多内容。

本文档主要介绍如何本地编写、预览与发布课程文档。有问题请在群里交流哦 ~


## 本地搭建

如果你想：

- 编写或更新实验文档，并将其发布在 github.io 上；
- 阅读最新的实验文档；
- github.io 出现问题无法访问；
- 其他可能的问题等。

你可以参考下面的内容进行本地搭建。

### 获取项目内容

首先将本仓库的内容从 GitHub clone 到本地。在你选定的本地路径下运行

```
git clone https://github.com/USTC-Compiler-2025/HomePage.git
```

即可。clone 完成后进入 HomePage 目录，该目录即为我们的项目根目录。

### 创建本地环境

你可以使用 Anaconda 等创建虚拟环境（ **推荐！** ），也可以在自己的环境中进行配置。我选择的 Python 版本是 3.8，其他 Python 版本理论上也可以正常工作。

你可以直接使用

```
pip install -r ./requirement.txt
```

安装我们所需要的包。如果你想自行配置，可以使用下面的指令安装 Mkdocs 包：

```
pip install mkdocs
```

以及使用下面的指令安装必须的插件资源：

```
pip install mkdocs-material
pip install mkdocs-glightbox
pip install mkdocs-material-extensions
pip install mkdocs-git-revision-date-localized-plugin
pip install mkdocs-git-authors-plugin
```


安装完成后，使用下面的命令搭建本地文档

```
mkdocs serve
```

你可以在 http://127.0.0.1:8000/ 访问本地实验文档。

## 内容编写

### 关于文件目录

Mkdocs 采用 Markdown 作为文本编写语言，因此我们可以直接编辑 `docs/` 目录下的 .md 文件以实现对网页内容的更改。例如：我们的实验文档目录可以概括如下：

```
HomePage/
├── docs/       <-- 这个是文档目录
│   ├── figs/
│   │   └── ......
│   ├── lab1/       <-- lab1 文件夹
│   │   ├── figs/
│   │   │   └── ......
│   │   ├── intro.md    <-- lab1 的文档源文件
│   │   ├── tasks.md
│   │   ├── FAQ.md
│   │   └── ......
│   ├── lab2/       <-- 以此类推
│   │   ├── figs/
│   │   │   └── ......
│   │   ├── intro.md
│   │   ├── tasks.md
│   │   ├── FAQ.md
│   │   └── ......
│   └── ......
├── overrides/       <-- 用于存放 Mkdocs 可能用到的资源文件
│   └── ......
├── mkdocs.yml       <-- Mkdocs 重要的约束文件
└── ......
```

网页的层次则统一在 `mkdocs.yml` 文件中进行约束。例如：

```
nav:
  - 首页: index.md
  - 课程平台介绍: exp_platform_intro/README.md
  - 公共文档:
    - Cminusf 语法语义介绍: common/cminusf.md
    - Light IR 简介: common/LightIR.md
    - C++ 介绍: common/simple_cpp.md
    - Logging 工具使用: common/logging.md
    - 龙芯汇编介绍: common/asm_intro.md
  - Lab0:
    - Lab0 简介: lab0/index.md
    - Linux 环境配置: lab0/linux.md
    - LLVM 等软件环境配置: lab0/software.md
    - Git 的使用: lab0/git.md
  - Lab1:
    - Lab1 简介: lab1/index.md
    - 正则表达式: lab1/正则表达式.md
    - Flex: lab1/Flex.md
    - Bison: lab1/Bison.md
    - 实验细节与要求: lab1/实验细节与要求.md
```

这里实际上就建立了网页目录与实际目录之间的映射关系。新建其他文件夹或文件时，只需要在 nav 下按照格式进行添加即可。

### 支持的风格

除了加粗 `**`、斜体 `*`、代码块 `` ` `` 等原生 Markdown 语法外，我们还使用了一系列插件以实现更为丰富的语法功能。例如：

**提示框**

格式为：

```
!!! Type "标题"

    内容在这里。这是不可折叠提示框
```

```
??? Type "标题"

    内容在这里。这是默认折叠的提示框
```

```
???+ Type "标题"

    内容在这里。这是默认展开的提示框
```

可选的 Type 包括：

- new
- settings
- note
- abstract
- info
- tip
- success
- question
- warning
- failure
- danger
- bug
- example
- quote


**Latex 公式**

你可以使用句内公式 `$$` 以及行间公式。注意：不支持矩阵等特殊环境，且行间公式需要前后空行。例如：

```
正文

$$
y=ax+b
$$

$$
y=ax+b
$$

正文
```

**代码块**

支持编号、标题等。格式为

```
    ``` language linenums="1" title="标题"
```
```

编号和标题可以选择性删除。

**插入图片**

可以使用下面的方式

```
<figure markdown>
![Image title](路径){ width="大小" }
<figcaption>图片名称，可以没有这行</figcaption>
</figure>
```

如果想要并排图片，可以使用表格语法。例如

```
<table><tr>
<td><figure>
<img src="../figs/log_wire_on.png">
<figcaption>输入为 1</figcaption>
</figure></td>
<td><figure>
<img src="../figs/log_wire_off.png">
<figcaption>输入为 0</figcaption>
</figure></td>
</tr></table>

```

实现了两张图片并排。


## 页面发布

Mkdocs 支持自动将本地内容生成网页，并发布至 Github 仓库中。我们只需要在本地输入

```
mkdocs gh-deploy
```

即可实现自动发布。你可以加上 `-b` 参数以指定发布的分支。例如：

```
mkdocs gh-deploy -b cur_page
```

会将生成的网页推送到 `cur_page` 分支上。

如果不指定 `-b` 参数，发布会固定推送到 `gh-pages` 分支中，因此**请不要自己修改 `gh-pages` 分支上的内容**！

**特别提示：在使用 `mkdocs gh-deploy` 前，请确保你的内容不会与其他人的内容产生冲突！否则 `gh-pages` 分支也会出现冲突！** 我们建议大家先在 `main` 分支上处理完冲突后再 `mkdocs gh-deploy`。 


## 分支约定

目前的分支包括

- `main`：主分支。包含全部的开发内容
- `gh-pages`：网页分支。这是部署在 `[github.io](https://ustc-compiler-2025.github.io/HomePage/)` 上的界面，包含不完全开发的实验教程。**每次实验的练习部分在实验发布时才放出。**


