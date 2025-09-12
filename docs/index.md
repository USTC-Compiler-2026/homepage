

# <strong>编译原理与技术</strong>



!!! Info "说明"

    本课程主页为 2025 秋季学期李诚老师班《编译原理与技术》课程主页，包含了理论课与实验课的相关信息。

<hr class="hr-my" data-content="(●′∀｀●) 我是分隔线 (●′∀｀●)">

## <strong>课程简介</strong>

本课程是计算机科学与技术专业的核心课，旨在深入探讨程序设计语言的设计和实现技术，涵盖与程序设计语言理论相关的知识，并培养学生将课程中学到的概念和技术应用于软件设计和开发的能力。内容包括编译器构建的基本原理，包括词法分析、语法分析、语义分析、中间代码生成、代码优化和目标代码生成。同时，本课程结合现代编译技术和趋势，使用 LLVM 和龙芯后端，基于自主设计的教学实验框架“建木”，设计现代化的编译器工具。

## <strong>课程信息</strong>

### <strong>上课时间与地点</strong>

    时间：1~15 周，1(6,7) 和 3(3,4)
    地点：高新校区 GT-B212
### <strong>联系方式</strong>

<strong>教师</strong>

- 主讲教师：李诚（<chengli7@ustc.edu.cn>）

<strong>助教</strong>

  - 王宇航（<wyh2022@mail.ustc.edu.cn>）
  - 艾琮焜（<ack@mail.ustc.edu.cn>）
  - 杨龙成（<ylc221606@mail.ustc.edu.cn>）
  - 孙婧雯（<sunj1ngwen@mail.ustc.edu.cn>）
  - 朱恩松（<eszhu@mail.ustc.edu.cn>）

<strong>其它</strong>

- QQ 群：910418526
<!-- - 录课视频（需校内统一身份认证）：<https://v.ustc.edu.cn/1/2024-1/capture-course/011163.02/detail> -->
<!-- - 希冀实验平台（作业及实验提交）：<http://cscourse.ustc.edu.cn/> -->
<!-- - 实验框架已公开，见参考资料 [<sup>2</sup>](#textbook2) -->


## <strong>公告</strong>
- 2025-09-08：[课程平台介绍](exp_platform_intro/README.md) 已发布
- 2025-09-14：[Lab0](lab0/index.md) 已发布，Deadline：**2025 年 9 月 24 日**
<!--
- 2024-09-18：第一次作业已发布，Deadline：**2024 年 9 月 20 日**
- 2024-09-18：[Lab1](lab1/index.md) 已发布，Deadline：**Phase1 : 2024 年 9 月 28 日 / Phase2 : 2024 年 10 月 7 日**
- 2024-09-25：第二次作业已发布，Deadline：**2024 年 9 月 30 日**
- 2024-10-08：[Lab2](lab2/index.md) 已发布，Deadline：
    - **Phase1 : 2024 年 10 月 18 日**
    - **Phase2 : 2024 年 10 月 27 日**
    - **Phase3 : 2024 年 11 月 2 日**
- 2024-11-03：[Lab3](lab3/index.md) 已发布，Deadline：
    - **Phase1 : 2024 年 11 月 13 日**
    - **Phase2 : 2024 年 11 月 27 日**
- 2024-12-01：[Lab4](lab4/index.md) 已发布，Deadline：
    - **Phase1 : 2024 年 12 月 14 日**
    - **Phase2 : 2024 年 12 月 24 日**
-->
<!--
- 2024-10-11：第三次作业：3.27、3.37、4.3、4.5、4.9（教材 [<sup>1</sup>](#textbook)），Deadline：**2024 年 10 月 18 日 16:00**
- 2024-11-01：[Lab3](lab3/index.md) 已发布，注意分阶段 Deadline
- 2024-11-01：第四次作业已发布，Deadline：**2024 年 11 月 8 日 17:25**
- 2024-11-14：第五次作业：4.13、4.15、5.5、6.5、6.6（教材 [<sup>1</sup>](#textbook)），Deadline：**2024 年 11 月 21 日 09:00**
- 2024-11-27：[Lab4](lab4/index.md) 已发布，Deadline：**2024 年 12 月 18 日**
- 2024-12-08：第六次作业：7.2c、7.5、7.12、8.1e、8.2e、8.6（教材 [<sup>1</sup>](#textbook)），Deadline：**2024 年 12 月 20 日 24:00**
- 2024-12-08：第七次作业：9.1、9.2、9.3、9.15af（教材 [<sup>1</sup>](#textbook)），Deadline：**2024 年 1 月 3 日 24:00**
- 2025-01-14：[建木杯–编译原理创新实验](innovative-lab/index.md) 已发布 -->

## <strong>教学课件</strong>

|     日期     |                          标题                          |                                                 课件                                                 |
|:----------:| :----------------------------------------------------: | :--------------------------------------------------------------------------------------------------: |
| 2025-09-08 |                   编译原理和技术导论                   |                               [Lecture1_Intro](ppt/Lecture1-Intro.pdf)                               |
| 2025-09-10 |                        词法分析                        |                        [Lexical-analysis](ppt/Lecture2-Lexical-analysis.pdf), [Triton_Intro](ppt/triton_intel.pdf)        |
<!--
| 2024-09-11 |               语法分析 - 上下文无关文法                |                            [Parser-part1](ppt/Lecture3-Parser-part1.pdf)                             |
| 2024-09-14 |             语法分析 - 自顶向下 - 递归下降             |                            [Parser-part2](ppt/Lecture4-Parser-part2.pdf)                             |
| 2024-09-16 |            语法分析 - 自顶向下 - LL(1) 文法            |                            [Parser-part3](ppt/Lecture5-Parser-part3.pdf)                             |
| 2024-09-18 |  语法分析 - 自底向上 - 移进规约 & LR 分析器的简单模型  |     [Parser-part4](ppt/Lecture6-Parser-part4.pdf), [Parser-part5](ppt/Lecture7-Parser-part5.pdf)     |
| 2024-09-23 |               语法分析 - 简单的 LR 方法                |                            [Parser-part6](ppt/Lecture8-Parser-part6.pdf)                             |
| 2024-09-25 |       语法分析 - 简单的 LR 方法 & 规范的 LR 方法       |     [Parser-part6](ppt/Lecture8-Parser-part6.pdf), [Parser-part7](ppt/Lecture9-Parser-part7.pdf)     |
| 2024-09-30 |               语法分析 - 规范的 LR 方法                |                            [Parser-part7](ppt/Lecture9-Parser-part7.pdf)                             |
| 2024-10-09 |                      中间代码表示                      |                                      [IR](ppt/Lecture10-IR.pdf)                                      |
| 2024-10-12 |                         习题课                         |     [HW1](ppt/2024-HW1-习题课.pdf), [HW2](ppt/2024-HW2-习题课.pdf), [Lab2](ppt/实验讲解Lab2.pdf)     |
| 2024-10-14 |      语法制导翻译 - 语法制导定义 & S/L 属性的定义      |      [Translation-part1](ppt/Lecture11-Translation-part1.pdf), [Translation-part2](ppt/Lecture12-Translation-part2.pdf)      |
| 2024-10-16 | 语法制导翻译 - 语法制导翻译方案 & L 属性定义的翻译方案 |      [Translation-part2](ppt/Lecture12-Translation-part2.pdf), [Translation-part3](ppt/Lecture13-Translation-part3.pdf)      |
| 2024-10-21 |          语法制导翻译 - L 属性定义的翻译方案           |                             [Translation-part4](ppt/Lecture14-Translation-part4.pdf)                             |
| 2024-10-23 |  中间代码翻译 - 简单语句翻译 & 控制流与布尔表达式翻译  | [IR-Translation-part1](ppt/Lecture15-IR%20Translation-part1.pdf), [IR-Translation-part2](ppt/Lecture16-IR%20Translation-part2.pdf) |
| 2024-10-28 |                    中间代码翻译 - 布尔表达式 & 控制流语句                    |                                                                              [IR-Translation-part3](ppt/Lecture17-IR%20Translation-part3.pdf), [IR-Translation-part4](ppt/Lecture18-IR%20Translation-part4.pdf)                                                                              |
| 2024-10-30 |                    中间代码翻译 - 类型表达式及自动构造                    |                                                                              [IR-Translation-part5](ppt/Lecture19-IR%20Translation-part5.pdf)                                                                              |
| 2024-11-04 |               中间代码翻译 - 类型表达式 & 符号表与声明语句                |                                        [IR-Translation-part6](ppt/Lecture20-IR%20Translation-part6.pdf)                                        |
| 2024-11-06 |              中间代码翻译 - 数组寻址 & 运行时管理 - 存储组织              |                                       [IR-Translation-part7](ppt/Lecture21-IR%20Translation-part7.pdf), [Runtime-part1](ppt/Lecture22-Runtime-part1.pdf)                                        |
| 2024-11-11 |           运行时管理 - 栈式分配          |                                       [Runtime-part2](ppt/Lecture23-Runtime-part2.pdf), [Runtime-part3](ppt/Lecture24-Runtime-part3.pdf)                                        |
| 2024-11-13 |                     代码生成 - 简单机器模型                     |                                                                              [CodeGen-part1](ppt/Lecture25-CodeGen-part1.pdf)                                                                              |
| 2024-11-18 | 机器无关代码优化 - 常见优化方法 | [IR Opt-part1](ppt/Lecture26-IR%20Opt-part1.pdf)|
| 2024-11-20 | 机器无关代码优化 - 数据流与到达定值分析  | [IR Opt-part2](ppt/Lecture27-IR%20Opt-part2.pdf)|
| 2024-11-25 | 机器无关代码优化 - 可用表达式分析 & 活跃变量分析 & 基本块内优化             |                                       [IR Opt-part3](ppt/Lecture28-IR%20Opt-part3.pdf), [IR Opt-part4](ppt/Lecture29-IR%20Opt-part4.pdf), [IR Opt-part5](ppt/Lecture30-IR%20Opt-part5.pdf)                                       |
| 2024-11-27 | 流图中的循环 & 寄存器分配              |                                                                              [Loop](ppt/Lecture31-Loop-part1.pdf), [Register](ppt/Lecture32-Register.pdf)                                |

-->
<!--| 2024-11-29 |                          面向目标机器的代码优化                           |                                                                              [part1](https://rec.ustc.edu.cn/share/d6169380-9045-11ee-8a37-87201671ab8d)                                                                              |
| 2024-12-04 |                               guest lecture                               |                                                                                                               无 slides                                                                                                               |
| 2024-12-06 |                               guest lecture                               |                                                                                                               无 slides                                                                                                               |
| 2024-12-13 |                                  复习课                                   |                                                                             [slides](https://rec.ustc.edu.cn/share/3a4ffcf0-995a-11ee-9fdc-a7ee4ffd604e)                                                                              | -->

## <strong>参考资料</strong>

### <strong>教材和参考书</strong>

- <div id='textbook'></div>[1] 陈意云、张昱，编译原理（第 3 版），高等教育出版社，2014
- <div id='textbook2'></div>[2] 李诚、徐伟，现代编译器设计与实现（实验讲义版本，高等教育出版社待出版，2024）。[教材资源及实验框架](https://ustc-compiler-principles.github.io/textbook/)
- A. V. Aho, M. S. Lam, R. Sethi, and J. D. Ullman 著，赵建华等译，编译原理，机械工业出版社，2017

### <strong>其他资料</strong>

- Stanford 课程主页：<http://web.stanford.edu/class/cs143/>
- MIT 课程主页：<http://6.035.scripts.mit.edu/fa18/>



## <strong>如何使用本文档？</strong>

### <strong>关于提示框</strong>

为了便于标识，我们在文档中提供了若干提示框。它们有着不同的形式与作用。例如：

!!! Note "一般的提示框长这样"

    里面是这样的文字。


??? Question "有的提示框默认是折叠的，需要点击才能展开"

    然后看到里面的内容。


???+ Success "有的提示框则是默认展开的"

    如果觉得碍事可以点击折叠。


!!! Example "大家可以在后续的文档中接触到它们。"

### <strong>关于搜索</strong>

你可以在右上角搜索框中输入想要查询的关键词，文档会给出对应的查询结果与跳转链接。

???+ Note "Tips"

    如果在阅读文档时发现有自己不清楚的知识点，可以先用搜索框查询其是否在先前的文档中出现过。

<!-- <hr class="hr-my" data-content="(●′∀｀●) 我是分隔线 (●′∀｀●)">


???+ Bug "评论系统"

    由于主页上的评论系统映射可能出现问题，以防万一，我们在这里进行一些补充。

    除了上面介绍的内容，本学期的实验文档我们还额外添加了评论系统。大家可以在各个界面下方找到类似的评论栏，登录自己的 GitHub 账号即可发表相应的评论。

    <strong>怎么使用？</strong>

    评论系统的输入采用 Markdown 格式。如果你之前没有用过 Markdown，可以简单地将其当做普通文本（txt）格式，直接输入文字并点击评论即可。如果你对 Markdown 语法有所了解，可以使用 **加粗**、 *斜体* 、句内的 `code block` 等特殊格式，以及相应的标题结构。

    除了留下自己的疑问，大家也可以解答其他同学的疑问。这是一个相互交流、相互合作的平台。我们鼓励合理范围内的讨论与思考~

    <strong>其他方式</strong>

    评论系统实际上是抓取了 [这个仓库](https://github.com/USTC-Compiler-2024/Compiler-Comments) 下讨论区的内容，所有的评论也会发布在这里。大家可以访问上面的仓库进行阅读。

    如果你没有或者无法登录 GitHub 账号也没关系。除了文档下方的评论系统，在课程群中大家也可以提出自己的问题，我们将统一进行解答。

    欢迎大家在评论系统里畅所欲言！ -->