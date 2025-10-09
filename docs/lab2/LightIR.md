# Light IR

## Light IR 简介

本课程以 Cminusf 语言为源语言，从 LLVM IR 中裁剪出了适用于教学的精简的 IR 子集，并将其命名为 Light IR。同时依据 LLVM 的设计，为 Light IR 提供了配套简化的 [C++ 库](./LightIR C++.md)，用于生成 IR。

如下是一段 C 语言代码 `easy.c` 与其对应的 LLVM IR 文件 `easy.ll` 和 Light IR `easy_light.ll` 示例。

- `easy.c`:

  ```c
  int main(){
    int a;
    int b;
    a = 1;
    b = 2;
    return a + b;
  }
  ```

- `easy.ll`:

  ```c
  ; 整个 .ll 文件称为 module
  ; ModuleID = 'easy.c'
  ; ...
  ; module 中至少有一个 main function
  define dso_local i32 @main() #0 {
    ; 此处 main function 仅有 1 个 basicblock
    ; basicblock 由一系列 instruction 组成
    %1 = alloca i32, align 4
    %2 = alloca i32, align 4
    %3 = alloca i32, align 4
    store i32 0, i32* %1, align 4
    store i32 1, i32* %2, align 4
    store i32 2, i32* %3, align 4
    %4 = load i32, i32* %2, align 4
    %5 = load i32, i32* %3, align 4
    %6 = add nsw i32 %4, %5
    ret i32 %6
  }
  ; ...
  ```


- `easy_light.ll`:

  ```c
  ; ModuleID = 'easy.c'
  ; ...
  ; 无 dso_local 函数说明符和 #0 属性组
  define i32 @main() {
    ; 此处 main function 仅有 1 个 basicblock, 可以选择标出 basicblock 或者像 easy.ll 忽略
	; operand 和 label 命名方法与 clang 生成的 llvm ir 不同, 但二者都可以工作
  label_entry:
	; 无 align 4 对齐
    %op0 = alloca i32
    %op1 = alloca i32
    store i32 1, i32* %op0
    store i32 2, i32* %op1
    %op2 = load i32, i32* %op0
    %op3 = load i32, i32* %op1
	; 无 nsw 溢出检查
    %op4 = add i32 %op2, %op3
    ret i32 %op4
  }
  ; ...
  ```

## Light IR 指令

### Light IR 指令假设

Light IR 指令从 LLVM IR 中裁剪得到，因此保留了 LLVM IR 如下的指令规范

- 采用 3 地址的方式
  - `%2 = add i32 %0, %1`
- 无限寄存器 + 静态单赋值形式
  - IR 中的变量均代表了一个虚拟寄存器，并且数量无上限
  - 每个虚拟寄存器只被赋值一次
- 强类型系统

  - 每个操作数都具备自身的类型，分为基本类型，以及组合类型

    基本类型：

    - `i1`：1 位宽的整数类型
    - `i32`：32 位宽的整数类型
    - `float`：单精度浮点数类型
    - `label`: 基本块的标识符类型

    组合类型：

    - 指针类型：`<type> *`，例如 `i32*, [10 x i32*]`
    - 数组类型：`[n x <type>]`，例如 `[10 x i32], [10 x [10 x i32]]`
    - 函数类型：`<ret-type>@(<arg-type>)`，由函数返回值类型与参数类型列表组合成的类型

    数组和指针类型可以嵌套，例如前面的 `[10 x [10 x i32]]` 就是嵌套后的类型。

### Light IR 指令详解

#### Terminator Instructions

**注**：`ret` 与 `br` 都是 Terminator Instructions 也就是终止指令，在 llvm 基本块的定义里，基本块只能有一条终止指令，并且一定是基本块的最后一条指令（`ret` 或 `br`）。

##### Ret

- 概念：返回指令。用于将控制流（以及可选的值）从函数返回给调用者。`ret`指令有两种形式：一种返回值，然后终结函数，另一种仅终结函数。
- 格式
  - `ret <type> <value>`
  - `ret void`
- 例子：
  - `ret i32 %0`
  - `ret void`

##### Br

- 概念：跳转指令。用于使控制流转移到当前函数中的另一个基本块。该指令有两种形式，分别对应于条件分支和无条件分支。
- 格式：
  - `br i1 <cond>, label <iftrue>, label <iffalse>`
  - `br label <dest>`
- 例子：
  - `br i1 %cond, label %truebb, label %falsebb`
  - `br label %bb`

#### Standard Binary Instructions

##### Add FAdd

- 概念：`add`指令返回其两个`i32`类型的操作数之和，返回值为`i32`类型，`fadd`指令返回其两个`float`类型的操作数之和，返回值为`float`类型。
- 格式：
  - `<result> = add <type> <op1>, <op2>`
  - `<result> = fadd <type> <op1>, <op2>`
- 例子：
  - `%2 = add i32 %1, %0`
  - `%2 = fadd float %1, %0`

##### Sub FSub

- 概念：`sub`指令返回其两个`i32`类型的操作数之差，返回值为`i32`类型，`fsub`指令返回其两个`float`类型的操作数之差，返回值为`float`类型。
- 格式与例子与`add`，`fadd`类似

##### Mul FMul

- 概念：`mul`指令返回其两个`i32`类型的操作数之积，返回值为`i32`类型，`fmul`指令返回其两个`float`类型的操作数之积，返回值为`float`类型。
- 格式与例子与`add`，`fadd`类似

##### SDiv FDiv

- 概念：`sdiv`指令返回其两个`i32`类型的操作数之商，返回值为`i32`类型，`fdiv`指令返回其两个`float`类型的操作数之商，返回值为`float`类型。
- 格式与例子与`add`，`fadd`类似

#### Memory Instructions

##### Alloca

- 概念： `alloca`指令在当前执行函数的栈帧（Stack Frame）上分配内存，并且得到指向该内存的指针。
- 格式：`<result> = alloca <type>`
- 例子：
  - `%ptr = alloca i32`
  - `%ptr = alloca [10 x i32]`


???+ Alloca 返回指针

	使用 `%1 = alloca i32` 在内存分配一个整数，得到的 `%1` 是 `i32*` 类型，而使用 `%2 = alloca [10 x i32]` 在内存分配一个数组，得到的 `%2` 是 `[10 x i32]*` 数组指针类型。要想获得数组的元素，需要对 `%2` 添加两个数组索引，类似于 `a[0][2]`，第一维对指针进行索引，第二维对数组进行索引。

##### Load

- 概念：`load`指令用于从内存中读取。
- 格式：`<result> = load <type>, <type>* <pointer>`
- 例子：`%val = load i32, i32* %ptr`

##### Store

- 概念：`store`指令用于写入内存。
- 格式：`store <type> <value>, <type>* <pointer>`
- 例子：`store i32 3, i32* %ptr`

#### CastInst

##### ZExt

- 概念：`zext`指令将其操作数**零**扩展为`type2`类型。
- 格式：`<result> = zext <type> <value> to <type2>`
- 例子：`%1 = zext i1 %0 to i32`

##### FpToSi

- 概念：`fptosi`指令将浮点值转换为`type2`（整数）类型。
- 格式：`<result> = fptosi <type> <value> to <type2>`
- 例子：`%Y = fptosi float 1.0E-247 to i32`

##### SiToFp

- 概念：`sitofp`指令将有符号整数转换为`type2`（浮点数）类型。
- 格式：`<result> = sitofp <type> <value> to <type2>`
- 例子：`%X = sitofp i32 257 to float`

#### Other Instructions

##### ICmp FCmp

- 概念：`icmp`指令根据两个整数的比较返回布尔值，`fcmp`指令根据两个浮点数的比较返回布尔值。
- 格式：
  - `<result> = icmp <cond> <type> <op1>, <op2>`
    - `<cond> = eq | ne | sgt | sge | slt | sle`
  - `<result> = fcmp <cond> <type> <op1>, <op2>`
    - `<cond> = eq | ne | ugt | uge | ult | ule`
- 例子：`i1 %2 = icmp sge i32 %0, %1`

##### Call

- 概念：`call`指令用于使控制流转移到指定的函数，其传入参数绑定到指定的值。在被调用函数中执行`ret`指令后，如果被调用函数返回值不为 `void` 类型，控制流程将在函数调用后继续执行该指令，并且该函数的返回值绑定到`result`参数。
- 格式：
  - `<result> = call <return ty> <func name>(<function args>)`
  - `call <func name>(<function args>)`
- 例子：
  - `%0 = call i32 @func( i32 %1, i32* %0)`
  - `call @func( i32 %arg)`

##### GetElementPtr

- 概念：`getelementptr`指令用于根据数组或指针的索引计算偏移后的地址。它 **仅执行地址计算** ，不访问内存。
- 格式
  - 根据数组索引计算数组元素的地址 `<result> = getelementptr <type>, <type>* <ptrval>, i32 0, i32 <idx2>`
  - 根据函数参数中指针索引计算偏移后的地址 `<result> = getelementptr <type>, <type>* <ptrval>, i32 <idx>`
  - `getelementptr` 格式中的 ` <type>, <type>* <ptrval>`， `<ptrval>` 类型为 `<type>*`
- 例子：
  - `%2 = getelementptr [10 x i32], [10 x i32]* %1, i32 0, i32 %0`
  - `%2 = getelementptr i32, i32* %1 i32 %0`

???+ 理解GetElementPtr格式与作用

	假设需要对 ir 中 `i32*` 类型操作数 `%p` 指向的内存空间进行存取操作，可以写成

	```c
	store i32 1, i32* %p
	%result = load i32, i32* %p
	```

	这个过程可以用 C 语言描述。用 `int* p` 来表示 `i32* %p`，C 语言描述为

	```c
	*p = 1;
	int result = *p;
	```
	
	而对于如下 C 代码描述，却好像无法找到对应的 Light IR 代码

	```c
	p[1] = 2;
	int result = p[1];
	```

	这时可以使用 `getelementptr` 指令达到类似效果

	```c
	%p1 = getelementptr i32, i32* %p, i32 1
	// int* p1 = p + 1
	store i32 1, i32* %p1
	// *p1 = 2;
	%result = load i32, i32* %p1
	// int result = *p1;
	```

	getelementptr 同时也可以用于数组元素的地址计算，例如

	```c
	%ar = alloca [2 x i32]
	// int ar[][2];
	%p = getelementptr [2 x i32], [2 x i32]* %ar, i32 0, i32 1
	// int* p = ar + ((0 * 2) + 1); (其实就是计算的 &ar[0][1])
	%result = load i32, i32* %p
	// int result = *p;
	```
	
	此处 `getelementptr` 拥有两维索引，因为 `%ar` 是 `[2 x i32]*` 类型。类似于 C 中的 `int ar[][2]`，第一维索引用于指定是哪个数组，第二维索引指定是哪个元素。由于 `alloca` 仅仅分配了 1 个 2 元素数组大小的空间，第一维索引只能是 0，但是当 `%ar` 并非来自于 `alloca` 时，非 0 的第一维索引确实可能存在

	```c
	define void @func([2 x i32]* %ar) {
		// int ar[][2];
		%p = getelementptr [2 x i32], [2 x i32]* %ar, i32 3, i32 1
		// int* p = ar + ((3 * 2) + 1); (其实就是计算的 &ar[3][1])
		%result = load i32, i32* %p
		// int result = *p;
		...
	```

	`getelementptr` 的返回值类型并非一定是其右边紧跟的类型，例如 `getelementptr [2 x i32], [2 x i32]* %ar, i32 3, i32 1` 返回的类型不是 `[2 x i32]` 而是 `i32*`。

	从 `getelementptr` 的对象类型和索引维度数可以确定返回值类型，例如 `%p = getelementptr [2 x i32], [2 x i32]* %ar, i32 3, i32 1` 进行二维地址计算，最细粒度地址计算是元素的地址，所以返回类型为元素指针，即 `i32*`。而 `%p = getelementptr [2 x i32], [2 x i32]* %ar, i32 3` 进行一维地址计算，最细粒度地址计算是数组的地址，所以返回类型为数组指针，即 `[2 x i32]*`。

	以下是一些 `getelementptr` 的例子

	```
	%op1 = getelementptr i32, i32* %op0, i32 1 ; %op1 为 i32*
	%op1 = getelementptr i32, [3 x i32]* %op0, i32 1 ; %op1 为 [2 x i32]*
	%op1 = getelementptr i32, [3 x i32]* %op0, i32 1, i32 2 ; %op1 为 i32*
	%op1 = getelementptr i32, [3 x [4 x i32]]* %op0, i32 1 ; %op1 为 [3 x [4 x i32]]*
	%op1 = getelementptr i32, [3 x [4 x i32]]* %op0, i32 1, i32 2 ; %op1 为 [4 x i32]*
	%op1 = getelementptr i32, [3 x [4 x i32]]* %op0, i32 1, i32 2, i32 3 ; %op1 为 i32*
	```

- 额外阅读：[The Often Misunderstood GEP Instruction](https://llvm.org/docs/GetElementPtr.html)
