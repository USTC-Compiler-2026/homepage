# Lab4

经过前序实验，同学们已经基本完成了一个贯穿从前端到后端的简单编译器，祝贺大家！然而，该编译器生成的代码只能保证语义正确、能在目标机器上运行。接下来，我们将让大家体验如何通过增加优化 pass 让生成的代码快起来。正如课上所讲，优化的方法有很多，由于时间关系，我们不能一一尝试。因此，我们为大家准备了 Mem2Reg (访存优化) 与 LICM (循环优化实验)，在实验中，同学们将实现这两个优化 pass。完成代码后，同学们可以在测试样例中，看到优化前后的性能差距。


## 同步实验仓库

在进行实验之前，首先拉取[实验仓库](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2024ustc-jianmu-compiler)的最新代码，具体步骤可以参考 [Lab2 中的指导](../lab2/index.md#实验要求)。

本次实验仓库更新的内容如下，每个阶段的文件将在对应文档详细说明：

```
.
├── ...
├── include
│   ├── ...	
│   └── passes
│       ├── DeadCode.hpp        # 死代码删除
│       ├── Dominators.hpp      # 支配树分析（需要阅读，根据需要修改）
│       ├── FuncInfo.hpp        # 纯函数分析
│       ├── LICM.hpp            # 循环不变量外提（需要阅读，根据需要修改）
│       ├── LoopDetection.hpp   # 循环分析（需要阅读，根据需要修改）
│       ├── Mem2Reg.hpp         # Mem2Reg 分析（需要阅读，根据需要修改）
│       └── PassManager.hpp     # PassManager：管理 pass 的运行
├── src
│   ├── ...
│   └── codegen
│       ├── ...
│       └── CodeGen.cpp         <-- 需要实现对phi指令的处理
│   └── passes
│       ├── ...
│       ├── Dominators.cpp      <-- 支配树分析实现，需要补全
│       ├── LICM.cpp            <-- 循环不变量外提实现，需要补全
│       ├── LoopDetection.cpp   <-- 循环分析实现，需要补全
│       └── Mem2Reg.cpp         <-- Mem2Reg 实现，需要补全
└── tests
    ├── ...
    └── 4-opt                   # 本地测试
```

## 实验内容

### 阶段一 Mem2Reg

阅读章节 [Mem2Reg](./Mem2Reg.md) ，完成 Dominators 支配树分析 与 Mem2Reg 访存优化 两个Pass。

!!! warning "Deadline"

    **2024 年 12 月 14 日 23:59**

### 阶段二 LICM

阅读章节[循环相关介绍](./loop.md) 和 [LICM](./LICM.md)，完成 LoopDetection 循环分析 与 LICM 循环不变量外提 两个Pass。

!!! warning "Deadline"

    **2024 年 12 月 24 日 23:59**

## 提交方式

- 在希冀平台提交实验仓库的 URL

  > 在提交之前，请确保你 fork 得到的远程仓库与本地同步：`git push origin master`
  
- 在希冀平台提交实验报告（实现方法、正确性验证、性能验证等）
