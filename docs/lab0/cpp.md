<!--TODO 感觉整个结构怪怪的？好像编排得不是很有逻辑-->

# C++ 常见语法

本课程实验框架使用 C/C++ 语言，因此需要大家了解一些基本的语法特性。以下内容将重点介绍一些常见且重要的 C++ 语法特性，这些特性在实验中将会频繁使用。

??? Tips "关于基本语法"

    我们默认大家已经有了基本的C/C++编程能力，至少在计算机程序设计(CS1001A)要求范围内的语法这里就不再赘述。

## 基础语法

### `std::string`字符串操作

C++ 提供了一个强大的字符串类 `std::string` ，相较于 C 风格的字符串更加易用，支持通过 `+` 拼接，还提供了许多方便的方法：

- `length`: 返回字符串长度。
- `push_back(c)`: 在字符串末尾添加一个字符 `c`。
- `append(str)`: 在字符串末尾添加字符串 `str`。
- `substr`: 取子串。

更多方法请查阅：[https://en.cppreference.com/w/cpp/string](https://en.cppreference.com/w/cpp/string)

### auto

`auto` 关键字可以用于当类型已知时自动推断类型，再类型明确的的情况下使用`auto`可以简化代码，例如：

```cpp
std::vector<std::string> v;
v.push_back("compile");
auto s = v.front(); // 这里 s 就是 std::string 类型
```

## 面向对象编程

### 类与对象

C++ 类是面向对象编程的核心概念之一，可以看作是对 C 语言结构体的扩展。类不仅可以包含数据成员（类似于结构体中的字段），还可以包含函数成员（方法），用于操作这些数据。

#### 类的定义

类是通过 `class` 关键字定义的。一个简单的类通常包含私有数据和公有方法：

```cpp
class MyClass {
private:
    int data_; // 私有成员变量

public:
    // 构造函数
    MyClass(int data) : data_(data) {}

    // 析构函数
    ~MyClass(){}

    // 公有成员函数
    void display() const {
        printf("Data: %d\n", data_);
    }
};
```

#### 访问控制

- **`private`**：私有成员只能被类的成员函数访问。
- **`public`**：公有成员可以在类外部访问。
- **`protected`**：保护成员可以被继承类访问。

#### 构造函数和析构函数

- **构造函数**：用于初始化对象。名称与类名相同，无返回类型。
- **析构函数**：用于清理对象。以 `~` 开头，名称与类名相同。

#### 成员函数

成员函数可以在类中定义，也可以在类外定义。通常使用 `::` 运算符（作用域解析运算符）在类外定义：

??? Info "作用域解析运算符 `::`"

    作用域解析运算符 `::` 在 C++ 中用于指定某个标识符的作用域。以下是几种常见的用法：

    1. **类的成员函数定义**

        用于在类外定义成员函数：

        ```cpp
        class MyClass {
        public:
            void display();
        };

        void MyClass::display() {
            printf("Hello, World!\n");
        }
        ```

    2. **访问类的静态成员**

        用于访问类的静态成员变量或函数，而不需要实例化对象：

        ```cpp
        class MyClass {
        public:
            static int value;
        };

        int MyClass::value = 10;

        int main() {
            printf("%d\n", MyClass::value);
        }
        ```

    3. **命名空间**

        用于访问命名空间中的成员：

        ```cpp
        namespace MyNamespace {
            int value = 20;
        }

        int main() {
            printf("%d\n", MyNamespace::value);
        }
        ```

    4. **全局作用域**

        用于访问全局变量或函数，特别是在局部作用域中存在同名标识符时：

        ```cpp
        int value = 5;

        int main() {
            int value = 10;
            printf("%d\n", ::value); // 输出全局变量 5
        }
        ```

    作用域解析运算符帮助明确标识符的来源，避免命名冲突。

??? Info "初始化成员列表"

    以`MyClass(int data) : data_(data) {}`为例

    - data_(data)：这是成员初始化列表，用于初始化类的数据成员。
    - data_ 是类的成员变量。
    - data_(data) 将参数 data 的值赋给成员变量 data_。

```cpp
void MyClass::display() const {
    printf("Data: %d\n", data_);
}
```

### 继承与多态

继承是面向对象编程中的重要特性，通过继承，子类可以从基类中继承属性和方法，从而实现代码的重用和扩展。此外，通过函数重写和虚函数，C++ 还提供了多态性，允许基类指针或引用调用子类的实现。

#### 基本继承

在 C++ 中，继承的基本语法如下：

```cpp
struct Animal {
  void eat() {
      std::cout << "Eating" << std::endl;
  }
};

struct Cat : public Animal {
  void nyan() {
      std::cout << "Nyan~" << std::endl;
  }
};

struct Dog : public Animal {
  void bark() {
      std::cout << "Bark!" << std::endl;
  }
};
```

??? Info "关于`struct`和`class`"
C++ 中默认`struct`的成员是`public`的，而`class`的成员是`private`的。在这里我们使用`struct`来定义类，是为了方便讲解，实际上在 C++ 中`class`和`struct`的区别仅在于默认的访问权限。

在这个例子中，`Cat` 和 `Dog` 类继承了 `Animal` 类，因此它们自动拥有了 `Animal` 类中的 `eat()` 方法。同时，它们可以定义自己的特有方法，如 `nyan()` 和 `bark()`。

#### 函数重写

函数重写（Override）是指子类可以提供一个与基类中同名、参数列表相同的函数，以替换基类中的实现。当基类的函数被声明为虚函数（`virtual`）时，子类可以选择重写这个函数，从而实现多态性。

```cpp
struct Animal {
  virtual void say() const {
      std::cout << "Animal sound" << std::endl;
  }
};

struct Cat : public Animal {
  void say() const override {  // 重写基类的 say 函数
      std::cout << "I'm a cat" << std::endl;
  }
};

struct Dog : public Animal {
  void say() const override {  // 重写基类的 say 函数
      std::cout << "I'm a dog" << std::endl;
  }
};
```

在这个例子中，`Cat` 和 `Dog` 类都重写了 `Animal` 类中的 `say()` 方法。关键字 `override` 明确标识了这是一个重写的函数，可以帮助编译器检查是否正确地覆盖了基类的虚函数。

#### 虚函数与多态

虚函数使得基类的指针或引用在运行时能够调用子类的实现，这是多态性的核心。基类中的函数被声明为虚函数后，当通过基类指针或引用调用这个函数时，实际执行的是子类的重写版本。

```cpp
int main() {
  Animal* a;
  Cat c;
  Dog d;

  a = &c;
  a->say(); // 输出 "I'm a cat"

  a = &d;
  a->say(); // 输出 "I'm a dog"

  return 0;
}
```

在上述代码中，当 `a` 指向 `Cat` 对象时，调用 `a->say()` 实际上调用的是 `Cat` 类中重写的 `say()` 方法；当 `a` 指向 `Dog` 对象时，调用的是 `Dog` 类中的 `say()` 方法。这就是多态的作用。

#### 类型转换与 `dynamic_cast`

在某些情况下，你可能需要将基类指针或引用转换为子类类型。为了确保这种转换的安全性，可以使用 `dynamic_cast`。

```cpp
Animal* a = new Cat();
Cat* cc = dynamic_cast<Cat*>(a);

if (cc != nullptr) {
    cc->nyan(); // 确保类型转换成功后调用子类方法
} else {
    std::cout << "转换失败" << std::endl;
}
```

这样我们就可以把基类指针又转回对应的子类指针。当然，如果指针不是 `Cat *` 类型的，`dynamic_cast` 将会返回 `nullptr`。（与 C 中统一使用 `NULL` 不同，在 C++ 中，我们用 `nullptr` 表示空指针）

??? Info "失败的`dynamic_cast`"

    对于指针类型的转换,如果转换失败（即无法将基类指针转换为派生类指针），`dynamic_cast` 会返回 `nullptr`。对于引用类型的转换,如果转换失败，`dynamic_cast` 会抛出异常。

在确信类型转换一定成功时，也可以使用 `static_cast` 来完成转换，可以省去运行时的类型检查。此外，`static_cast` 还用于基本类型的转换，例如：

```cpp
float f = 3.14;
int n = static_cast<int>(f);
```

!!! warning

    你应该避免使用 C 风格的强制类型转换，例如 `int n = (int)f`，因为 C++ 风格的类型转换会被编译器检查，而 C 风格的不会。

## 高级特性

### 标准模板库 (STL) 与容器

STL (Standard Template Library)，意为标准模板库，包含了许多常用的数据结构，在我们的实验中你可能会用到：

- `std::vector`：动态数组，支持快速随机访问和自动扩展。
- `std::map`：有序关联容器，使用红黑树实现，键值对按键排序。
- `std::set`：有序集合容器，元素唯一且按排序规则排列。
- `std::unordered_map`：无序关联容器，使用哈希表实现，支持快速查找。
- `std::list`：双向链表，适合频繁的插入和删除操作。

??? Info "关联容器"

    关联容器是容器用于存储键值对(key-value pairs)，并通过键(key)来快速查找对应的值(value)。我们可将其理解为一种映射关系。

这里的 `std` 是 C++ 中的命名空间，可以防止标识符的重复，详见 [维基百科](https://en.wikipedia.org/wiki/Namespace)

同时，这些容器都是模板 [Template](<https://en.wikipedia.org/wiki/Template_(C%2B%2B)>)，需要实例化，例如一个可变长的整形数组应该实例化为 `std::vector<int>`。

??? Info "`<int>`是什么"

    标准模板库中提供了许多常用数据结构。设想这样一种实际情况，开发者完成了一个高效的`stack`库(栈)的开发，这个栈中的元素是`int`类型的，但是还需一个`float`类型的栈。
    再写一个`stack_float`库当然是可以的，但是二者之间大部分的逻辑是相同的，我们如何能复用大段的代码来提高开发效率呢？

    一种方法是使用`typedef`。然而，这种方法有两个缺点：首先，每次修改类型时都需要编辑头文件；其次，在每个程序中只能使用这种技术生成一种类型，即不能让`typedef`同时代表两种不同的类型，因此不能使用这种方法在一个程序中同时定义`float`栈和`double`栈

    在这种情况下，我们可以使用类模版来解决这个问题，我们可以使用`stack<int>`, `stack<float>`来分别定义`int`栈和`float`栈

    其它的容器同理，如`std::vector<int>`, `std::map<int, std::pair<int, float>>`, ...

更多信息可以参考：[https://en.cppreference.com/w/cpp/standard_library](https://en.cppreference.com/w/cpp/standard_library)，我们在这里就不再对各个容器的具体使用进行详细介绍。

### 内存管理与智能指针

C++11 引入了许多智能指针类型来帮助自动内存管理，本实验中用到的有两种，分别是：

1. `std::shared_ptr`: 引用计数智能指针，使用一个共享变量来记录指针管理的对象被引用了几次。当对象引用计数为 0 时，说明当前该对象不再有引用，并且进程也无法再通过其它方式来引用它，也就意味着可以回收内存，这相当于低级的垃圾回收策略。可以用 `std::make_shared` 来创建。
2. `std::unique_ptr`: 表示所有权的智能指针，该指针要求它所管理的对象只能有一次引用，主要用于语义上不允许共享的对象（比如 `llvm::Module`）。当引用计数为 0 时，它也会回收内存。可以用 `std::make_unique` 来创建。

在涉及到内存管理时，应该尽量使用智能指针。

## 常用技术

### for 循环

`for(auto ...)` 语法是 C++11 引入的范围基于 `for` 循环，用于简化遍历容器的代码。

#### 基本语法

```cpp
for (auto element : container) {
    // 使用 element
}
```

- **`auto`**：自动推断循环变量的类型。
- **`element`**：循环变量，用于访问容器中的每个元素。
- **`container`**：要遍历的容器，如数组、`std::vector`、`std::list` 等。

#### 示例

##### 只读遍历

```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};
for (auto number : numbers) {
    std::cout << number << " ";
}
```

##### 引用遍历（修改元素）

```cpp
for (auto &number : numbers) {
    number *= 2; // 修改容器中的元素
}
```

#### 注意

- 使用引用 `&` 可以直接修改容器中的元素。
- 如果不需要修改，直接使用 `auto` 即可。
- 只读遍历创建了容器中元素的一个拷贝，而引用遍历并没有拷贝，而是创建了对原始元素的引用。

### 函数重载

C++ 中的函数可以重载，即可以有同名函数，但是要求它们的形参必须不同。如果想进一步了解，可以阅读[详细规则](https://en.cppreference.com/w/cpp/language/overload_resolution)。下面是函数重载的示例：

```cpp
struct Point {
  int x;
  int y;
};

struct Line {
  Point first;
  Point second;
};

void print(Point p) {
  printf("(%d, %d)", p.x, p.y);
}

void print(Line s) {
  print(s.first); // s.first == Point { ... }
  printf("->");
  print(s.second); // s.second == Point { ... }
}
```

上面的示例定义了两个 `print` 函数，并且它们的参数列表的类型不同。它们实际上是两个不同的函数（并且拥有不同的内部名字），但是 C++ 能够正确的识别函数调用时使用了哪一个定义（前提是你正确使用了这一特性），并且在编译时就会链接上正确的实现。我们可以看到，这种特性非常符合人的直觉，并且没有带来额外开销。
