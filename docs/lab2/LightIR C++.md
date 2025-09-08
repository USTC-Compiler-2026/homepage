# Light IR C++ 库

Light IR C++ 库依据 LLVM 设计，用于生成 IR。在介绍其核心类之前，先展示 Light IR 的结构。

!!! warning

    在必做实验阶段，请不要对 Light IR C++ 库进行直接修改

## Light IR 结构

![image-lightir](./figs/lightir.png)

实验中需要生成的 IR 代码有着相对固定的结构模式：

- 最上层的是 module，对应一个 Cminusf 源文件。包含全局变量 global_variable 和函数 function。
- function 由头部和函数体组成。function 的头部包括返回值类型、函数名和参数表。函数体可以由一个或多个 basicblock 构成。
- basicblock 是指程序顺序执行的语句序列，只有一个入口和一个出口。basicblock 由若干指令 instruction 构成。
- 注意一个 basicblock 中 **只能有一条终止指令**（Ret/Br）。

!!! note

    为了区别 Light IR 中的概念与我们实现的 Light IR C++ 库。我们用小写 plain text 来表示 Light IR 中的概念，例如 module；用大写的 code block 来表示 C++ 实现，例如  `Module`。

## Light IR C++ 类总览

在上一节中，可以看出 Light IR 最顶层的结构是 module，并具有层次化结构，在 Light IR C++ 库中，对应有层次化类的设计，如图所示，`Module`, `Function`, `BasicBlock`, `Instruction` 类分别对应了 Light IR 中 module，function，basicblock，instruction 的概念。

![module_relation](./figs/module_relation.png)

### Light IR C++ 数据基类：Value，User

#### Value

`Value` 类代表一个可用于指令操作数的带类型的数据，包含众多子类，`Instruction` 也是其子类之一，表示指令在创建后可以作为另一条指令的操作数。`Value` 成员 `use_list_` 是 `Use` 类的列表，每个 Use 类记录了该 `Value` 的一次被使用的情况。

!!! note "use-list 详解"

    例如，如果存在指令 `%op2 = add i32 %op0, %op1`，那么 `%op0`、`%op1` 就被 `%op2` 所使用，`%op0` 基类 `Value` 的 `use_list_` 里就会有 `Use(%op2, 0)`（这里的 0 代表 `%op0` 是被使用时的第一个参数）。同理，`%op1` 的 `use_list_` 里有 `Use(%op2, 1)`。

![value_inherit](./figs/value_inherit.png)

!!! note

    `Instruction` 类是 `Value` 的子类，这表示，指令在使用操作数创建后的返回值也可以作为另一条指令创建的操作数。

#### User

`User` 作为 `Value` 的子类，含义是使用者，`Instruction` 也是其子类之一，`User` 类成员 `operands_` 是`Value` 类的列表，表示该使用者使用的操作数列表。如图是 `User` 类的子类继承关系。

![user_inherit](./figs/user_inherit.jpg)

!!! note

    `Value` 类的 use-list，与 User 类的 operand-list 构成了指令间的依赖关系图。

### Light IR C++ 类型基类：Type

在 [Light IR 指令假设](./LightIR.md#light-ir-指令假设)中提到，Light IR 保留了 LLVM IR 的强类型系统，包含基本类型与组合类型，`Type` 类是所有类型基类，其子类继承关系如图所示，其中 `IntegerType`, `FloatType` 对应表示 Light IR 中的 `i1`，`i32`，`float` 基本类型。`ArrayType`，`PointerType`，`FunctionType` 对应表示组合类型：数组类型，指针类型，函数类型。

![type_inherit](./figs/type_inherit.png)

获取基本类型的接口在 `Module` 类中，获取组合类型的接口则在组合类型对应的类中：

```c++
// 获取 i1 基本类型
auto int1_type = module->get_int1_type();
// 获取 [2 x i32] 数组类型
auto array_type = ArrayType::get(Int32Type, 2);
```

### 使用 Light IR C++ 库生成 IR

本小节将以下列 Light IR 片段介绍使用 Light IR C++ 接口层次化顺序生成 IR 的过程：

```c
define i32 @main() #0 {
entry：
  %1 = alloca i32
  store i32 72, i32* %1
  %2 = load i32, i32* %1
  ret i32 %2
}
```

#### 创建 `Module`, `Function`, `BasicBlock` 的接口

创建 module：

```cpp
auto module = new Module();
```

为 module 添加 main function 定义：

```cpp
// 从 module 中获取 i32 类型
Type *Int32Type = module->get_int32_type();
auto mainFun = Function::create(FunctionType::get(Int32Type, {Int32Type}), "main", module);
```

为 main function 创建 function 内的第一个 basicblock：

```cpp
auto bb = BasicBlock::create(module, "entry", mainFun);
```

接下来需要用辅助类 `IRBuilder` 向 basicblock 中插入指令。

#### `IRBuilder`: 生成 IR 指令的辅助类

Light IR C++ 库为生成 IR 指令提供了辅助类：`IRBuilder`。该类提供了独立创建 IR 指令的接口，可以创建指令的同时并将它们插入 basicblock 中，`IRBuilder` 类提供以下接口：

```cpp
class IRBuilder {
public:
    // 返回当前插入的基本块
    BasicBlock *get_insert_block()
    // 设置当前插入的基本块
    void set_insert_point(BasicBlock *bb)
    // 创建一条除法指令，并加入当前基本块中
    BinaryInst *create_isdiv(Value *lhs, Value *rhs)
    // 创建的 instr_type 类型的指令，并插入到当前基本块中
    // 可根据 IRBuilder.h 查看每一条指令的创建细节
    Instruction *create_[instr_type](...);
};
```

在创建 module，main function，basicblock 后：

```cpp
auto module = new Module();
// 从 module 中获取 i32 类型
auto *Int32Type = module->get_int32_type();
auto mainFun = Function::create(FunctionType::get(Int32Type, {Int32Type}), "main", module);
auto bb = BasicBlock::create(module, "entry", mainFun);
```

创建 `IRBuilder`，并使用 `IRBuilder` 创建新指令。

```cpp
// 实例化 IRBuilder
auto builder = new IRBuilder(nullptr, module);
// 将 IRBuilder 插入指令位置设置为 bb 尾部
builder->set_insert_point(bb);
// 为变量 x 分配栈上空间
auto xAlloca = builder->create_alloca(Int32Type);
// 创建 store 指令，将 72 常数存到 x 分配空间里
builder->create_store(ConstantInt::get(72, module), xAlloca);
// 创建 load 指令，将 x 内存值取出来
auto xLoad = builder->create_load(xAlloca);
// 创建 ret 指令，将 x 取出的值返回
builder->create_ret(xLoad);
```

至此，使用 Light IR C++ 接口层次化顺序生成 IR 的过程流程结束。

## Light IR C++ 库核心类定义

本节梳理了在生成 IR 过程中可能会用到的接口，学生可按需进行查阅。

### Module

- 概念：一个编译单元。对应一个 Cminusf 文件。

??? info "Module 定义"

    ```cpp
    class Module {
      public:
        Module();
        ~Module() = default;

        Type *get_void_type();
        Type *get_label_type();
        IntegerType *get_int1_type();
        IntegerType *get_int32_type();
        PointerType *get_int32_ptr_type();
        FloatType *get_float_type();
        PointerType *get_float_ptr_type();

        PointerType *get_pointer_type(Type *contained);
        ArrayType *get_array_type(Type *contained, unsigned num_elements);
        FunctionType *get_function_type(Type *retty, std::vector<Type *> &args);

        void add_function(Function *f);
        llvm::ilist<Function> &get_functions();
        void add_global_variable(GlobalVariable *g);
        llvm::ilist<GlobalVariable> &get_global_variable();

        void set_print_name();
        std::string print();

      private:
        // The global variables in the module
        llvm::ilist<GlobalVariable> global_list_;
        // The functions in the module
        llvm::ilist<Function> function_list_;

        std::unique_ptr<IntegerType> int1_ty_;
        std::unique_ptr<IntegerType> int32_ty_;
        std::unique_ptr<Type> label_ty_;
        std::unique_ptr<Type> void_ty_;
        std::unique_ptr<FloatType> float32_ty_;
        std::map<Type *, std::unique_ptr<PointerType>> pointer_map_;
        std::map<std::pair<Type *, int>, std::unique_ptr<ArrayType>> array_map_;
        std::map<std::pair<Type *, std::vector<Type *>>,
                std::unique_ptr<FunctionType>>
            function_map_;
    };
    ```

#### 属性

- `global_list_`：全局变量列表。

- `function_`：函数列表。

- `int1_ty_`：int1 类型。

- `int32_ty_`：int32 类型。

- `label_ty_`：基本块类型。

- `void_ty_`：void 类型。

- `float32_ty_`：float 类型。

- `pointer_map_`：存储不同类型指针的映射，用于缓存和快速查找指向特定类型的指针类型。键是基础类型，值是指向该类型的指针类型。

- `array_map_`：存储不同数组类型的映射，用于缓存和快速查找具有特定元素类型和大小的数组类型。键是基础类型和数组大小的组合，值是对应的数组类型。

- `function_map_`：存储不同函数类型的映射。
  用于缓存和快速查找具有特定返回类型和参数类型的函数类型。键是返回类型和参数类型的组合，值是对应的函数类型。

#### 接口

- 获取各个类型

  ```c++
  Type *get_void_type();
  Type *get_label_type();
  IntegerType *get_int1_type();
  IntegerType *get_int32_type();
  PointerType *get_int32_ptr_type();
  FloatType *get_float_type();
  PointerType *get_float_ptr_type();

  PointerType *get_pointer_type(Type *contained);
  ArrayType *get_array_type(Type *contained, unsigned num_elements);
  FunctionType *get_function_type(Type *retty, std::vector<Type *> &args);
  ```

- `add_function`：将函数 f 添加到该模块的函数链表上。在函数被创建的时候会自动调用此方法。
- `get_functions`：获取当前 module 的所有函数，即返回 private 属性 `functions_`，常在遍历一个 module 的所有 function 时使用。
- `add_global_variable`：将全局变量 g 添加到该模块的全局变量链表上。在全局变量被创建时会自动调用。
- `get_global_variable`：获取当前 module 的所有全局变量，即返回 private 属性 `global_list_`，常在遍历一个 module 的所有 global_variable 时使用。

#### 常用接口

- 获取各种类型
- `get_functions`
- `get_global_variable`

### GlobalVaribale

- 概念：全局变量。

??? info "GlobalVariable 定义"

    ```cpp
    class GlobalVariable : public User, public llvm::ilist_node<GlobalVariable> {
      private:
        bool is_const_;
        Constant *init_val_;
        GlobalVariable(std::string name, Module *m, Type *ty, bool is_const,
                      Constant *init = nullptr);

      public:
        GlobalVariable(const GlobalVariable &) = delete;
        static GlobalVariable *create(std::string name, Module *m, Type *ty,
                                      bool is_const, Constant *init);
        virtual ~GlobalVariable() = default;
        Constant *get_init() { return init_val_; }
        bool is_const() { return is_const_; }
        std::string print();
    };
    ```

#### 属性

- `is_const_`：是否是 const 类型全局变量。
- `init_val_`：全局变量初始化的值。
- 构造函数是私有的，意味着不能直接实例化 `GlobalVariable` 对象。

#### 接口

- `create`：创建一个只想全局变量的指针。
- `get_init`：返回全局变量的初始化值。
- `is_const`：检查该全局变量是否为常量。

#### 常用接口

- `get_init`
- `is_const`

### Function 和 Argument

#### Function

- 概念：函数。该类描述了一个过程，包含多个基本块。

??? info "Funtion 定义"

    ```cpp
    class Function : public Value, public llvm::ilist_node<Function> {
      public:
        Function(const Function &) = delete;
        Function(FunctionType *ty, const std::string &name, Module *parent);
        ~Function() = default;
        static Function *create(FunctionType *ty, const std::string &name,
                                Module *parent);

        FunctionType *get_function_type() const;
        Type *get_return_type() const;

        void add_basic_block(BasicBlock *bb);

        unsigned get_num_of_args() const;
        unsigned get_num_basic_blocks() const;

        Module *get_parent() const;

        void remove(BasicBlock *bb);
        BasicBlock *get_entry_block() { return &*basic_blocks_.begin(); }

        llvm::ilist<BasicBlock> &get_basic_blocks() { return basic_blocks_; }
        std::list<Argument> &get_args() { return arguments_; }

        bool is_declaration() { return basic_blocks_.empty(); }

        void set_instr_name();
        std::string print();

      private:
        llvm::ilist<BasicBlock> basic_blocks_;
        std::list<Argument> arguments_;
        Module *parent_;
        unsigned seq_cnt_; // print use
    };
    ```

##### 属性

- `basic_blocks_`：基本块列表。
- `arguments_`：参数列表。
- `parent_`：所属模块。
- `seq_cnt_`：序列计数器。

##### 接口

构造函数与析构函数：

- 删除复制构造函数，禁止复制构造 `Function`，放置以外的复制操作。
- 构造函数：需要的参数包括函数类型，函数名称和所属模块。
- 静态创建方法 `create`：返回一个指向新创建的 `Function`对象的指针。
- 使用默认的析构函数。

成员函数：

- `get_function_type`：获取函数类型。

- `get_return_type`：获取函数返回值的类型。

  函数的返回值在 Cminusf 中只会是 void/int/float。

- `add_basic_block`：添加基本块。

- `get_num_of_args`：获取参数数量。

- `get_num_basic_blocks`：获取基本块数量。

- `get_parent`：获取所属 module。

- `remove`：移除基本块。

- `get_entry_block`：获取入口基本块。

- `get_basic_blocks`：获取基本块列表。

- `get_args`：获取参数列表。

- `is_declaration`：返回函数是否仅为声明（没有定义）。

- `set_instr_name`：设置函数内所有指令的名称。

##### 常用接口

- `get_basic_blocks`：常在遍历一个 function 的基本块列表时使用。
- `is_declaration`：判断函数是否仅为声明，常在跳过没有基本块的函数声明的优化 pass 中使用。
- `get_num_basic_blocks`：获取基本块数量。
- `get_num_of_args`
- `get_entry_block`：设置 branch 跳转的目标时常用到一个函数的入口基本块。

#### Argument

- 概念：函数的参数。

??? info "Argument 定义"

    ```cpp
    class Argument : public Value {
      public:
        Argument(const Argument &) = delete;
        explicit Argument(Type *ty, const std::string &name = "",
                          Function *f = nullptr, unsigned arg_no = 0)
            : Value(ty, name), parent_(f), arg_no_(arg_no) {}
        virtual ~Argument() {}

        inline const Function *get_parent() const { return parent_; }
        inline Function *get_parent() { return parent_; }

        /// For example in "void foo(int a, float b)" a is 0 and b is 1.
        unsigned get_arg_no() const {
            assert(parent_ && "can't get number of unparented arg");
            return arg_no_;
        }

        virtual std::string print() override;

      private:
        Function *parent_;
        unsigned arg_no_; // argument No.
    };
    ```

#### 属性

- `parent_`：父函数。
- `arg_no_`：argument 的位置。

#### 接口

- `get_parent`：获取这个参数对应的函数。
- `get_arg_no`：获取 argument 的参数号。

### BasicBlock

- 概念：基本块。是一个单入口单出口的代码块，可以作为分支指令目标对象。

  继承自 Value 和 ilist_node 双向链表节点，用于将 Basicblock 对象链接到一个 ilist 中。

??? info "BasicBlock 定义"

    ```cpp
    class BasicBlock : public Value, public llvm::ilist_node<BasicBlock> {
        public:
        ~BasicBlock() = default;
        static BasicBlock *create(Module *m, const std::string &name,
                                  Function *parent) {
            auto prefix = name.empty() ? "" : "label_";
            return new BasicBlock(m, prefix + name, parent);
        }

        /****************api about cfg****************/
        std::list<BasicBlock *> &get_pre_basic_blocks() { return pre_bbs_; }
        std::list<BasicBlock *> &get_succ_basic_blocks() { return succ_bbs_; }

        void add_pre_basic_block(BasicBlock *bb) { pre_bbs_.push_back(bb); }
        void add_succ_basic_block(BasicBlock *bb) { succ_bbs_.push_back(bb); }
        void remove_pre_basic_block(BasicBlock *bb) { pre_bbs_.remove(bb); }
        void remove_succ_basic_block(BasicBlock *bb) { succ_bbs_.remove(bb); }

        // If the Block is terminated by ret/br
        bool is_terminated() const;
        // Get terminator, only accept valid case use
        Instruction *get_terminator();

        /****************api about Instruction****************/
        void add_instruction(Instruction *instr);
        void add_instr_begin(Instruction *instr) { instr_list_.push_front(instr); }
        void erase_instr(Instruction *instr) { instr_list_.erase(instr); }
        void remove_instr(Instruction *instr) { instr_list_.remove(instr); }

        llvm::ilist<Instruction> &get_instructions() { return instr_list_; }
        bool empty() const { return instr_list_.empty(); }
        int get_num_of_instr() const { return instr_list_.size(); }

        /****************api about accessing parent****************/
        Function *get_parent() { return parent_; }
        Module *get_module();
        void erase_from_parent();

        virtual std::string print() override;

        private:
        BasicBlock(const BasicBlock &) = delete;
        explicit BasicBlock(Module *m, const std::string &name, Function *parent);

        std::list<BasicBlock *> pre_bbs_;
        std::list<BasicBlock *> succ_bbs_;
        llvm::ilist<Instruction> instr_list_;
        Function *parent_;
    };
    ```

可以发现，`BasicBlock` 继承自 `Value` 类，故享有 `Value` 类自带的一些接口及属性，如 `get_name`，`set_name`，`add_use`等等，可酌情使用。

#### 属性

- `pre_bbs_`：是一个 std 的`list`类型的列表，存储前驱基本块列表。
- `succ_bbs_`：是一个 std 中的 `list`类型的列表，存储后继基本块列表。
- `instr_inst_`：指令列表。
- `parent_`：父函数。

#### 接口

构造函数和静态创建方法：

- 使用默认的析构函数。
- 用 `create` 来创建 `BasicBlock` 对象，返回一个指向新创建对象的指针。如果 name 为空，使用默认前缀 `label_`。

控制流图相关 API：

- `get_pre_basic_blocks`：返回前驱基本块集合。
- `get_succ_basic_blocks`：返回后继基本块集合。
- `add_pre_basic_block`：添加前驱基本块。
- `add_succ_basic_block`：添加后继基本块。
- `remove_pre_basic_block`：移除前驱基本块。
- `remove_succ_basic_block`：移除后继基本块。
- `is_terminated`：如果基本块由一条 branch/ret 指令终止，返回 true，表明这个基本块合法。
- `get_terminator`：获取最后一条指令（ret/branch 指令）。

指令相关的 API：

- `add_instruction`：在该基本块添加指令。
- `add_instr_begin`：在基本块最前端添加一条指令。
- `erase_instr`：删除一条指令，并注销这条指令的生命周期（不能再次引用）。
- `remove_instr`：移除一条指令，这条指令的内存并没有释放，依然可以作为一个对象插入其他基本块等。
- `get_instructions`：获取该基本块的指令列表。
- `empty`：判断基本块是否为空。
- `get_num_of_instr`：获取基本块中的指令数量。

处理父类的 API

- `get_parent`：获取基本块的父函数。
- `get_module`：获取当前所在 module。
- `erase_from_parent`：从父 function 中删除这个基本块。

#### 常用接口

- `get_pre_basic_blocks`
- `get_succ_basic_blocks`
- `remove_pre_basic_block`
- `remove_succ_basic_block`
- `add_pre_basic_block`
- `add_succ_basic_block`
- `is_terminated`
- `get_terminator`
- `get_instructions`
- `remove_instr`
- `erase_instr`

> 注意区分 remove_instr 和 erase_instr 两个接口，在撰写优化 pass 的时候会有所使用。

### Instruction

- 概念：指令。该类是所有 LLVM 指令的基类。子类包含 IR 部分中所有指令。

??? info "Instruction 定义"

    ```c++
    class Instruction : public User, public llvm::ilist_node<Instruction> {
      public:
        enum OpID : uint32_t {
            // Terminator Instructions
            ret,
            br,
            // Standard binary operators
            add,
            sub,
            mul,
            sdiv,
            // float binary operators
            fadd,
            fsub,
            fmul,
            fdiv,
            // Memory operators
            alloca,
            load,
            store,
            // Int compare operators
            ge,
            gt,
            le,
            lt,
            eq,
            ne,
            // Float compare operators
            fge,
            fgt,
            fle,
            flt,
            feq,
            fne,
            // Other operators
            phi,
            call,
            getelementptr,
            zext, // zero extend
            fptosi,
            sitofp
            // float binary operators Logical operators

        };
        /* @parent: if parent!=nullptr, auto insert to bb
        * @ty: result type */
        Instruction(Type *ty, OpID id, BasicBlock *parent = nullptr);
        Instruction(const Instruction &) = delete;
        virtual ~Instruction() = default;

        BasicBlock *get_parent() { return parent_; }
        const BasicBlock *get_parent() const { return parent_; }
        void set_parent(BasicBlock *parent) { this->parent_ = parent; }

        // Return the function this instruction belongs to.
        Function *get_function();
        Module *get_module();

        OpID get_instr_type() const { return op_id_; }
        std::string get_instr_op_name() const;

        bool is_void() {
            return ((op_id_ == ret) || (op_id_ == br) || (op_id_ == store) ||
                    (op_id_ == call && this->get_type()->is_void_type()));
        }

        bool is_phi() const { return op_id_ == phi; }
        bool is_store() const { return op_id_ == store; }
        bool is_alloca() const { return op_id_ == alloca; }
        bool is_ret() const { return op_id_ == ret; }
        bool is_load() const { return op_id_ == load; }
        bool is_br() const { return op_id_ == br; }

        bool is_add() const { return op_id_ == add; }
        bool is_sub() const { return op_id_ == sub; }
        bool is_mul() const { return op_id_ == mul; }
        bool is_div() const { return op_id_ == sdiv; }

        bool is_fadd() const { return op_id_ == fadd; }
        bool is_fsub() const { return op_id_ == fsub; }
        bool is_fmul() const { return op_id_ == fmul; }
        bool is_fdiv() const { return op_id_ == fdiv; }
        bool is_fp2si() const { return op_id_ == fptosi; }
        bool is_si2fp() const { return op_id_ == sitofp; }

        bool is_cmp() const { return ge <= op_id_ and op_id_ <= ne; }
        bool is_fcmp() const { return fge <= op_id_ and op_id_ <= fne; }

        bool is_call() const { return op_id_ == call; }
        bool is_gep() const { return op_id_ == getelementptr; }
        bool is_zext() const { return op_id_ == zext; }

        bool isBinary() const {
            return (is_add() || is_sub() || is_mul() || is_div() || is_fadd() ||
                    is_fsub() || is_fmul() || is_fdiv()) &&
                  (get_num_operand() == 2);
        }

        bool isTerminator() const { return is_br() || is_ret(); }

      private:
        OpID op_id_;
        BasicBlock *parent_;
    };
    ```

#### 子类

- IBinaryInst：add、sub、mul、sdiv
- FBinaryInst：fadd、fsub、fmul、fdiv
- ICmpInst：ge、gt、le、lt、eq、ne
- FCmpInst：fge、fgt、fle、flt、feq、fne
- CallInst：call
- BranchInst：br（包括有条件跳转和无条件跳转）
- ReturnInst：ret
- GetElementPtrInst：getelementptr
- StoreInst：store
- LoadInst：load
- AllocInst：alloca
- ZextInst：zext
- FpToSiInst：fptosi
- SiToFpInst：sitofp
- PhiInst：phi

#### 属性

- `op_id_`：指令类型。
- `parent_`：父基本块。

#### 接口

- `get_parent`：获取其父基本块。
- `set_parent`：设置其父基本块。
- `set_parent_null`：将其父基本块设空。
- `get_function`：获取这条指令属于哪个函数块。
- `get_module`：获取这个指令属于哪个 module。
- `get_instr_type`：指令类型。
- `get_instr_op_name`：指令名。

- 一些指令判断的接口 诸如 `is_phi`。

### Constant

- 概念：常量。不同类型的常量由不同类来表示。

??? info "Constant 及相关类 定义"

    ```cpp
    class Constant : public User {
      private:
        // int value;
      public:
        Constant(Type *ty, const std::string &name = "") : User(ty, name) {}
        ~Constant() = default;
    };

    class ConstantInt : public Constant {
      private:
        int value_;
        ConstantInt(Type *ty, int val) : Constant(ty, ""), value_(val) {}

      public:
        int get_value() { return value_; }
        static ConstantInt *get(int val, Module *m);
        static ConstantInt *get(bool val, Module *m);
        virtual std::string print() override;
    };

    class ConstantArray : public Constant {
      private:
        std::vector<Constant *> const_array;

        ConstantArray(ArrayType *ty, const std::vector<Constant *> &val);

      public:
        ~ConstantArray() = default;

        Constant *get_element_value(int index);

        unsigned get_size_of_array() { return const_array.size(); }

        static ConstantArray *get(ArrayType *ty,
                                  const std::vector<Constant *> &val);

        virtual std::string print() override;
    };

    class ConstantZero : public Constant {
      private:
        ConstantZero(Type *ty) : Constant(ty, "") {}

      public:
        static ConstantZero *get(Type *ty, Module *m);
        virtual std::string print() override;
    };

    class ConstantFP : public Constant {
      private:
        float val_;
        ConstantFP(Type *ty, float val) : Constant(ty, ""), val_(val) {}

      public:
        static ConstantFP *get(float val, Module *m);
        float get_value() { return val_; }
        virtual std::string print() override;
    };

    ```

#### ConstantInt

##### 属性

- `value_`：常量的 int 型数值。

##### 接口

- `get_value`：返回该常量中存储的数。
- `static ConstantInt *get(int val, Module *m);`：以值 val 来创建常量。
- `static ConstantInt *get(bool val, Module *m);`：以值 val 来创建 bool 常量。

#### ConstantFP

##### 属性

- `val_`：常量的 float 型数值。

##### 接口

- `static ConstantFP *get(float val, Module *m);`：以值 val 创建并返回浮点数常量。
- `get_value`：返回该常量中存的值。

#### ConstantZero

##### 接口

- `static ConstantZero *get(Type *ty, Module *m);`：返回并创建 ty 类型的 零常量。

#### ConstantArray

##### 属性

- `const_array`：常量数组的值，是一个 vector 容器。

##### 接口

- `get_element_value`：获取数组中第 i 个值。
- `get_size_of_array`：获取数组元素个数。

### IRBuilder

- 概念：生成 IR 的辅助类。该类提供了独立的接口创建各种 IR 指令，并将它们插入基本块中（注意：该辅助类不做任何类型检查）。

??? info "IRBuilder 定义"

    ```cpp
    class IRBuilder {
      private:
        BasicBlock *BB_;
        Module *m_;

      public:
        IRBuilder(BasicBlock *bb, Module *m) : BB_(bb), m_(m){};
        ~IRBuilder() = default;
        Module *get_module() { return m_; }
        BasicBlock *get_insert_block() { return this->BB_; }
        void set_insert_point(BasicBlock *bb) {
            this->BB_ = bb;
        } // 在某个基本块中插入指令
        IBinaryInst *create_iadd(Value *lhs, Value *rhs) {
            return IBinaryInst::create_add(lhs, rhs, this->BB_);
        } // 创建加法指令（以及其他算术指令）
        IBinaryInst *create_isub(Value *lhs, Value *rhs) {
            return IBinaryInst::create_sub(lhs, rhs, this->BB_);
        }
        IBinaryInst *create_imul(Value *lhs, Value *rhs) {
            return IBinaryInst::create_mul(lhs, rhs, this->BB_);
        }
        IBinaryInst *create_isdiv(Value *lhs, Value *rhs) {
            return IBinaryInst::create_sdiv(lhs, rhs, this->BB_);
        }

        ICmpInst *create_icmp_eq(Value *lhs, Value *rhs) {
            return ICmpInst::create_eq(lhs, rhs, this->BB_);
        }
        ICmpInst *create_icmp_ne(Value *lhs, Value *rhs) {
            return ICmpInst::create_ne(lhs, rhs, this->BB_);
        }
        ICmpInst *create_icmp_gt(Value *lhs, Value *rhs) {
            return ICmpInst::create_gt(lhs, rhs, this->BB_);
        }
        ICmpInst *create_icmp_ge(Value *lhs, Value *rhs) {
            return ICmpInst::create_ge(lhs, rhs, this->BB_);
        }
        ICmpInst *create_icmp_lt(Value *lhs, Value *rhs) {
            return ICmpInst::create_lt(lhs, rhs, this->BB_);
        }
        ICmpInst *create_icmp_le(Value *lhs, Value *rhs) {
            return ICmpInst::create_le(lhs, rhs, this->BB_);
        }

        CallInst *create_call(Value *func, std::vector<Value *> args) {
            return CallInst::create_call(static_cast<Function *>(func), args,
                                        this->BB_);
        }

        BranchInst *create_br(BasicBlock *if_true) {
            return BranchInst::create_br(if_true, this->BB_);
        }
        BranchInst *create_cond_br(Value *cond, BasicBlock *if_true,
                                  BasicBlock *if_false) {
            return BranchInst::create_cond_br(cond, if_true, if_false, this->BB_);
        }

        ReturnInst *create_ret(Value *val) {
            return ReturnInst::create_ret(val, this->BB_);
        }
        ReturnInst *create_void_ret() {
            return ReturnInst::create_void_ret(this->BB_);
        }

        GetElementPtrInst *create_gep(Value *ptr, std::vector<Value *> idxs) {
            return GetElementPtrInst::create_gep(ptr, idxs, this->BB_);
        }

        StoreInst *create_store(Value *val, Value *ptr) {
            return StoreInst::create_store(val, ptr, this->BB_);
        }
        LoadInst *create_load(Value *ptr) {
            assert(ptr->get_type()->is_pointer_type() &&
                  "ptr must be pointer type");
            return LoadInst::create_load(ptr, this->BB_);
        }

        AllocaInst *create_alloca(Type *ty) {
            return AllocaInst::create_alloca(ty, this->BB_);
        }
        ZextInst *create_zext(Value *val, Type *ty) {
            return ZextInst::create_zext(val, ty, this->BB_);
        }

        SiToFpInst *create_sitofp(Value *val, Type *ty) {
            return SiToFpInst::create_sitofp(val, this->BB_);
        }
        FpToSiInst *create_fptosi(Value *val, Type *ty) {
            return FpToSiInst::create_fptosi(val, ty, this->BB_);
        }

        FCmpInst *create_fcmp_ne(Value *lhs, Value *rhs) {
            return FCmpInst::create_fne(lhs, rhs, this->BB_);
        }
        FCmpInst *create_fcmp_lt(Value *lhs, Value *rhs) {
            return FCmpInst::create_flt(lhs, rhs, this->BB_);
        }
        FCmpInst *create_fcmp_le(Value *lhs, Value *rhs) {
            return FCmpInst::create_fle(lhs, rhs, this->BB_);
        }
        FCmpInst *create_fcmp_ge(Value *lhs, Value *rhs) {
            return FCmpInst::create_fge(lhs, rhs, this->BB_);
        }
        FCmpInst *create_fcmp_gt(Value *lhs, Value *rhs) {
            return FCmpInst::create_fgt(lhs, rhs, this->BB_);
        }
        FCmpInst *create_fcmp_eq(Value *lhs, Value *rhs) {
            return FCmpInst::create_feq(lhs, rhs, this->BB_);
        }

        FBinaryInst *create_fadd(Value *lhs, Value *rhs) {
            return FBinaryInst::create_fadd(lhs, rhs, this->BB_);
        }
        FBinaryInst *create_fsub(Value *lhs, Value *rhs) {
            return FBinaryInst::create_fsub(lhs, rhs, this->BB_);
        }
        FBinaryInst *create_fmul(Value *lhs, Value *rhs) {
            return FBinaryInst::create_fmul(lhs, rhs, this->BB_);
        }
        FBinaryInst *create_fdiv(Value *lhs, Value *rhs) {
            return FBinaryInst::create_fdiv(lhs, rhs, this->BB_);
        }
    };
    ```

#### 属性

- `BB_`：当前基本块，指创建指令的基本块。
- `m_`：当前 module。

#### 接口

- `get_module`：获取当前 module。
- `get_insert_block`：获取当前待插入指令的 基本块。
- `set_insert_point`：把当前待插入指令的基本块改为 bb。
- `create_xxx`：创建 xxx 指令。

### Type

- 概念：IR 的类型（包含 `VoidType`、`LabelType`、`FloatType`、`IntegerType`、`ArrayType`、`PointerType`）。module 中可以通过 API 获得基本类型，并创建自定义类型。
- 子类介绍：其中 `ArrayType`、`PointerType` 可以嵌套得到自定义类型，而 `VoidType`、`IntegerType`，`FloatType` 可看做 IR 的基本类型，`LabelType` 是 `BasicBlcok` 的类型，可作为跳转指令的参数，`FunctionType` 表示函数类型。其中 `VoidType` 与 `LabelType` 没有对应的子类，通过 `Type` 中的 `tid_` 字段判别，而其他类型均有对应子类。

??? info "Type 及其相关类定义 "

    ```c++
    class Type {
      public:
        enum TypeID {
            VoidTyID,     // Void
            LabelTyID,    // Labels, e.g., BasicBlock
            IntegerTyID,  // Integers, include 32 bits and 1 bit
            FunctionTyID, // Functions
            ArrayTyID,    // Arrays
            PointerTyID,  // Pointer
            FloatTyID     // float
        };

        explicit Type(TypeID tid, Module *m);
        ~Type() = default;

        TypeID get_type_id() const { return tid_; }

        bool is_void_type() const { return get_type_id() == VoidTyID; }
        bool is_label_type() const { return get_type_id() == LabelTyID; }
        bool is_integer_type() const { return get_type_id() == IntegerTyID; }
        bool is_function_type() const { return get_type_id() == FunctionTyID; }
        bool is_array_type() const { return get_type_id() == ArrayTyID; }
        bool is_pointer_type() const { return get_type_id() == PointerTyID; }
        bool is_float_type() const { return get_type_id() == FloatTyID; }
        bool is_int32_type() const;
        bool is_int1_type() const;

        // Return related data member if is the required type, else throw error
        Type *get_pointer_element_type() const;
        Type *get_array_element_type() const;

        Module *get_module() const { return m_; }
        unsigned get_size() const;

        std::string print() const;

      private:
        TypeID tid_;
        Module *m_;
    };

    class IntegerType : public Type {
      public:
        explicit IntegerType(unsigned num_bits, Module *m);

        unsigned get_num_bits() const;

      private:
        unsigned num_bits_;
    };

    class FunctionType : public Type {
      public:
        FunctionType(Type *result, std::vector<Type *> params);

        static bool is_valid_return_type(Type *ty);
        static bool is_valid_argument_type(Type *ty);

        static FunctionType *get(Type *result, std::vector<Type *> params);

        unsigned get_num_of_args() const;

        Type *get_param_type(unsigned i) const;
        std::vector<Type *>::iterator param_begin() { return args_.begin(); }
        std::vector<Type *>::iterator param_end() { return args_.end(); }
        Type *get_return_type() const;

      private:
        Type *result_;
        std::vector<Type *> args_;
    };

    class ArrayType : public Type {
      public:
        ArrayType(Type *contained, unsigned num_elements);

        static bool is_valid_element_type(Type *ty);

        static ArrayType *get(Type *contained, unsigned num_elements);

        Type *get_element_type() const { return contained_; }
        unsigned get_num_of_elements() const { return num_elements_; }

      private:
        Type *contained_;       // The element type of the array.
        unsigned num_elements_; // Number of elements in the array.
    };

    class PointerType : public Type {
      public:
        PointerType(Type *contained);
        Type *get_element_type() const { return contained_; }

        static PointerType *get(Type *contained);

      private:
        Type *contained_; // The element type of the ptr.
    };

    class FloatType : public Type {
      public:
        FloatType(Module *m);
        static FloatType *get(Module *m);

      private:
    };
    ```

#### 子类

- IntegerType：整数类型。
- FunctionType：函数类型。
  - `is_valid_return_type`：ty 类型是否是合法的返回值类型。
  - `is_valid_argument_type`：ty 类型是否是合法的参数类型。
  - `get_num_of_args`：参数值个数。
  - `get_param_type`：第 i 个参数的类型。
  - `get_return_type`：获取返回值类型。
- ArrayType：数组类型。
  - `is_valid_element_type`：ty 类型是否是合法的数组变量类型。
  - `get_element_type`：获取数组变量的类型。
  - `get_num_of_elements`：获取数组元素个数。
- PointerType：指针类型。
  - `get_element_type`：获取指针指向变量的类型。
- FloatType：浮点数类型。

#### 接口

- 各种获取类型和判断类型的函数，可以查看定义。

### User && Use

#### User

- 概念：使用者。维护了 use-def 信息，表示使用者用了哪些值。

??? info "User 定义"

    ```cpp
    class User : public Value {
      public:
        User(Type *ty, const std::string &name = "") : Value(ty, name){};
        virtual ~User() { remove_all_operands(); }

        const std::vector<Value *> &get_operands() const { return operands_; }
        unsigned get_num_operand() const { return operands_.size(); }

        // start from 0
        Value *get_operand(unsigned i) const { return operands_.at(i); };
        // start from 0
        void set_operand(unsigned i, Value *v);
        void add_operand(Value *v);

        void remove_all_operands();
        void remove_operand(unsigned i);

      private:
        std::vector<Value *> operands_; // operands of this value
    };
    ```

##### 属性

由于`User`是继承自 `Value`的子类，所以本质上也是一个 Value，只是额外增加了一个作用域表示其使用过的所有 value。

- `operands_`：表示这个 user 一共使用了哪些 value。
- `type_`：Value 的类型。
- `use_list_`：使用这个值的所有 user。
- `name_`：value 的名字。

##### 接口

- 初始化：用一个 value 或者 type+name 都可以实例化一个 User。
- 析构：用 `remove_all_operands`删除所有 value 的使用。
- `get_operands`：获得使用的所有 value 列表。
- `get_num_operand`：使用了多少 value。
- `get_operand(i)`：获得第 i 个使用的 value（从 0 开始）。
- `set_operand`：设置第 i 个使用的 value 为新的 `Value*` v。
- `add_operand`：将某个 value 添加到 user 的使用列表。
- `remove_all_operands`：删除所有的使用。
- `remove_operand`：删除第 i 个使用的 value。

#### Use

- 概念：代表了值的使用情况。

```c++
/* For example: op = func(a, b)
 *  for a: Use(op, 0)
 *  for b: Use(op, 1)
 */
```

##### 属性

- `val_`：使用者。
- `arg_no_`：使用者所在的操作数。

### Value

- 概念：值。代表一个可能用于指令操作数的带类型数据，是最基础的类，维护了 def-use 信息，即该值被哪些使用者使用。一个 BB，普通的变量，函数都可以是一个 Value。

??? info "Value 定义"

    ```cpp
    class Value {
      public:
        explicit Value(Type *ty, const std::string &name = "")
            : type_(ty), name_(name){};
        virtual ~Value() { replace_all_use_with(nullptr); }

        std::string get_name() const { return name_; };
        Type *get_type() const { return type_; }
        const std::list<Use> &get_use_list() const { return use_list_; }

        bool set_name(std::string name);

        void add_use(User *user, unsigned arg_no);
        void remove_use(User *user, unsigned arg_no);

        void replace_all_use_with(Value *new_val);
        void replace_use_with_if(Value *new_val, std::function<bool(Use *)> pred);

        virtual std::string print() = 0;

        template<typename T>
        T *as()
        {
          static_assert(std::is_base_of<Value, T>::value, "T must be a subclass of Value");
          const auto ptr = dynamic_cast<T*>(this);
          assert(ptr && "dynamic_cast failed");
          return ptr;
        }
        template<typename T>
        [[nodiscard]] const T* as() const {
            static_assert(std::is_base_of<Value, T>::value, "T must be a subclass of Value");
            const auto ptr = dynamic_cast<const T*>(this);
            assert(ptr);
            return ptr;
        }
        // is 接口
        template <typename T>
        [[nodiscard]] bool is() const {
            static_assert(std::is_base_of<Value, T>::value, "T must be a subclass of Value");
            return dynamic_cast<const T*>(this);
        }

      private:
        Type *type_;
        std::list<Use> use_list_; // who use this value
        std::string name_;        // should we put name field here ?
    };
    ```

#### 属性

- `type_`：Value 的类型。
- `use_list_`：使用这个值的所有 user。
- `name_`：value 的名字。

#### 接口

- 初始化：需要 **类型** 和 **名字**。

- 析构：用 nullptr 替换所有的 使用过这个值的位置。

- `get_name`：获取 `name_`。

- `get_type`：获取 `type_`。

- `set_name`：设置 value 的 `name`。

  返回值表示是否设置成功，默认对已经有 name 的 value 不可修改。

- `add_use`：添加该值的使用情况。

  给 user value 添加 user 引用，arg_no 表示是第几个参数。

- `remove_use`：删除该值的使用情况。

  给 user value 删除 user 引用，`arg_no`表示是第几个参数。

- `replace_all_use_with`：

  用`new_val`替换当前 value 的所有引用。

- `replace_use_with_if`：

  用 `new_val`替换当前 value 的所有引用，但是只替换满足添加的引用。

> 注意 Value 并没有对 == operator 进行重构，因此默认比较只有两个 Value 对象的内存地址相同才会认为是相等的。

- `as`：将某个 Value 类型的对象转换成其派生类类型的对象。
- `is`：判断某个基类是 Value 类型的对象是否是某派生类。

#### 常用接口

- `replace_all_use_with`
- `replace_use_with_if`
- `as`：在优化 pass 撰写 实验中可能用到。
- `is`：在优化 pass 撰写 实验中可能用到。
