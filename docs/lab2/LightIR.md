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
    
???+ float

	在 LLVM IR 中，浮点常量的十进制写法仅在该数值能够被 **二进制浮点格式精确表示** 时才被接受。若某个十进制小数无法用有限的二进制位精确表示（例如 5.55），则编译器会拒绝这种直接表示方式，必须改用其 **IEEE 754 位级表示的十六进制形式**，或采用带指数的科学计数法显式指明。简而言之，像 5.5 这类可以被二进制浮点精确表达的常数可以直接写作 `float 5.5`，而 5.55 则需要写成其对应的 IEEE 754 十六进制形式。需要注意的是，**在本实验中，浮点数常量统一采用双精度** 表示。
	

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


???+ Alloca	返回指针

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
	store i32 2, i32* %p1
	// *p1 = 2;
	%result = load i32, i32* %p1
	// int result = *p1;
	```

	getelementptr 同时也可以用于数组元素的地址计算，例如

	```c
	%ar = alloca [2 x i32]
	// int ar[][2]; 虽然分配的是一维数组, %ar 却是 [2 x i32]* 类型, 与 ar[][2] 访问模式类似
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

	`getelementptr` 的返回值类型**并非一定是其右边紧跟的类型**，例如 `getelementptr [2 x i32], [2 x i32]* %ar, i32 3, i32 1` 返回的类型不是 `[2 x i32]` 而是 `i32*`。

	从 `getelementptr` 的对象类型和索引维度数可以确定返回值类型，例如 `%p = getelementptr [2 x i32], [2 x i32]* %ar, i32 3, i32 1` 进行二维地址计算，最细粒度地址计算是元素的地址，所以返回类型为元素指针，即 `i32*`。而 `%p = getelementptr [2 x i32], [2 x i32]* %ar, i32 3` 进行一维地址计算，最细粒度地址计算是数组的地址，所以返回类型为数组指针，即 `[2 x i32]*`。

	以下是一些 `getelementptr` 的例子

	```
	%op1 = getelementptr i32, i32* %op0, i32 1 ; %op1 为 i32*
	%op1 = getelementptr [3 x i32], [3 x i32]* %op0, i32 1 ; %op1 为 [3 x i32]*
	%op1 = getelementptr [3 x i32], [3 x i32]* %op0, i32 1, i32 2 ; %op1 为 i32*
	%op1 = getelementptr [3 x [4 x i32]], [3 x [4 x i32]]* %op0, i32 1 ; %op1 为 [3 x [4 x i32]]*
	%op1 = getelementptr [3 x [4 x i32]], [3 x [4 x i32]]* %op0, i32 1, i32 2 ; %op1 为 [4 x i32]*
	%op1 = getelementptr [3 x [4 x i32]], [3 x [4 x i32]]* %op0, i32 1, i32 2, i32 3 ; %op1 为 i32*
	```

- 额外阅读：[The Often Misunderstood GEP Instruction](https://llvm.org/docs/GetElementPtr.html)


### 从 Cminusf 生成 Light IR

##### 变量

Cminusf 中每个变量都可能会被修改多次，故它们不适合翻译为 Light IR 中的虚拟寄存器（虚拟寄存器只能被赋值一次）。

在 C 语言中，局部变量最终会存储到函数栈帧上，而 `alloca` 正是在函数栈帧分配内存的指令，每个 Cminusf 变量刚好可以转化为一个 `alloca` 分配变量。

`alloca` 得到指向变量的指针，对变量的读取和赋值需要转换为对应的 `load` 和 `store`。

```c
int a;
// %a = alloca i32
// %a 类型为 i32*
a = 1;
// store i32 1, i32* %a
```

我们不要求翻译后的 Light IR 中虚拟寄存器名字与 Cminusf 变量名相同，但是保持相同变量名确实可以带来更好的程序可读性。在编写翻译程序时，你可以通过 `Value` 类的 `set_name` 方法来设置 Light IR 的虚拟寄存器名字，如果未设置，程序也会自动分配一个 `op1` 类似格式的名字。注意 Cminusf 允许同一函数中存在多个同名变量，但是 Light IR 不允许同一函数中具有同名虚拟寄存器，若你要自己设置名字，需要保证每个名字不同。

```c
int a;
{
	int a; // ok
} 
```

```c
%a = alloca i32
%a = alloca i32 // error
```

我们建议将所有 `alloca` 语句放在 Light IR 函数的入口块的开头处，即使变量声明可能位于 Cminusf 函数的各个位置。考虑如下函数

```c
while(1)
{
	int b;
}
```

当将 `%b = alloca i32` 放置到循环内而非函数开头时，`alloca` 会在每次循环时都在函数栈帧上分配内存，由于函数栈帧上的内存只会在函数运行结束后释放，这将导致内存泄漏。

为了将 `alloca` 语句创建在基本块开头，你可以使用 `create_alloca_begin` 函数，它创建 `alloca` 并置于基本块开头处。多个 `alloca` 之间的顺序无关紧要，它们不会影响程序的正确性和性能。

##### 全局变量

Cminusf 的全局变量同样可能被修改多次。在 C 语言中，全局变量被分配到程序的数据段，Light IR 同样有在程序数据段创建变量的方法。

```C
int a;
// @a = global i32 0
// @a 类型为 i32*
int b[2];
// @b = global [2 x i32] zeroinitializer
// @b 类型为 [2 x i32]*
```

`global` 同样得到指针，你可以对其进行 `load` 和 `store`

```C
a = 1;
// store i32 1, i32* @a
```


##### 表达式中间结果


Cminusf 中的表达式需要使用 Light IR 的多个表达式来表达，由于其中的临时变量只会被赋值一次，可以使用虚拟寄存器来表示临时变量，而不必使用 `alloca`。

```c
int a;
// %a = alloca i32
a = (1 + 2) + 3;
// %op0 = add i32 1 2
// %op1 = add i32 %op0 3
// store i32 %op1, i32* %a
```


##### 函数参数类型

Cminusf 中的简单类型都拥有对应的 Light IR 类型

```c
void f(int a, float b)
// define void @f(i32 %a, float %b)
```

而对于数组函数参数，其没有具体大小的 `[]` 实际上代表了指针

```c
void f(int a[], int b[][2])
// define void @f(i32* %a, [2 x i32]* %b)
```

##### 函数参数传递

Cminusf 使用值传递参数，你需要传递值而非地址。这意味着你需要在传递时进行 `load` 操作

```c
int a;
// %a = alloca i32
f(a);
// %op0 = load i32, i32* %a
// call void @f(i32 %op0)
```

另一个需要关注的点是传递数组参数，由于函数参数和数组类型不同，你可能需要使用 `getelementptr`

```C
void f(int a[]);
// define void @f(i32* %a)
void main(void)
{
	int a[2];
	// %a = alloca [2 x i32]
	f(a);
	// %op0 = getelementptr [2 x i32], [2 x i32]* %a, i32 0, i32 0
	// call void @f(i32* %op0)
}
```

当然在参数类型相同时就不用

```C
void f(int a[]);
// define void @f(i32* %a)
void g(int a[])
// define void @g(i32* %a)
{
	f(a);
	// call void @f(i32* %a)
	// 这不是翻译时的标准做法，它经过了 mem2reg 优化
}
```

##### 函数参数存储

Cminusf 的函数形参是一个变量，例如 `void f(int a)`，它可以在函数内被赋值。
然而，Light IR 位于函数参数位置的标识符是虚拟寄存器，例如 `define void f(i32 %a)`，`%a` 是虚拟寄存器，值无法改变。

解决这个问题的做法是在函数开始执行时将函数参数传入的值存进函数栈帧

```C
void g(int a, int b[])
// define void @g(i32 %a, i32* %b)
{
	// %ap = alloca i32
	// %bp = alloca i32*
	// store i32 %a, i32* %ap
	// store i32* %b, i32** %bp
	a = 1;
	// store i32 1, i32* %ap
	*b = 2;
	// %op0 = load i32*, i32** %bp
	// store i32 2, i32* %op0
}
```

Cminusf 没有指针赋值操作，函数参数中的指针永远不会改变。上述例子的 `%bp = alloca i32*` 是多余的，`%bp` 的 `load` 操作可以用 `%b` 替代，转化为

```C
void g(int a, int b[])
// define void @g(i32 %a, i32* %b)
{
	// %ap = alloca i32
	// store i32 %a, i32* %ap
	a = 1;
	// store i32 1, i32* %ap
	*b = 2;
	// store i32 2, i32* %b
}
```


这样做会使 IR 生成变得困难，所以这步操作通常在 IR 优化而非在从 Cminusf 生成 Light IR 阶段进行。

