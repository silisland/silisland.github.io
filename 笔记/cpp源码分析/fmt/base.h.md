

# Namespace macros
```c++
//line 391
// Namespace macros.
// 说明：使用 `FMT_BEGIN_NAMESPACE`/`FMT_END_NAMESPACE` 封装库到 `fmt::v12`内联命名空间中，以便支持 ABI 兼容（不同主版本可以并存）。
#ifndef FMT_BEGIN_NAMESPACE
#  define FMT_BEGIN_NAMESPACE \
  namespace fmt {           \
  inline namespace v12 {
#  define FMT_END_NAMESPACE \
  }                       \
  }
#endif

//line - 456
FMT_BEGIN_NAMESPACE


//line - 3290
FMT_END_NAMESPACE

```

当代码中使用`FMT_BEGIN_NAMESPACE`时，它会被预处理器替换成：
```cpp
namespace fmt {
inline namespace v12 {
```

而 `define FMT_END_NAMESPACE `被替换成
```cpp
  }                       \
  }
#endif
```





```cpp
// Metafunction aliases and small utilities.
// 说明：为了减少对某些标准头或较新特性的直接依赖（例如 C++14/17
// 中引入的别名模板），这里定义一组轻量别名：`enable_if_t`，
// `conditional_t`，`bool_constant` 等。它们使代码可读性更好并且在
// 老编译器上保持兼容。
// 这些别名仅依赖于 `<type_traits>`，是库内部广泛使用的基础设施。
// Implementations of enable_if_t and other metafunctions for older systems.
// line - 466
template <bool B, typename T = void>
using enable_if_t = typename std::enable_if<B, T>::type;
template <bool B, typename T, typename F>
using conditional_t = typename std::conditional<B, T, F>::type;
template <bool B> using bool_constant = std::integral_constant<bool, B>;
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
template <typename T> using decay_t = typename std::decay<T>::type;
using nullptr_t = decltype(nullptr);
```

# C++ 模板元编程基础知识整理

## 1. 模板别名 (Template Aliases)

### 什么是模板别名
模板别名使用 `using` 关键字为复杂的模板类型定义一个简单的别名。

### 示例代码
```cpp
template <bool B, typename T = void>
using enable_if_t = typename std::enable_if<B, T>::type;
```

上面的代码定义了一个模板别名 `enable_if_t`，它等价于 `typename std::enable_if<B, T>::type`。

## 2. SFINAE 和 std::enable_if

### 作用
- SFINAE (Substitution Failure Is Not An Error)：替换失败不是错误
- `std::enable_if` 用于条件性地启用模板函数

### 用法示例
```cpp
template <bool B, typename T = void>
using enable_if_t = typename std::enable_if<B, T>::type;

// 当条件 B 为 true 时，T 类型有效
template <typename T>
typename std::enable_if<std::is_integral<T>::value, void>::type
process(T value) {
    // 只有当 T 是整型时才可用
    cout << "Processing integral: " << value << endl;
}
```

## 3. 类型特征 (Type Traits)

### 常见的类型特征
| 模板别名 | 等价写法 | 作用 |
|---------|---------|------|
| `enable_if_t<B, T>` | `typename std::enable_if<B, T>::type` | 条件启用模板 |
| `conditional_t<B, T, F>` | `typename std::conditional<B, T, F>::type` | 三元条件选择类型 |
| `remove_reference_t<T>` | `typename std::remove_reference<T>::type` | 移除引用 |
| `remove_const_t<T>` | `typename std::remove_const<T>::type` | 移除const修饰 |
| `decay_t<T>` | `typename std::decay<T>::type` | 类型退化(去除引用、cv限定符等) |
| `underlying_t<T>` | `typename std::underlying_type<T>::type` | 获取枚举底层类型 |

## 4. typename 关键字

### 作用
在模板中告诉编译器某个依赖名称是一个类型而不是值。

### 示例
```cpp
template <typename T>
typename std::enable_if<std::is_integral<T>::value, void>::type
func(T value) {
    // typename 告诉编译器 std::enable_if<...>::type 是一个类型
}
```

## 5. 模板模板参数

### 示例
```cpp
template <typename T>
using remove_cvref_t = typename std::remove_cv<remove_reference_t<T>>::type;
```
- `remove_cvref_t` 是一个模板别名
- 它接收一个类型参数 `T`
- 然后应用 `remove_reference_t<T>` 得到无引用类型
- 再应用 `std::remove_cv<...>` 去除 const/volatile 限定符

## 6. std::conditional

### 作用
根据布尔值选择类型，相当于类型的三元运算符。

### 示例
```cpp
template <bool B, typename T, typename F>
using conditional_t = typename std::conditional<B, T, F>::type;

// 如果 B 为 true，则 conditional_t<B, T, F> 等价于 T
// 如果 B 为 false，则 conditional_t<B, T, F> 等价于 F
using IntOrFloat = conditional_t<true, int, float>; // 等价于 int
```

## 7. std::remove_reference

### 作用
移除类型的引用部分。

### 示例
```cpp
template <typename T>
using remove_reference_t = typename std::remove_reference<T>::type;

// remove_reference_t<int&> 等价于 int
// remove_reference_t<const int&> 等价于 const int
// remove_reference_t<int> 等价于 int
```

## 8. std::remove_const

### 作用
移除类型的 const 限定符。

### 示例
```cpp
template <typename T>
using remove_const_t = typename std::remove_const<T>::type;

// remove_const_t<const int> 等价于 int
// remove_const_t<int> 等价于 int
```

## 9. std::decay (类型退化)

### 作用
将类型转换为其"退化"形式，通常用于模拟函数参数推导规则。

### 示例
```cpp
template <typename T>
using decay_t = typename std::decay<T>::type;

// decay_t<int&> 等价于 int
// decay_t<const int&> 等价于 int
// decay_t<int[]> 等价于 int*
// decay_t<int(int)> 等价于 int(*)(int)
```

## 10. std::underlying_type

### 作用
获取枚举类型的底层整数类型。

### 示例
```cpp
enum Color: int { Red, Green, Blue };
using underlying_t<Color> = int; // 获取 Color 的底层类型 int
```

## 11. 为什么需要这些模板别名？

### 目的
1. **简化代码**：将复杂的模板表达式简化为易读的形式
2. **提高兼容性**：在旧版C++中提供C++14/17的特性
3. **提高可维护性**：集中管理复杂类型表达式

### 对比示例
```cpp
// 不使用别名的复杂写法
typename std::enable_if<std::is_integral<T>::value, void>::type func1(T val);

// 使用别名的简洁写法
enable_if_t<std::is_integral<T>::value, void> func2(T val);
```

这些知识是现代C++元编程的基础，特别在编写库代码时经常使用，如这里的fmt库就大量使用了这些技术来提供灵活且高效的API。










