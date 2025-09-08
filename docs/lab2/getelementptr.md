# 剖析 LLVM 中的 getelementptr：实例详解

> 这篇文章翻译自[LLVM's getelementptr, by example](https://blog.yossarian.net/2020/09/19/LLVMs-getelementptr-by-example)，原作者是[William Woodruff](https://yossarian.net/)，发布于 2020 年 9 月。

![one-fear-gep](figs/one-fear-gep.png)

## 前言

我已经在基于 LLVM 的项目中摸爬滚打了两年多，对阅读 LLVM IR 也早已驾轻就熟。尽管如此，每次我看到`getelementptr`指令的时候，我**仍然**要停下来想一想它的语义。所以这篇文章将用来帮助我自己（也许也能够帮助读者你）更加熟悉`getelementptr`。

## 背景

首先，你要知道有个东西叫做[LLVM](https://llvm.org/)。

LLVM 最初代表“Low Level Virtual Machine”（低级虚拟机），[但现在这个缩写已经完全不代表任何东西了](https://lists.llvm.org/pipermail/llvm-dev/2011-December/046445.html)。

LLVM 项目包含了很多部分，但对于普通开发者来说，LLVM 最出名的部分大概是作为 C/C++ 优化编译器的 Clang[^1]。

除了本身是个相当强大的编译器工程，LLVM 最为人称道的就是它的中间表示（Intermediate Representation，IR）。有一个设计优秀、成熟稳定的 IR，对一个优化编译器来说有不少好处：

- **高效优化**：LLVM 的 IR 能准确反映原始程序的语义（包括 C 的抽象语义），同时提供了重要的优化原语。所有的优化都是通过一系列独立的转换步骤（“passes”）直接在 IR 上进行的。
- **加速语言开发**：编程语言的开发者可以选择将 LLVM 的 IR 作为目标，而不是生成目标平台上执行的代码，这样可以直接利用 LLVM 支持的[所有平台](https://github.com/llvm/llvm-project/tree/main/llvm/lib/Target)和现有的优化功能。
- **程序分析**：同样的特性让 LLVM 的 IR 不仅适合做优化，也适合做静态和动态的程序分析。再加上 LLVM 的 IR 和 API 的稳定性，它已经成为程序分析研究中的首选工具。

简单来说，LLVM 的 IR 大致可以分为四个层级，范围逐渐缩小：

- [**模块（Modules)**](https://llvm.org/doxygen/classllvm_1_1Module.html)，大致对应翻译单元（比如一个 C 源文件和它的头文件）
- [**函数（Functions）**](https://llvm.org/doxygen/classllvm_1_1Function.html)，对应的就是源码中的函数（你猜对了）
- [**基本块（Basic Blocks）**](https://llvm.org/doxygen/classllvm_1_1BasicBlock.html)，对应的是通过控制流连接的一段顺序执行的代码段。
- [**指令（Instructions）**](https://llvm.org/doxygen/classllvm_1_1Instruction.html)，对应的是具体的操作（比如加法、减法、load、store 等）

指令操作的是[**值（Values）**](https://llvm.org/doxygen/classllvm_1_1Value.html)，而函数、基本块和指令本身也是一种值[^2]（当然还有其他类型的值），这样就形成了完整的框架。

这篇博客讨论的是一个单独的 LLVM 指令：`getelementptr`。

## 第一步

以下是 LLVM 10 的[LLVM 语言参考手册](https://llvm.org/docs/LangRef.html#getelementptr-instruction)对`getelementptr`的定义和描述：

```
<result> = getelementptr <ty>, <ty>* <ptrval>{, [inrange] <ty> <idx>}*
<result> = getelementptr inbounds <ty>, <ty>* <ptrval>{, [inrange] <ty> <idx>}*
<result> = getelementptr <ty>, <ptr vector> <ptrval>, [inrange] <vector index type> <idx>
```

> `getelementptr`指令用于获取聚合数据结构的子元素的地址。它只执行地址计算，而不访问内存。该指令还可以用于计算一组这样的地址的向量。

换句话说：它是 x86 指令[`lea`](https://c9x.me/x86/html/file_module_x86_id_153.html)的增强版：它计算某个字段的有效地址，而实际上不访问任何内存。我说它是“增强版”是因为它能做一些单个`lea`无法做到的事情：

- 单个`lea`能计算间接地址的能力受到 x86 寻址模式的限制：如果不能通过缩放、索引、基址和位移参数的组合计算数据结构中的有效地址，就需要多个`lea`才能到达最后要求的地址。而在`getelementptr`中，LLVM 前端可以生成任意数量的嵌套索引，仅用一个`getelementptr`指令就能实现。
- 像大多数 LLVM 指令一样，`getelementptr`有一个“向量”变体：它不仅返回一个计算的地址，还可以返回一组一起计算的地址。因此，对于向量变体，某些（甚至全部）的`getelementptr`索引参数必须是向量本身。

我已经把官方的概述读了几十遍，**大体**上我也理解它。然而，每次真正阅读 IR 中的`getelementptr`时，我还是有些眼花缭乱。LLVM 的开发者们也意识到了`getelementptr`有多么让人感到迷茫困惑：他们专门创建了[一整个详细的页面](https://llvm.org/docs/GetElementPtr.html)来详细解释它的语法，为什么它不涉及内存操作，以及构造一个有效的`getelementptr`所需要的各种限制条件。

非常令人遗憾的是，这个页面没有很多与真实 C 代码的对照示例。所以，这就是我们在这里要做的。所有示例都将使用全局变量和`-fno-discard-value-names`（告诉编译器不要丢弃变量和函数的名称）来使生成的 IR 更容易阅读。

## 基本寻址

让我们来看一个简单的 C 函数，它仅返回全局数组中的第一个元素：

```c
long *nums = {1, 2, 3};
long index_first(void) {
    return nums[0];
}
```

生成的 IR 如下（稍微做了化简和整理）：

```llvm
@nums = dso_local global i64* inttoptr (i64 1 to i64*), align 8

define dso_local i64 @index_first() #0 {
  %0 = load i64*, i64** @nums, align 8
  %arrayidx = getelementptr inbounds i64, i64* %0, i64 0
  %1 = load i64, i64* %arrayidx, align 8
  ret i64 %1
}
```

（在[godbolt](https://godbolt.org/z/G6G1xr)上查看）

我们一行一行来看：

1. 我们将全局变量`nums`加载（`load`）到`%0`中，类型为`i64*`[^3]。
2. 我们使用`getelementptr`计算一个`i64`（第一个参数）的地址，使用：

   - `%0`作为基址（第二个参数）
   - `0`作为索引（第三个参数）

   。。。然后我们将计算得到的地址存储到`%arrayidx`中。

3. 我们从计算出的地址（`%arrayidx`）中加载（`load`）一个`i64`到`%1`。
4. 我们返回加载的值（`%1`）。

看起来还不错！`getelementptr`的行为与它所模拟的简单索引操作**完全**一致。

**如果我们将`nums`改为数组呢？** 在 C 中，数组实际上就是指针，所以结果应该是一样的，对吧？

```c
long nums[] = {1, 2, 3};
long index_first(void) {
    return nums[0];
}
```

错啦！LLVM IR 并不是 C，它不遵循 C 的规则。尤其是对有固定大小的数组类型，它可以被转换为指针，但这一切不会自动发生：

```llvm
@nums = dso_local global [3 x i64] [i64 1, i64 2, i64 3], align 16

define dso_local i64 @index_first() #0 {
  %0 = load i64, i64* getelementptr inbounds ([3 x i64], [3 x i64]* @nums, i64 0, i64 0), align 16
  ret i64 %0
}
```

上面这段代码一定程度上变简单了，但在某些地方又变复杂了：`getelementptr`现在内嵌在`load`指令中[^4]，但是我们也省去了一个`load`指令和两个 SSA 变量。

我们对`getelementptr`进行分析：

- 我们正在计算一个`[3 x i64]`（即 3 个`i64`组成的数组）的地址（第一个参数）。
- `nums`是我们的基址，类型为`[3 x i64]*`（第二个参数）。

等下，为什么有**两个**`i64 0`索引而不是一个呢？咋回事？

据我理解，这是我们给予`nums`的新类型（也就是数组类型）所造成的诡异之处：它不再是指针，而是数组。然而，因为我们**通过指针**访问它（`getelementptr`就是这么定义的，这里的参数会被视作一个指针），所以我们实际上需要两个 0 索引：第一个用于解引用指针，第二个用于索引数组本身。我们所做的操作与前面的相同，只是在 IR 中稍微绕了一些弯子[^5]。

## 更多的间接寻址

当我们增加一个源代码中的间接层次，例如一个索引变量，会发生什么呢？

```c
long nums[] = {1, 2, 3};
long i;
long index_i(void) {
  return nums[i];
}
```

```llvm
@nums = dso_local global [3 x i64] [i64 1, i64 2, i64 3], align 16
@i = common dso_local global i64 0, align 8

define dso_local i64 @index_i() #0 {
  %0 = load i64, i64* @i, align 8
  %arrayidx = getelementptr inbounds [3 x i64], [3 x i64]* @nums, i64 0, i64 %0
  %1 = load i64, i64* %arrayidx, align

8
  ret i64 %1
}
```

（在[godbolt](https://godbolt.org/z/MvMj5o)上查看）

这和我们之前的简单示例几乎是相同的！唯一有变化的是最后的索引从 0 变成了`%0`（它对应于加载的变量`i`）。这进一步确认了我们关于两个`i64 0`的观点：第一个（仍然存在）只是解引用指针，而第二个执行实际的索引操作。

接下来，我们加点~~料~~维度：

```c
long nums[3][3] = { {1, 2, 3}, {2, 3, 4}, {3, 4, 5} };
long i;
long index_i2(void) {
  return nums[i][i];
}
```

生成的 IR：

```llvm
@nums = dso_local local_unnamed_addr global [3 x [3 x i64]] [[3 x i64] [i64 1, i64 2, i64 3], [3 x i64] [i64 2, i64 3, i64 4], [3 x i64] [i64 3, i64 4, i64 5]], align 16
@i = common dso_local local_unnamed_addr global i64 0, align 8

define dso_local i64 @index_i2() local_unnamed_addr #0 {
  %0 = load i64, i64* @i, align 8
  %arrayidx1 = getelementptr inbounds [3 x [3 x i64]], [3 x [3 x i64]]* @nums, i64 0, i64 %0, i64 %0
  %1 = load i64, i64* %arrayidx1, align 8
  ret i64 %1
}
```

（在[godbolt](https://godbolt.org/z/6rshW3)上查看）

完全就像我们预期的那样：我们只是用相同的索引（`%0`）访问了聚合类型中更深的一层，`getelementptr`也自然而然的有了一个额外的参数。

再加一点数学运算就能看得很清晰了，索引的顺序与我们预期是一致的。

```C
long nums[3][3] = { {1, 2, 3}, {2, 3, 4}, {3, 4, 5} };
long i;
long index_i2(void) {
  return nums[i][i + 1];
}
```

变成了：

```llvm
@nums = dso_local local_unnamed_addr global [3 x [3 x i64]] [[3 x i64] [i64 1, i64 2, i64 3], [3 x i64] [i64 2, i64 3, i64 4], [3 x i64] [i64 3, i64 4, i64 5]], align 16
@i = common dso_local local_unnamed_addr global i64 0, align 8

define dso_local i64 @index_i2() local_unnamed_addr #0 {
  %0 = load i64, i64* @i, align 8
  %add = add nsw i64 %0, 1
  %arrayidx1 = getelementptr inbounds [3 x [3 x i64]], [3 x [3 x i64]]* @nums, i64 0, i64 %0, i64 %add
  %1 = load i64, i64* %arrayidx1, align 8
  ret i64 %1
}
```

（在[godbolt](https://godbolt.org/z/c85rsx)上查看）

## 结构体和其他复合类型

之前我们讨论了指针和数组，但实际上，`getelementptr`对于任何聚合数据结构（包括结构体）都有很好的支持。

请做好心理准备，我们来看一个更复杂的例子：

```c
typedef struct foo {
  struct {
    long field1;
    struct {
      long field2;
      long field3;
      union {
        long field4;
        char field5[32];
      } quux;
      long field6;
      long field7;
    } baz;
    long field8;
  } bar;
} foo;

foo chunky;

char take_field(void) {
  return chunky.bar.baz.quux.field5[17];
}
```

再一次地，LLVM 无比可靠地吐出了一个优雅简单的`getelementptr`：

```llvm
%struct.foo = type { %struct.anon }
%struct.anon = type { i64, %struct.anon.0, i64 }
%struct.anon.0 = type { i64, i64, %union.anon, i64, i64 }
%union.anon = type { i64, [24 x i8] }

@chunky = common dso_local local_unnamed_addr global %struct.foo zeroinitializer, align 8

define dso_local signext i8 @take_field() local_unnamed_addr #0 {
  %0 = load i8, i8* getelementptr inbounds (%struct.foo, %struct.foo* @chunky, i64 0, i32 0, i32 1, i32 2, i32 1, i64 9), align 1
  ret i8 %0
}
```

（在[godbolt](https://godbolt.org/z/8hf8xT)上查看）

我们再来一步一步解析这条`getelementptr`：

- `i64 0`：解引用指针！`getelementptr`接受了一个指向`chuncky`的**指针**，而非`chunky`自身。
- `i32 0`：结构体`foo`的第一个字段（即`foo.bar`）。
  - 注意到这里没有其他合法的索引：因为`foo`唯一的一个字段就是`bar`。
- `i32 1`：嵌套聚合的第二个字段（即`foo.bar.baz`[^6]）。
  - 注意这里的`i32 0`会是`foo.bar.field1`，类似的，`i32 2`会是`foo.bar.field8`。换句话说，**我们每进入一个新的聚合，我们就会从 0 开始重新计数**。
- `i32 2`：下一层嵌套的第三个字段（即`foo.bar.baz.quux`）。
- `i32 1`：再下一层嵌套的第二个字段（即`foo.bar.baz.quux.field5`）。
  - 注意这里的`quux`是一个`union`而非`struct`，但 LLVM 不关心这种事，在它看来都是一样的聚合数据结构。
- `i64 9`：。。。啥玩意儿？

怎么回事？我们的`17`跑到哪去了？这个`9`又是从哪冒出来的？

让我们仔细看看`union`的定义：

```llvm
%union.anon = type { i64, [24 x i8] }
```

首先，这并非一个真正的`union`。**LLVM 并没有真正的"`union`"类型。**

但是，如果我们仔细观察，我们就会发现 LLVM 聪明的地方：`%union.anon` 实际上是一个结构体，其中后一部分`field5`只有$24$个字节，而非定义中的$32$个字节——开头的$8$个字节属于`field4`。就这样，LLVM 模拟了 C 中的`union`语义而避免了实现自己的`union`。

所以为什么是`i64 9`？我们要注意，这里涉及的是`[24 x i8]`，也就是 field5 的后 24 个字节。换句话说，由于前面有`i32 1`的索引，我们已经跳过了`field5`的前$8$个字节（也就是`sizeof(i64) == 8`），于是$8 + 9 = 17$，刚好是我们要的索引！LLVM 并没有让我们失望。

等下，但如果我们尝试访问`field5`和`field4`重叠的部分，比如`field5[6]`，会发生什么呢？

```llvm
%0 = load i8, i8* getelementptr inbounds ([32 x i8], [32 x i8]* bitcast (%union.anon* getelementptr inbounds (%struct.foo, %struct.foo* @chunky, i64 0, i32 0, i32 1, i32 2) to [32 x i8]*), i64 0, i64 6), align 2
```

![](figs/equations.gif)

这里有一个嵌套的`getelementptr`表达式，但它实际上并没有它看起来那么糟糕：

- 里面一层 `getelementptr`获取整个`union`也就是`foo.bar.baz.quux`。
- `bitcast`将其转换为适当的`union`变量指针（`[32 x i8]*`）。
- 外面一层`getelementptr`则对转换后的指针解引用（还记得我们前面说的吗，这是`i64 0`在这里的作用），然后获取位于下标`6`的`field5[6]`的地址。

换句话说，当我们需要访问`union`中的某些字段时，有时需要两个或更多的`getelementptr`[^7]操作来计算最终的地址。

## 向量地址计算

（译者注：这个部分似乎有着不少错误，因此做了不少调整）

比较令人尴尬的是，我花了差不多半个小时才理解上面的结构体的示例。那么，为啥不继续~~被虐~~学习呢？

正如文章一开始所提到的那样，`getelementptr`不仅可以处理单个地址和索引，它还能处理成组的向量地址和索引。

我花了大约一个小时试图让 LLVM 生成一个向量化的`getelementptr`，但非常不幸失败了。所以，我决定手动构造几个例子。

首先，我们来看一个基本的情况：用一个常量标量值索引向量中的每一个地址：

```
%foo = getelementptr i64, <8 x i64*> %tbl, i64 0
```

注意，第一个参数**并不**是一个向量，它依旧是我们计算地址时的基础类型。

我们在第二个参数才引入了向量：一个`8 x i64*`类型的向量。

最后是我们的常数标量`0`。这个值会被应用到`%tbl`的每个成员，相当于：

```c
long *foo[8], *tbl[8];

foo[0] = tbl[0][0];
foo[1] = tbl[1][0];
foo[2] = tbl[2][0];
// ...
foo[7] = tbl[7][0];
```

或者说：

```C
long *foo[8], *tbl[8];

foo[0] = &tbl[0] + 0 * sizeof(long);
foo[1] = &tbl[1] + 0 * sizeof(long);
foo[2] = &tbl[2] + 0 * sizeof(long);
// ...
foo[7] = &tbl[7] + 0 * sizeof(long);
```

就是这样！一条指令完成了 8 次地址计算。

接下来，我们使用一个非常数的标量：

```
%bar = getelementptr i64, <8 x i64*> %tbl, i64 %offset
```

如果你理解了前面的例子，这一步也很简单：我们不再使用`0`进行索引，而是使用`%offset`中指定的值：

```c
long *foo[8], *tbl[8];
long offset;

foo[0] = tbl[0][offset];
foo[1] = tbl[1][offset];
foo[2] = tbl[2][offset];
// ...
foo[7] = tbl[7][offset];
```

或者说：

```c
long *foo[8], *tbl[8];
long offset;

foo[0] = &tbl[0] + offset * sizeof(long);
foo[1] = &tbl[1] + offset * sizeof(long);
foo[2] = &tbl[2] + offset * sizeof(long);
// ...
foo[7] = &tbl[7] + offset * sizeof(long);
```

上面使用的都是标量索引。那如果用向量索引呢？

```llvm
%bar = getelementptr i64, <8 x i64*> %tbl, <8 x i64> %offsets
```

首先，注意到`%offsets`和`%tbl`的大小必须要相同：它们都是`<8 x i64>`。这是向量化形式的一个要求：在`getelementptr`中使用的向量必须具有相同的大小。

这是因为`getelementptr`的语义是非常简单的，它不会进行任何线性代数的运算，只是将基（或过程中间）的向量的每个元素映射到其对应的索引：

```c
long *foo[8], *tbl[8];
long offsets[8];

foo[0] = tbl[0][offsets[0]];
foo[1] = tbl[1][offsets[1]];
foo[2] = tbl[2][offsets[2]];
// ...
foo[7] = tbl[7][offsets[7]];
```

或者说：

```c
long *foo[8], *tbl[8];
long offsets[8];

foo[0] = tbl[0] + offsets[0] * sizeof(long);
foo[1] = tbl[1] + offsets[1] * sizeof(long);
foo[2] = tbl[2] + offsets[2] * sizeof(long);
// ...
foo[7] = tbl[7] + offsets[7] * sizeof(long);
```

值得一提的是，`getelementptr`的索引参数没有数量限制，也不需要按特定顺序排列：

比如，假设我们引入结构体，有了一个更复杂的情况

```C
struct RT {
  char A;
  int B[10][20];
  char C;
};
struct ST {
  int X;
  double Y;
  struct RT Z;
};
```

那么

```llvm
%bar = getelementptr  %struct.ST, <8 x ptr> %tbl, <8 x i64> %ind1, i32 2, i32 1, <8 x i32> %ind4, i64 13
```

这样的 LLVM IR 也是可行的。

## 一些其他的碎碎念

我还没有提到`getelementptr`指令中的一些其他可以使用的关键词，比如`inbounds`和`inrange`。

简单来说：

- **`inbounds`**：如果最终计算出的地址（或在向量模式下逐个计算出的地址）不在任何一个已分配对象的边界内，那么`inbounds`会将`getelementptr`的结果变成一个“有毒的值”（poison value）。“有毒的值”的概念超出了本文的范围，但顾名思义，它们通常是你不希望出现的东西[^8]。

- **`inrange`**：如果从最终计算出的地址进行的任何加载或存储操作超出了`inrange`标记索引（或索引组）的范围，那么这些操作会被视为未定义行为。目前，`inrange`仅适用于`getelementptr`表达式（例如嵌套在另一条指令中的表达式），而不是独立的完整指令。

## 总结

总的来说，`getelementptr`确实有点不直观且冗长，但也没有那么糟糕。

`getelementptr`总是非常准确而严格的：它总能告诉我们它计算的地址的类型，从哪里开始计算，以及如何得到最终有效地址的，沿着聚合数据结构的“路径”。它功能强大且灵活，就像 LLVM IR 本身一样。

[^1]: 在开源领域，LLVM 是唯一能与[GCC](https://en.wikipedia.org/wiki/GNU_Compiler_Collection)抗衡的竞争者，而且优势明显。

[^2]: 字面上来说：`llvm::Function`等类型都是从`llvm::Value`派生而来的。

[^3]: 你可能会注意到，我们从一个`i64**`类型中加载数据，尽管`nums`只是一个`i64*`。这是 LLVM 语义上的一些“有趣”之处：`load`指令需要一个`<ty>*`类型作为来源，所以加载一个`i64*`就需要`i64**`。

[^4]: 如果你是个杠精，你可能会说，这些内联的 GEP 在 LLVM 中实际上并不是指令，而是常量表达式（即`llvm::GetElementPtrConstantExpr`而非`llvm::GetElementPtrInst`）。但两者的语义对于我的解释来说是相同的，所以别挑我的毛病。

[^5]: 此事在 LLVM 官方文档中亦有[记载](https://llvm.org/docs/GetElementPtr.html#why-is-the-extra-0-index-required)。

[^6]: 我完全不知道为什么 LLVM 在这个 GEP 中会在`i64`和`i32`之间切换。大多数 LLVM 文档似乎暗示这其实无关紧要。

[^7]: 同样说给杠精：这些是常量表达式，而不是“真正”的 GEP，因此它们基本上不会带来额外的开销。

[^8]: 没有`inbounds`的话，`getelementptr`总会返回某个结果地址，这个地址可能有效，也可能无效。
