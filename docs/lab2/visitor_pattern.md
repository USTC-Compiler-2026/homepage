# 访问者模式

### 简介

Visitor Pattern（访问者模式）是一种在 LLVM 项目源码中被广泛使用的设计模式。在本实验中，指的是 **语法树** 类有一个方法接受 **访问者**，将自身引用传入 **访问者**，而 **访问者** 类中集成了根据语法树节点内容生成 IR 的规则。

我们从一个例子出发。如下 AST 共三个节点，一个顶层节点具有两个叶节点。它能表达 `1 + 2`，`2 x 4` 等简单四则运算

```CPP
class TopNode {
	ChildNode* left_;
	ChildNode* right_;
	string op_;
}
class ChildNode {
	string value_;
}
```

我们现在希望对 AST 进行操作，例如，将 AST `printf` 输出成 `1 + 2` 的形式，为 AST 代表的表达式求值。对于 `printf` 可以写成这样

```CPP
class TopNode {
	ChildNode* left_;
	ChildNode* right_;
	string op_;
	void print() {
		left_->print();
		cout << op_;
		right->print();
	}
}
class ChildNode {
	string value_;
	void print(){
		cout << value_;
	}
}
```

设想以下，我们拥有一个足够好用的库，它可以完成 Flex 和 Bison 的功能，并且还能把 `1 + 2` 转换为 AST，而无需我们手动完成 lab1。

这个工具库显然不知道我们需要对 AST 进行什么操作，它就无法写成上面这样，也就是无法在 AST 代码内实现 `print` 函数（因为它不知道我们需要输出功能，也不知道我们要输出成什么格式），虽然我们可以在使用库生成代码后手动将 `print` 函数加进去，但只要使用库重新生成 AST 的代码，我们的更改就会被覆盖。

作为替代，工具库实现了访问者类 `NodeVisitor`

```CPP
class TopNode {
	ChildNode* left_;
	ChildNode* right_;
	string op_;
	void accept(NodeVisitor* visitor) {
		visitor->visit(*this);
	}
}
class ChildNode {
	string value_;
	void accept(NodeVisitor* visitor) {
		visitor->visit(*this);
	}
}
class NodeVisitor {
	virtual void visit(TopNode* topNode) = 0;
	virtual void visit(ChildNode* topNode) = 0;
}
```

你只需要在另一个文件里继承 `NodeVisitor` 并实现接口如下

```CPP
class NodePrinter: NodeVisitor {
	void visit(TopNode* topNode){
		topNode->left_->accept(this);
		cout << topNode->op_;
		topNode->right->accept(this);
	}
	virtual void visit(ChildNode* childNode) {
		cout << childNode->value_;
	}
}
```

现在你只需要实现一个类，无需在工具库生成的源代码上改动，并且工具库重新生成代码后，你的代码也不会被覆盖。这种 `accept` 和 `visit` 的代码结构就是 **访问者模式**。

另一个好处是，当你想实现给 `1 + 2` 的 AST 求值的功能时，你只需要在另一个文件重新实现一个具有该功能的 `NodeVisitor`，而原来用作输出功能的 `NodeVisitor` 完全不需要更改。因为它们写在了两个文件，分别维护，你也绝对不会遇到他们更改时导致的 `git` 冲突。



!!! info

    有关 Visitor Pattern 的含义、模式和特点，可参考[维基百科](https://en.wikipedia.org/wiki/Visitor_pattern#C++_example)。

下面我们将通过实例来帮助大家理解 Visitor Pattern 的调用流程。

### 实验内容

<!-- TODO: 重写 -->

在 `tests/2-ir-gen/warmup/calculator` 目录下提供了一个接受算术表达式，利用访问者模式，产生计算算数表达式的中间代码的程序，其中 `calc_ast.hpp` 定义了语法树的不同节点类型，`calc_builder.cpp` 实现了访问不同语法树节点 `visit` 函数。**阅读这两个文件和目录下的其它相关代码**，理解语法树是如何通过访问者模式被遍历的，并回答相应[思考题](./visitor_pattern.md#思考题)。

### 编译、运行

**编译**

```shell
$ cd 2025ustc-jianmu-compiler
$ mkdir build
$ cd build
# 使用 cmake 生成 makefile 等文件
$ cmake ..
# 使用 make 进行编译
$ make
```

如果构建成功，你会在 `build` 文件夹下找到 calc 可执行文件。

**运行与测试**

```shell
# 在 build 目录下操作
$ ./calc
Input an arithmatic expression (press Ctrl+D in a new line after you finish the expression):
4 * (8 + 4 - 1) / 2
result and result.ll have been generated.
$ ./result
22
```

其中，`result.ll` 是 `calc` 产生的中间代码，`result` 是中间代码编译产生的二进制可执行文件，运行它就可以输出算数表达式的结果。

## 思考题

1. 分析 `calc` 程序在输入为 `4 * (8 + 4 - 1) / 2` 时的行为：

   a.请画出该表达式对应的抽象语法树（使用 `calc_ast.hpp` 定义的语法树节点来表示，并给出节点成员存储的值），并给节点使用数字编号。

   b.请给出示例代码在用访问者模式遍历该语法树时，访问者到达语法树节点的顺序。序列请按如下格式指明（序号为问题 1.a 中的编号）：3->2->5->1->1
