# C++ 模板元编程详解 - 基于 fmt 库

> 本文档详细讲解 fmt 库中使用的模板元编程技术，通过丰富的示例帮助理解这些概念。

## 目录

0. [[#0. 模板参数基础]]
1. [[#1. 模板别名 (Template Aliases)]]
2. [[#2. SFINAE 和 std::enable_if]]
3. [[#3. 类型特征 (Type Traits)]]
4. [[#4. typename 关键字详解]]
5. [[#5. 条件类型选择]]
6. [[#6. 类型修饰操作]]
7. [[#7. 实际应用场景]]

---

## 0. 模板参数基础

### 0.1 模板参数概述

模板参数是模板定义中的占位符，用于在编译期传递类型或值信息。C++ 支持多种类型的模板参数。

### 0.2 类型模板参数 `template <typename T>`

`typename` 关键字用于声明一个类型模板参数，表示该参数是一个类型而不是值。

#### 0.2.1 基础语法

```cpp
// 基础类型模板
template <typename T>
class MyClass {
public:
    T value;
    void process(T param) {
        // 使用类型 T
    }
};

// 使用示例
MyClass<int> intObj;      // T = int
MyClass<double> doubleObj;  // T = double
MyClass<std::string> strObj; // T = std::string
```

#### 0.2.2 函数模板

```cpp
template <typename T>
T add(T a, T b) {
    return a + b;
}

// 编译器会根据参数类型生成不同的函数版本
int result1 = add(3, 5);           // T = int
double result2 = add(3.5, 2.1);    // T = double
std::string result3 = add(std::string("Hello "), std::string("World")); // T = std::string
```

#### 0.2.3 多个类型参数

```cpp
template <typename T, typename U>
class Pair {
public:
    T first;
    U second;
    
    Pair(T f, U s) : first(f), second(s) {}
};

// 使用
Pair<int, double> p1(42, 3.14);
Pair<std::string, bool> p2("hello", true);
```

#### 0.2.4 默认类型参数

```cpp
template <typename T = int>
class Container {
public:
    T value;
    
    Container(T v = T()) : value(v) {}
};

// 使用
Container<> c1;              // 使用默认类型 int
Container<int> c2;            // 显式指定 int
Container<double> c3(3.14);    // 显式指定 double
```

### 0.3 非类型模板参数 `template <bool B>`

除了类型参数，模板还可以接受非类型参数，如整数、布尔值、指针等。

#### 0.3.1 布尔模板参数

```cpp
template <bool EnableDebug>
class DebugLogger {
public:
    void log(const std::string& message) {
        if constexpr (EnableDebug) {
            std::cout << "[DEBUG] " << message << std::endl;
        }
    }
};

// 使用
DebugLogger<true> debugLogger;    // 启用调试
DebugLogger<false> releaseLogger;  // 禁用调试

debugLogger.log("This will be printed");    // 输出
releaseLogger.log("This won't be printed"); // 不输出
```

#### 0.3.2 整数模板参数

```cpp
template <size_t Size>
class FixedArray {
public:
    int data[Size];
    
    constexpr size_t size() const { return Size; }
    
    int& operator[](size_t index) { return data[index]; }
};

// 使用
FixedArray<5> arr1;  // 大小为 5 的数组
FixedArray<10> arr2; // 大小为 10 的数组

static_assert(arr1.size() == 5, "Size mismatch");
static_assert(arr2.size() == 10, "Size mismatch");
```

#### 0.3.3 组合使用类型和非类型参数

```cpp
template <typename T, size_t N>
class TypedArray {
public:
    T data[N];
    
    constexpr size_t size() const { return N; }
    
    T& operator[](size_t index) { return data[index]; }
};

// 使用
TypedArray<int, 5> intArr;      // 5 个 int 的数组
TypedArray<double, 3> doubleArr; // 3 个 double 的数组
```

### 0.4 混合模板参数 `template <bool B, typename T = void>`

这是 fmt 库中常用的模式，结合了布尔类型参数和默认类型参数。

#### 0.4.1 基础示例

```cpp
template <bool B, typename T = void>
struct ConditionalType {
    // 如果 B 为 true，type 存在且为 T
    // 如果 B 为 false，type 不存在
    using type = T;
};

// 使用
using TrueType = ConditionalType<true, int>::type;  // TrueType = type = int
using FalseType = ConditionalType<false, int>; // 编译错误：没有 type 成员
```

#### 0.4.2 fmt 库中的 enable_if_t

```cpp
// fmt 库中 enable_if_t 的定义
template <bool B, typename T = void>
using enable_if_t = typename std::enable_if<B, T>::type;

// 等价于：
// 当 B = true 时：
//   using enable_if_t = typename std::enable_if<true, T>::type;
//   std::enable_if<true, T>::type = T
//   所以 enable_if_t = T
//
// 当 B = false 时：
//   using enable_if_t = typename std::enable_if<false, T>::type;
//   std::enable_if<false, T> 没有 type 成员
//   SFINAE 生效，使用 enable_if_t 的地方会编译失败

// 实际应用
template <typename T>
enable_if_t<std::is_integral<T>::value, std::string>
format_integral(T value) {
    return std::to_string(value);
}

// 当 T = int 时：
//   std::is_integral<int>::value = true
//   enable_if_t<true, std::string> = std::string
//   函数签名：std::string format_integral(int value)
//
// 当 T = double 时：
//   std::is_integral<double>::value = false
//   enable_if_t<false, std::string> 不存在
//   SFINAE 生效，该函数被排除
```

#### 0.4.3 实际应用：条件类型定义

```cpp
template <bool IsSigned, typename T = int>
using signed_or_unsigned = conditional_t<IsSigned, T, std::make_unsigned_t<T>>;

// 使用
using SignedInt = signed_or_unsigned<true, int>;    // int
using UnsignedInt = signed_or_unsigned<false, int>;  // unsigned int

// 更复杂的例子
template <bool UsePointer, typename T = int>
using storage_type = conditional_t<
    UsePointer,
    T*,
    T
>;

storage_type<true, int> ptrStorage;    // int*
storage_type<false, int> valueStorage; // int
```

#### 0.4.4 默认参数的作用

```cpp
template <bool B, typename T = void>
using my_enable_if = typename std::enable_if<B, T>::type;

// 情况 1：只提供布尔参数，使用默认类型 void
template <typename T>
my_enable_if<std::is_integral<T>::value>
process_integral(T value) {
    // 返回类型是 void
    std::cout << "Processing: " << value << std::endl;
}

// 情况 2：提供布尔参数和自定义类型
template <typename T>
my_enable_if<std::is_integral<T>::value, std::string>
to_string(T value) {
    // 返回类型是 std::string
    return std::to_string(value);
}

// 使用
process_integral(42);        // 正常工作，返回 void
std::string s = to_string(42); // 正常工作，返回 std::string
```

### 0.5 模板参数包 `template <typename... Args>`

C++11 引入了可变参数模板，允许接受任意数量的模板参数。

```cpp
#include <iostream>

// 可变参数模板函数
template <typename... Args>
void print_all(Args... args) {
    (std::cout << ... << args) << ... << std::endl;
}

// 使用
print_all(1, 2.5, "hello", 'a');  // 输出：1 2.5 hello a

// 可变参数模板类
template <typename... Types>
class TypeList {
public:
    static constexpr size_t size = sizeof...(Types);
    
    template <size_t Index>
    using type = typename std::tuple_element<Index, std::tuple<Types...>>::type;
};

// 使用
using MyTypes = TypeList<int, double, std::string>;
static_assert(MyTypes::size == 3, "Size mismatch");

using FirstType = MyTypes::type<0>; // int
using SecondType = MyTypes::type<1>; // double
using ThirdType = MyTypes::type<2>; // std::string
```

### 0.6 模板参数推导

#### 0.6.1 函数模板参数推导

```cpp
template <typename T>
void func(T param) {
    std::cout << "Type: " << typeid(T).name() << std::endl;
}

int x = 42;
func(x);           // T 推导为 int
func(42);          // T 推导为 int
func(3.14);        // T 推导为 double
func("hello");     // T 推导为 const char*
```

#### 0.6.2 完美转发

```cpp
template <typename T>
void wrapper(T&& arg) {
    // 使用 std::forward 完美转发
    process(std::forward<T>(arg));
}

int x = 42;
wrapper(x);       // T = int&, arg 是左值引用
wrapper(42);      // T = int, arg 是右值引用
wrapper(std::move(x)); // T = int, arg 是右值引用
```

### 0.7 模板特化

```cpp
// 主模板
template <typename T>
struct TypeInfo {
    static constexpr const char* name = "Unknown";
};

// 特化版本：针对 int
template <>
struct TypeInfo<int> {
    static constexpr const char* name = "int";
};

// 特化版本：针对 double
template <>
struct TypeInfo<double> {
    static constexpr const char* name = "double";
};

// 特化版本：针对指针类型
template <typename T>
struct TypeInfo<T*> {
    static constexpr const char* name = "Pointer";
};

// 使用
std::cout << TypeInfo<int>::name << std::endl;           // 输出：int
std::cout << TypeInfo<double>::name << std::endl;       // 输出：double
std::cout << TypeInfo<std::string>::name << std::endl;   // 输出：Unknown
std::cout << TypeInfo<int*>::name << std::endl;         // 输出：Pointer
```

---

## 1. 模板别名 (Template Aliases)

### 1.1 基本概念

模板别名使用 `using` 关键字为复杂的模板类型定义一个简单的别名，使代码更加简洁易读。

### 1.2 基础示例

```cpp
#include <type_traits>
#include <iostream>

// 定义模板别名
template <bool B, typename T = void>
using enable_if_t = typename std::enable_if<B, T>::type;

// 使用前（复杂写法）
template <typename T>
typename std::enable_if<std::is_integral<T>::value, void>::type
process_old(T value) {
    std::cout << "Processing integral: " << value << std::endl;
}

// 使用后（简洁写法）
template <typename T>
enable_if_t<std::is_integral<T>::value, void>
process_new(T value) {
    std::cout << "Processing integral: " << value << std::endl;
}

int main() {
    process_old(42);    // 正常工作
    process_new(42);    // 正常工作
    // process_old(3.14); // 编译错误：double 不是整型
    // process_new(3.14); // 编译错误：double 不是整型
    return 0;
}
```

### 1.3 fmt 库中的实际应用

```cpp
// fmt 库中定义的模板别名（base.h 第 466 行）
namespace fmt {

template <bool B, typename T = void>
using enable_if_t = typename std::enable_if<B, T>::type;

template <bool B, typename T, typename F>
using conditional_t = typename std::conditional<B, T, F>::type;

template <bool B> 
using bool_constant = std::integral_constant<bool, B>;

template <typename T>
using remove_reference_t = typename std::remove_reference<T>::type;

template <typename T>
using remove_const_t = typename std::remove_const<T>::type;

template <typename T>
using remove_cvref_t = typename std::remove_cv<remove_reference_t<T>>::type;

template <typename T>
using make_unsigned_t = typename std::make_unsigned<T>::type;

template <typename T>
using underlying_t = typename std::underlying_type<T>::type;

template <typename T> 
using decay_t = typename std::decay<T>::type;

using nullptr_t = decltype(nullptr);

} // namespace fmt
```

### 1.4 为什么需要模板别名？

> [!tip] 模板别名的优势
> 1. **简化代码**：减少重复的 `typename` 和 `::type`
> 2. **提高可读性**：让代码意图更加清晰
> 3. **便于维护**：修改类型时只需改一处
> 4. **兼容性**：在旧编译器上提供新标准特性

```cpp
// 对比示例：不使用别名 vs 使用别名

// ❌ 不使用别名 - 冗长且难以阅读
template <typename T>
typename std::enable_if<
    std::is_integral<typename std::remove_cv<
        typename std::remove_reference<T>::type
    >::type>::value,
    typename std::make_unsigned<T>::type
>::type
func_old(T value) {
    return static_cast<typename std::make_unsigned<T>::type>(value);
}

// ✅ 使用别名 - 简洁明了
template <typename T>
enable_if_t<
    std::is_integral<remove_cvref_t<T>>::value,
    make_unsigned_t<T>
>
func_new(T value) {
    return static_cast<make_unsigned_t<T>>(value);
}
```

---

## 2. SFINAE 和 std::enable_if

### 2.1 SFINAE 原理

SFINAE (Substitution Failure Is Not An Error) 是 C++ 模板的重要特性：**模板参数替换失败不是错误，而是简单地排除该重载**。

### 2.2 基础示例

```cpp
#include <type_traits>
#include <iostream>
#include <string>

// SFINAE 基础示例
template <typename T>
typename std::enable_if<std::is_integral<T>::value>::type
print_type(T value) {
    std::cout << "Integral type: " << value << std::endl;
}

template <typename T>
typename std::enable_if<std::is_floating_point<T>::value>::type
print_type(T value) {
    std::cout << "Floating point type: " << value << std::endl;
}

template <typename T>
typename std::enable_if<std::is_same<T, std::string>::value>::type
print_type(T value) {
    std::cout << "String type: " << value << std::endl;
}

int main() {
    print_type(42);           // 调用第一个版本
    print_type(3.14);         // 调用第二个版本
    print_type(std::string("hello")); // 调用第三个版本
    return 0;
}
```

### 2.3 std::enable_if 工作原理

```cpp
// std::enable_if 的简化实现
template <bool B, typename T = void>
struct enable_if {};

// 特化版本：当 B 为 true 时
template <typename T>
struct enable_if<true, T> {
    using type = T;
};

// 使用示例
template <typename T>
typename enable_if<std::is_integral<T>::value, T>::type
abs(T value) {
    return value < 0 ? -value : value;
}

// 当 T = int 时：
// std::is_integral<int>::value = true
// enable_if<true, int>::type = int
// 函数签名变为：int abs(int value)

// 当 T = double 时：
// std::is_integral<double>::value = false
// enable_if<false, int> 没有 type 成员
// SFINAE 生效，该函数被排除
```

### 2.4 fmt 库中的 SFINAE 应用

```cpp
// fmt 库使用 SFINAE 来选择不同的格式化实现

// 只有当 T 是整型时才启用此函数
template <typename T>
enable_if_t<std::is_integral<T>::value, std::string>
format(T value) {
    return format_integer(value);
}

// 只有当 T 是浮点型时才启用此函数
template <typename T>
enable_if_t<std::is_floating_point<T>::value, std::string>
format(T value) {
    return format_float(value);
}

// 只有当 T 是字符串类型时才启用此函数
template <typename T>
enable_if_t<std::is_same<T, std::string>::value || 
            std::is_same<T, const char*>::value, std::string>
format(T value) {
    return std::string(value);
}
```

### 2.5 SFINAE 进阶示例

```cpp
#include <type_traits>
#include <iostream>
#include <vector>

// 检测类型是否有 size() 方法
template <typename T, typename = void>
struct has_size : std::false_type {};

template <typename T>
struct has_size<T, std::void_t<decltype(std::declval<T>().size())>> 
    : std::true_type {};

// 根据是否有 size() 方法选择不同实现
template <typename T>
enable_if_t<has_size<T>::value, size_t>
get_size(const T& container) {
    return container.size();
}

template <typename T>
enable_if_t<!has_size<T>::value, size_t>
get_size(const T&) {
    return 1; // 单个元素返回 1
}

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    int num = 42;
    
    std::cout << "Vector size: " << get_size(vec) << std::endl; // 输出 5
    std::cout << "Int size: " << get_size(num) << std::endl;    // 输出 1
    
    return 0;
}
```

---

## 3. 类型特征 (Type Traits)

### 3.1 类型特征概述

类型特征是用于在编译期查询和操作类型属性的模板类。

### 3.2 fmt 库中使用的类型特征

| 模板别名 | 等价写法 | 作用 | 示例 |
|---------|---------|------|------|
| `enable_if_t<B, T>` | `typename std::enable_if<B, T>::type` | 条件启用模板 | `enable_if_t<true, int>` → `int` |
| `conditional_t<B, T, F>` | `typename std::conditional<B, T, F>::type` | 三元条件选择类型 | `conditional_t<true, int, float>` → `int` |
| `remove_reference_t<T>` | `typename std::remove_reference<T>::type` | 移除引用 | `remove_reference_t<int&>` → `int` |
| `remove_const_t<T>` | `typename std::remove_const<T>::type` | 移除 const | `remove_const_t<const int>` → `int` |
| `remove_cvref_t<T>` | `typename std::remove_cv<remove_reference_t<T>>::type` | 移除引用和 cv 限定符 | `remove_cvref_t<const int&>` → `int` |
| `decay_t<T>` | `typename std::decay<T>::type` | 类型退化 | `decay_t<int[5]>` → `int*` |
| `underlying_t<T>` | `typename std::underlying_type<T>::type` | 获取枚举底层类型 | 见下文示例 |

### 3.3 详细示例

#### 3.3.1 remove_reference_t

```cpp
#include <type_traits>
#include <iostream>

template <typename T>
using remove_reference_t = typename std::remove_reference<T>::type;

// 验证类型
static_assert(std::is_same<remove_reference_t<int&>, int>::value, "Error");
static_assert(std::is_same<remove_reference_t<int&&>, int>::value, "Error");
static_assert(std::is_same<remove_reference_t<int>, int>::value, "Error");
static_assert(std::is_same<remove_reference_t<const int&>, const int>::value, "Error");

// 实际应用：实现完美转发的辅助函数
template <typename T>
void process(T&& value) {
    // 移除引用后获取原始类型
    using ValueType = remove_reference_t<T>;
    
    if constexpr (std::is_integral<ValueType>::value) {
        std::cout << "Processing integral: " << value << std::endl;
    } else if constexpr (std::is_floating_point<ValueType>::value) {
        std::cout << "Processing float: " << value << std::endl;
    }
}

int main() {
    int x = 42;
    process(x);      // T = int&, ValueType = int
    process(100);    // T = int, ValueType = int
    process(3.14);   // T = double, ValueType = double
    return 0;
}
```

#### 3.3.2 remove_const_t

```cpp
#include <type_traits>

template <typename T>
using remove_const_t = typename std::remove_const<T>::type;

// 类型验证
static_assert(std::is_same<remove_const_t<const int>, int>::value, "Error");
static_assert(std::is_same<remove_const_t<int>, int>::value, "Error");
static_assert(std::is_same<remove_const_t<const int*>, const int*>::value, "Error");
static_assert(std::is_same<remove_const_t<int* const>, int*>::value, "Error");

// 实际应用：创建可修改的副本
template <typename T>
class Container {
public:
    using value_type = remove_const_t<T>;
    
    void set(const value_type& val) {
        value_ = val;
    }
    
    value_type get() const { return value_; }
    
private:
    value_type value_;
};

int main() {
    Container<const int> c;  // value_type = int
    c.set(42);
    int x = c.get();
    return 0;
}
```

#### 3.3.3 remove_cvref_t

```cpp
#include <type_traits>

template <typename T>
using remove_cvref_t = typename std::remove_cv<
    typename std::remove_reference<T>::type
>::type;

// 类型验证
static_assert(std::is_same<remove_cvref_t<const int&>, int>::value, "Error");
static_assert(std::is_same<remove_cvref_t<volatile int&>, int>::value, "Error");
static_assert(std::is_same<remove_cvref_t<const volatile int&>, int>::value, "Error");
static_assert(std::is_same<remove_cvref_t<int>, int>::value, "Error");

// 实际应用：类型清理
template <typename T>
void analyze_type(T&& param) {
    // 清理类型，获取纯净的类型
    using CleanType = remove_cvref_t<T>;
    
    std::cout << "Original type: " << typeid(T).name() << std::endl;
    std::cout << "Clean type: " << typeid(CleanType).name() << std::endl;
}

int main() {
    const int& x = 42;
    analyze_type(x);  // T = const int&, CleanType = int
    return 0;
}
```

#### 3.3.4 decay_t

```cpp
#include <type_traits>
#include <iostream>

template <typename T>
using decay_t = typename std::decay<T>::type;

// 类型验证
static_assert(std::is_same<decay_t<int&>, int>::value, "Error");
static_assert(std::is_same<decay_t<const int&>, int>::value, "Error");
static_assert(std::is_same<decay_t<int[5]>, int*>::value, "Error");
static_assert(std::is_same<decay_t<int(int)>, int(*)(int)>::value, "Error");

// 实际应用：模拟函数参数推导
template <typename T>
void print_decay_info() {
    using DecayedType = decay_t<T>;
    std::cout << "Original type: " << typeid(T).name() << std::endl;
    std::cout << "Decayed type: " << typeid(DecayedType).name() << std::endl;
}

int main() {
    print_decay_info<const int&>();   // const int& -> int
    print_decay_info<int[10]>();      // int[10] -> int*
    print_decay_info<int(int)>();     // 函数类型 -> 函数指针
    
    return 0;
}
```

#### 3.3.5 underlying_t

```cpp
#include <type_traits>
#include <iostream>

template <typename T>
using underlying_t = typename std::underlying_type<T>::type;

// 定义枚举
enum class Color : int {
    Red = 1,
    Green = 2,
    Blue = 3
};

enum class SmallEnum : uint8_t {
    A = 0,
    B = 1
};

// 实际应用：枚举与整数转换
template <typename E>
constexpr underlying_t<E> to_underlying(E e) noexcept {
    return static_cast<underlying_t<E>>(e);
}

int main() {
    Color c = Color::Green;
    
    // 获取底层类型
    using ColorBase = underlying_t<Color>;
    static_assert(std::is_same<ColorBase, int>::value, "Error");
    
    // 转换为底层整数
    int value = to_underlying(c);
    std::cout << "Color value: " << value << std::endl;  // 输出 2
    
    // SmallEnum 的底层类型
    using SmallBase = underlying_t<SmallEnum>;
    static_assert(std::is_same<SmallBase, uint8_t>::value, "Error");
    
    return 0;
}
```

---

## 4. typename 关键字详解

### 4.1 typename 的作用

在模板中，`typename` 用于告诉编译器某个依赖名称是一个**类型**而不是**值**。

### 4.2 基础示例

```cpp
#include <vector>
#include <iostream>

template <typename T>
class Container {
public:
    // ❌ 错误：编译器不知道 T::value_type 是类型还是静态变量
    // T::value_type x;  
    
    // ✅ 正确：使用 typename 明确告诉编译器这是类型
    typename T::value_type x;
    
    void print() {
        std::cout << "Type: " << typeid(x).name() << std::endl;
    }
};

int main() {
    Container<std::vector<int>> c;
    c.print();  // x 的类型是 int
    return 0;
}
```

### 4.3 何时需要 typename

> [!warning] 需要使用 typename 的情况
> 当访问**依赖于模板参数**的**嵌套类型**时，必须使用 `typename`。

```cpp
template <typename T>
void func() {
    // 情况 1：访问嵌套类型 - 需要 typename
    typename T::iterator it;
    
    // 情况 2：访问静态成员 - 不需要 typename
    int x = T::static_value;
    
    // 情况 3：访问嵌套类型的成员 - 需要 typename
    typename T::value_type val;
}

// 更复杂的例子
template <typename T>
class Wrapper {
public:
    // 嵌套类型定义
    using value_type = typename T::value_type;
    using iterator = typename T::iterator;
    using const_iterator = typename T::const_iterator;
    
    // 嵌套模板类型
    template <typename U>
    using rebind = typename T::template rebind<U>;
};
```

### 4.4 fmt 库中的 typename 使用

```cpp
// fmt 库中大量使用 typename
namespace fmt {

template <bool B, typename T = void>
using enable_if_t = typename std::enable_if<B, T>::type;
//             ^^^^^^^^ 这里需要 typename

template <typename T>
using remove_reference_t = typename std::remove_reference<T>::type;
//                         ^^^^^^^^ 这里需要 typename

template <typename T>
using remove_cvref_t = typename std::remove_cv<
    typename std::remove_reference<T>::type
//  ^^^^^^^^ 这里需要 typename
>::type;

} // namespace fmt
```

---

## 5. 条件类型选择

### 5.1 conditional_t

`conditional_t` 相当于类型的三元运算符，根据布尔值选择不同的类型。

```cpp
#include <type_traits>
#include <iostream>

template <bool B, typename T, typename F>
using conditional_t = typename std::conditional<B, T, F>::type;

// 基础示例
static_assert(std::is_same<conditional_t<true, int, float>, int>::value, "Error");
static_assert(std::is_same<conditional_t<false, int, float>, float>::value, "Error");

// 实际应用：根据条件选择不同的容器类型
template <bool UseVector>
class Container {
public:
    using data_type = conditional_t<UseVector, 
                                    std::vector<int>, 
                                    std::list<int>>;
    
    void add(int value) {
        data_.push_back(value);
    }
    
    size_t size() const { return data_.size(); }
    
private:
    data_type data_;
};

// 实际应用：根据大小选择整数类型
template <size_t Size>
using int_type = conditional_t<
    Size == 1, int8_t,
    conditional_t<
        Size == 2, int16_t,
        conditional_t<
            Size == 4, int32_t,
            conditional_t<
                Size == 8, int64_t,
                void
            >
        >
    >
>;

static_assert(std::is_same<int_type<1>, int8_t>::value, "Error");
static_assert(std::is_same<int_type<2>, int16_t>::value, "Error");
static_assert(std::is_same<int_type<4>, int32_t>::value, "Error");
static_assert(std::is_same<int_type<8>, int64_t>::value, "Error");

int main() {
    Container<true> vec_container;   // 使用 vector
    Container<false> list_container; // 使用 list
    
    vec_container.add(1);
    list_container.add(2);
    
    return 0;
}
```

### 5.2 bool_constant

```cpp
#include <type_traits>

template <bool B> 
using bool_constant = std::integral_constant<bool, B>;

// 使用示例
using true_type = bool_constant<true>;
using false_type = bool_constant<false>;

// 实际应用：自定义类型特征
template <typename T>
struct is_small_type : bool_constant<(sizeof(T) <= sizeof(void*))> {};

static_assert(is_small_type<int>::value, "int is small");
static_assert(!is_small_type<long double>::value, "long double is large");

// 组合使用
template <typename T>
using small_or_void = conditional_t<is_small_type<T>::value, T, void>;
```

---

## 6. 类型修饰操作

### 6.1 make_unsigned_t

```cpp
#include <type_traits>
#include <iostream>

template <typename T>
using make_unsigned_t = typename std::make_unsigned<T>::type;

// 类型验证
static_assert(std::is_same<make_unsigned_t<int>, unsigned int>::value, "Error");
static_assert(std::is_same<make_unsigned_t<long>, unsigned long>::value, "Error");
static_assert(std::is_same<make_unsigned_t<short>, unsigned short>::value, "Error");

// 实际应用：安全的绝对值函数
template <typename T>
constexpr make_unsigned_t<T> abs(T value) {
    static_assert(std::is_signed<T>::value, "T must be signed");
    return value < 0 ? static_cast<make_unsigned_t<T>>(-value) 
                     : static_cast<make_unsigned_t<T>>(value);
}

int main() {
    int x = -42;
    auto result = abs(x);
    std::cout << "abs(-42) = " << result << std::endl;
    
    // result 的类型是 unsigned int
    static_assert(std::is_same<decltype(result), unsigned int>::value, "Error");
    
    return 0;
}
```

### 6.2 组合使用多个类型修饰

```cpp
#include <type_traits>
#include <iostream>

namespace my_lib {

// 定义所有需要的别名
template <typename T>
using remove_reference_t = typename std::remove_reference<T>::type;

template <typename T>
using remove_const_t = typename std::remove_const<T>::type;

template <typename T>
using remove_cv_t = typename std::remove_cv<T>::type;

template <typename T>
using remove_cvref_t = remove_cv_t<remove_reference_t<T>>;

template <typename T>
using make_unsigned_t = typename std::make_unsigned<T>::type;

// 组合使用：获取无符号的纯净类型
template <typename T>
using clean_unsigned_t = make_unsigned_t<remove_cvref_t<T>>;

} // namespace my_lib

// 测试
int main() {
    using T1 = my_lib::clean_unsigned_t<const int&>;
    static_assert(std::is_same<T1, unsigned int>::value, "Error");
    
    using T2 = my_lib::clean_unsigned_t<volatile long&>;
    static_assert(std::is_same<T2, unsigned long>::value, "Error");
    
    return 0;
}
```

---

## 7. 实际应用场景

### 7.1 fmt 库中的格式化系统

```cpp
// fmt 库使用模板元编程实现高效的格式化

namespace fmt {

// 根据类型选择格式化策略
template <typename T>
using format_type = conditional_t<
    std::is_integral<T>::value,
    integral_formatter<T>,
    conditional_t<
        std::is_floating_point<T>::value,
        float_formatter<T>,
        conditional_t<
            std::is_same<T, std::string>::value,
            string_formatter,
            custom_formatter<T>
        >
    >
>;

// 类型安全的格式化函数
template <typename T>
enable_if_t<std::is_integral<T>::value, std::string>
format(T value) {
    return format_integer(value);
}

template <typename T>
enable_if_t<std::is_floating_point<T>::value, std::string>
format(T value) {
    return format_float(value);
}

} // namespace fmt
```

### 7.2 类型安全的容器

```cpp
#include <type_traits>
#include <vector>

template <typename T>
class safe_container {
public:
    // 确保存储的是值类型
    using value_type = remove_cvref_t<T>;
    
    // 确保迭代器类型正确
    using iterator = typename std::vector<value_type>::iterator;
    using const_iterator = typename std::vector<value_type>::const_iterator;
    
    // 只接受可转换为 value_type 的参数
    template <typename U>
    enable_if_t<std::is_convertible<U, value_type>::value>
    add(U&& value) {
        data_.push_back(std::forward<U>(value));
    }
    
    // 只对整型提供 sum 方法
    template <typename U = value_type>
    enable_if_t<std::is_integral<U>::value, U>
    sum() const {
        U total = 0;
        for (const auto& item : data_) {
            total += item;
        }
        return total;
    }
    
private:
    std::vector<value_type> data_;
};
```

### 7.3 编译期类型检查

```cpp
#include <type_traits>

// 检查类型是否可以被序列化
template <typename T, typename = void>
struct is_serializable : std::false_type {};

template <typename T>
struct is_serializable<T, std::void_t<
    decltype(std::declval<std::ostream&>() << std::declval<T>()),
    decltype(std::declval<std::istream&>() >> std::declval<T&>())
>> : std::true_type {};

// 只对可序列化类型提供 save/load 函数
template <typename T>
enable_if_t<is_serializable<T>::value>
save(const std::string& filename, const T& data) {
    std::ofstream file(filename);
    file << data;
}

template <typename T>
enable_if_t<is_serializable<T>::value>
load(const std::string& filename, T& data) {
    std::ifstream file(filename);
    file >> data;
}
```

---

## 总结

> [!note] 关键要点
> 1. **模板别名**简化了复杂的类型表达式，提高代码可读性
> 2. **SFINAE** 允许在编译期根据类型特征选择不同的实现
> 3. **类型特征**提供了丰富的类型查询和操作工具
> 4. **typename** 用于明确告诉编译器某个名称是类型
> 5. **条件类型选择**允许根据编译期条件选择不同类型
> 6. **类型修饰操作**用于清理和转换类型

这些技术是现代 C++ 库开发的基础，fmt 库正是通过这些技术实现了高效、类型安全的格式化功能。

---

## 参考资料

- [[base.h]] - fmt 库源码
- [C++ Reference - Type Traits](https://en.cppreference.com/w/cpp/header/type_traits)
- [C++ Templates: The Complete Guide](https://www.tmplbook.com/)
