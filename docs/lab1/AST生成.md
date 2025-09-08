# AST 生成

## `syntax_tree` 向 AST 的转换

现在我们开始实现 由 `syntax_tree` 向 `AST` 的转换。

`AST`中所有结点类都在 `include/common/ast.hpp` 中有所定义，并在 `src/common/ast.cpp` 补充了相关实现。

关于 `AST` 的具体知识请参考 [AST](./AST.md)

### 实验内容

本部分需要各位同学在 `src/common/ast.cpp` 中，根据 Cminusf 的文法规则和 AST 结点类型完成由 `syntax_tree` 向 `AST` 的转化。

在 `src/common/ast.cpp` 中，你需要补充部分 `TODO` 部分的代码逻辑，能够通过测试脚本生成恰当的 `AST`。例如输入

```c++
int main(void) { return 0; }
```

可以得到如下 `AST`：

```
program
--fun-declaration: main
----compound-stmt
------return-stmt
--------simple-expression
----------additive-expression
------------term
--------------num (int): 0
```

**这一部分必须严格遵守我们给出的语法，输出必须与标准程序输出完全一致。**

这部分一共有 5 个 `TODO` 待实现，请按照 `TODO` 的顺序实现，并仿照其他结点的实现逻辑，填充代码。

- `TODO1`：`ASTFunDeclaration`
- `TODO2`：`ASTCompoundStmt`
- `TODO3`：`ASTNum`
- `TODO4`：`ASTAdditiveExpression`
- `TODO5`：`ASTSelectionStmt`

## 实验要求

### 仓库目录结构

与本次实验有关的文件如下。

```
.
├── build
├── CMakeLists.txt
├── include
│   └── common
│       ├── ast.hpp	# ast所在头文件
│       ├── logging.hpp
│       └── syntax_tree.h # syntax_tree所在头文件
├── README.md
├── src
│   ├── CMakeLists.txt
│   ├── cminusfc
│   │   ├── CMakeLists.txt
│   │   └── main.cpp # 主函数入口
│   ├── common
│   │   ├── ast.cpp # 你需要修改本文件
│   │   ├── CMakeLists.txt
│   │   ├── logging.cpp
│   │   └── syntax_tree.c # syntax_tree所在头文件
│   ├── logging
│   └── parser
└── tests
    ├── 1-parser
    │   ├── cleanup.sh # 清空生成结果的脚本
    │   ├── eval_phase1.sh
    │   ├── eval_phase2.sh	# 测试本阶段所用的脚本
    │   ├── input # 针对 lab1 的测试样例
    │   ├── output_standard
    │   ├── output_standard_ast # 标准的ast输出
    └── testcases_general # 整个课程所用到的测试样例
```

### 编译、运行和评测

首先将你的实验仓库克隆的本地虚拟机中。要编译和测试 `AST`，请按照以下步骤在本地虚拟机中进行操作：

#### 编译

```c++
$ cd 2024ustc-jianmu-compiler
$ mkdir build
$ cd build
# 使用 cmake 生成 makefile 等文件
$ cmake ..
# 使用 make 进行编译, 这里也可以使用 sudo make install 将其加到环境变量中
$ make
```

如果构建成功，你会在 `build` 文件夹下找到 `cminusfc` 可执行文件，用于生成 `AST`。

```
$ ls cminusfc
cminusfc

$ ./cminusfc
./cminusfc: no input file
```

> 这里需要给 `cminusfc` 可执行文件 提供测试样例，也即 cminus 文件。

#### 运行

我们在 `tests/testcases_general` 文件夹中准备了一些通用案例。

```c++
# 返回 2024ustc-jianmu-compiler 的根目录

# 运行 cminusfc，进行AST生成
$ ./build/cminusfc ./tests/testcases_general/1-return.cminus
program
--fun-declaration: main
----compound-stmt
------return-stmt: void
```

#### 测试

测试样例分为两个部分，分别是 `tests/testcases_general` 和 lab1 限定的 `tests/1-parser`。

我们重点使用 `tests/1-parser` 考察 AST 的正确性。其结构如下：

```
.
|-- input # 输入目录，包含 xxx.cminus 文件
|   |-- easy
|   |-- hard
|   `-- normal
|-- output_standard_ast # 助教标准输出目录，包含 xxx.ast 文件
|   |-- easy
|   |-- normal
|   |-- hard
|   `-- testcases_general
|-- output_student_ast # 学生输出目录，测试过程中产生 xxx.ast 文件
|   |-- easy
|   |-- hard
|   |-- normal
|   `-- testcases_general
|-- eval_phase2.sh # 测试脚本
`-- cleanup.sh
```

我们使用 `diff` 命令进行结果比较

```c++
$ cd 2024ustc-jianmu-compiler
$ export PATH="$(realpath ./build):$PATH"
$ cd tests/1-parser
$ cminusfc input/normal/local-decl.cminus > output_student_ast/normal/local-decl.ast
$ diff output_student_ast/normal/local-decl.ast output_standard_ast/normal/local-decl.ast
[输出为空，代表没有区别，该测试通过]
```

我们提供了 `eval_phase2.sh` 脚本进行快速批量测试。

该脚本有两种使用方式

- 使用 `-all` 参数一键测试 4 个测试集的所有样例。并会在最后给出正确的样例个数，如下。

  ```
  innerpeace@innerpeace:~/stl_debug/tests/1-parser$ ./eval_phase2.sh -all
  [info] Found 2 valid files in /home/innerpeace/stl_debug/tests/1-parser/input/easy
  [info] Skipping FAIL_comment2.cminus (starts with FAIL)
  [info] Skipping FAIL_comment.cminus (starts with FAIL)
  [info] Skipping FAIL_function.cminus (starts with FAIL)
  [info] Skipping FAIL_id.cminus (starts with FAIL)
  [info] Analyzing id.cminus
  [info] Comparing id.cminus...
  [info] id.cminus is correct!
  [info] Score for /home/innerpeace/stl_debug/tests/1-parser/input/easy: 2/2
  [info] Found 5 valid files in /home/innerpeace/stl_debug/tests/1-parser/input/normal
  ...
  [info] local-decl.cminus is correct!
  [info] Analyzing skip_spaces.cminus
  [info] Comparing skip_spaces.cminus...
  [info] skip_spaces.cminus is correct!
  [info] Score for /home/innerpeace/stl_debug/tests/1-parser/input/normal: 5/5
  [info] Found 6 valid files in /home/innerpeace/stl_debug/tests/1-parser/input/hard
  ...
  [info] Analyzing You_Should_Pass.cminus
  [info] Comparing You_Should_Pass.cminus...
  [info] You_Should_Pass.cminus is correct!
  [info] Score for /home/innerpeace/stl_debug/tests/1-parser/input/hard: 6/6
  [info] Found 21 valid files in /home/innerpeace/stl_debug/tests/1-parser/../testcases_general
  ...
  [info] Analyzing 9-assign_cast.cminus
  [info] Comparing 9-assign_cast.cminus...
  [info] 9-assign_cast.cminus is correct!
  [info] Score for /home/innerpeace/stl_debug/tests/1-parser/../testcases_general: 21/21
  [info] Total score for all testcases: 34/34
  ```

  一共有 34 个测试样例（除去 phase1 中的 6 个无法生成 `syntax_tree` 的样例）

  > 请按 TODO 的顺序完成，如果严格按照顺序实现，参考得分如下。当然，你也可以一次性全部做完再做测试
  >
  > - `TODO1`：`Total score for all testcases: 1/34`
  > - `TODO2`：`Total score for all testcases: 8/34`
  > - `TODO3`：`Total score for all testcases: 15/34`
  > - `TODO4`：`Total score for all testcases: 23/34`
  > - `TODO5`：`Total score for all testcases: 34/34`

- 第二种使用方法：

  第一个参数可以是 `easy`、 `normal`、 `hard` 以及 `testcases_general`，并且有第二个可选参数，用于批量 `diff` 和助教提供的标准参考结果进行比较。

  第二个参数为 `no/yes/verbose`

  - `no`：不进行`diff`
  - `yes`：进行`diff`
  - `verbose`：进行`diff`，并得到更详细的输出

脚本运行后会将生成结果放在 `tests/1-parser/output_student_ast` 文件夹里，而助教的参考输出则在 `tests/1-parser/output_standard_ast` 中。
