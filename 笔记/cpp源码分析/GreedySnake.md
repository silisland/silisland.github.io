#GreedySnake
#snake



# Note

- struct的默认访问权限是public（公开的），而class的默认访问权限是private（私有的）。





# 函数重载与运算符重载


#函数重载
```cpp
   public:
      void print(int i) {
        cout << "整数为: " << i << endl;
      }
 
      void print(double  f) {
        cout << "浮点数为: " << f << endl;
      }
 
      void print(char c[]) {
        cout << "字符串为: " << c << endl;
      }

```

#运算符重载

## 重载运算符 `==`  `+`   `+=`   `<<`   示例

```cpp
#include <iostream>
using namespace std;

struct Point {
    int x, y;
    bool operator==(const Point& other) const {   
    //成员函数形式：左操作数是隐含的 *this，因此函数参数只需表示右操作数,即传入同为struct结构体的other
        return x == other.x && y == other.y;
    }//函数声明末尾的const关键字表示这是一个常量成员函数,不能放在bool前。
	
    // 重载+运算符（成员函数版本）
    Point operator+(const Point& other) const {
        return Point(x + other.x, y + other.y);
    }
    
    // 重载+=运算符（修改当前对象）
    Point& operator+=(const Point& other) {
        x += other.x;
        y += other.y;
        return *this;  // 返回当前对象的引用
    }
};

// 重载+运算符（非成员函数版本）- 全局函数
Point operator+(const Point& lhs, const Point& rhs) {
    return Point(lhs.x + rhs.x, lhs.y + rhs.y);
}

// 重载<<运算符（用于输出）
ostream& operator<<(ostream& os, const Point& p) {
    os << "(" << p.x << ", " << p.y << ")";//输出示例(1,2)
    return os;
}

int main() {
    Point p1 = {10, 20};  // 创建点1
    Point p2 = {10, 20};  // 创建点2（与p1相同）
    Point p3 = {30, 40};  // 创建点3（与p1不同）
    // 使用重载的==运算符进行比较
    cout << "p1 == p2: " << (p1 == p2) << endl;  // 输出: 1 (true)
    cout << "p1 == p3: " << (p1 == p3) << endl;  // 输出: 0 (false)
//////////////////////////	
    Point p1(1, 2);
    Point p2(3, 4);
    Point p3; 
 
    // 使用成员函数版本的+
    p3 = p1 + p2;
    cout << "p1 + p2 = " << p3 << endl;  // (4, 6)
    
    // 使用+=运算符
    p1 += p2;
    cout << "p1 += p2 后，p1 = " << p1 << endl;  // (4, 6)
    
    // 使用全局函数版本的+
    Point p4 = operator+(p1, p2);  // 等价于 p1 + p2
    cout << "全局函数: p1 + p2 = " << p4 << endl;  // (7, 10)
	  
	// 链式调用
    Point p5 = p1 + p2 + Point(1, 1);
    cout << "链式调用: p1 + p2 + (1,1) = " << p5 << endl;  // (8, 11)
    
    return 0;
}


```

## 混合类型运算
```cpp
// 支持 Point + int（向量缩放）
Point operator+(const Point& p, int scalar) {
    return Point(p.x + scalar, p.y + scalar);
}

// 支持 int + Point
Point operator+(int scalar, const Point& p) {
    return Point(p.x + scalar, p.y + scalar);
}

// 支持 Point * int（向量缩放）
Point operator*(const Point& p, int scalar) {
    return Point(p.x * scalar, p.y * scalar);
}

// 使用示例
Point p(1, 2);
Point p2 = p + 5;      // (6, 7)
Point p3 = 5 + p;      // (6, 7) 
Point p4 = p * 3;      // (3, 6)



```



## 聚合类型
#聚合类型cpp
### 聚合类型 vs 非聚合类型对比

| 特性        | 聚合类型       | 非聚合类型                |
| --------- | ---------- | -------------------- |
| **构造函数**  | 无用户定义构造函数  | 有用户定义构造函数            |
| **访问控制**  | 所有成员public | 有private/protected成员 |
| **继承**    | 无基类        | 有基类                  |
| **虚函数**   | 无虚函数       | 有虚函数                 |
| **初始化方式** | 支持聚合初始化    | 需要构造函数初始化            |

```cpp
示例：
// 这是一个聚合类型
struct Point {
    int x;
    int y;
    double z;
	bool operator==(const Point& other) const {  // C++20中，这个仍然是聚合类型！
        return x == other.x && y == other.y;
    }
};

// 这也是一个聚合类型（C++14后允许类内初始化）
struct Point2 {
    int x = 0;      // C++14允许
    int y = 0;      // C++14允许
    static int count; // 静态成员不影响聚合性
   
};

非聚合类型示例：
// 不是聚合类型（有用户定义构造函数）
struct Point3 {
    int x, y;
    Point3(int x, int y) : x(x), y(y) {} // 用户定义构造函数
};

// 不是聚合类型（有private成员）
struct Point4 {
private:
    int x;
public:
    int y;
};

// 不是聚合类型（有虚函数）
struct Point5 {
    int x, y;
    virtual void print() {} // 虚函数
};

// 不是聚合类型（有基类）
struct Base {};
struct Point6 : Base { // 继承
    int x, y;
};




```

### 优点
聚合类型最大的优势是支持**聚合初始化（Aggregate Initialization）**，这是C++最优雅的初始化语法之一。

简洁性：不需要定义构造函数
灵活性：可以部分初始化
清晰性：初始化语法直观明了
兼容性：与C语言结构体初始化兼容

```cpp
#include <iostream>
#include <string>
using namespace std;

struct Student {
    string name;
    int age;
    double gpa;
};

int main() {
    // ✅ 聚合初始化 - 按成员顺序初始化
    Student s1 = {"张三", 20, 3.8};
    
    // ✅ C++11后可以省略=
    Student s2{"李四", 21, 3.9};
    
    // ✅ 指定成员初始化（C++20支持）
    Student s3{.name = "王五", .age = 22, .gpa = 4.0};
    
    // ✅ 部分初始化（未指定的成员会进行值初始化）
    Student s4{"赵六"}; // age=0, gpa=0.0
    
    cout << s1.name << " " << s1.age << " " << s1.gpa << endl;
    cout << s2.name << " " << s2.age << " " << s2.gpa << endl;
    
    return 0;
}


```

✅ 何时使用聚合类型
数据载体：主要用于存储数据，没有复杂行为
配置结构：需要简单初始化的配置对象
POD类型：需要与C兼容的Plain Old Data
性能敏感：避免构造函数开销
❌ 何时避免聚合类型
需要封装：需要隐藏内部实现细节
需要验证：构造时需要参数验证
需要复杂初始化：需要执行复杂的初始化逻辑
需要多态：需要虚函数和继承


现代C++中的演进
**C++11/14**
严格的聚合类型定义
不能有用户定义的构造函数或成员函数

**C++17**
稍微放宽限制
允许有默认成员初始化器

**C++20**
重大变化：允许有用户定义的构造函数和成员函数
支持指定成员初始化（.member = value语法  、运算符重载）
聚合类型变得更加实用和灵活




# `_kbhit()` 和 `_getch()`

 Windows 平台特有的输入函数，用于检测和获取键盘输入。

## `_kbhit()`

### 功能
- **检查是否有按键被按下**（非阻塞）
- 如果有按键按下，返回非零值
- 如果没有按键按下，返回 0

### 特点
- **非阻塞函数**：立即返回，不会暂停程序执行
- 用于检测是否发生了键盘输入事件

## `_getch()`

### 功能
- **获取按下的键值**（阻塞）
- 返回按下的字符的 ASCII 码值
- 按任意键后返回

### 特点
- **阻塞函数**：等待直到有按键按下才继续执行
- 不需要按回车键确认
- 不会在屏幕上显示输入的字符

```cpp
void input() {
    if (_kbhit()) {           // 检查是否有按键被按下
        char key = _getch();  // 获取按下的键
        switch (key) {
            case 'w': case 'W':  // 根据按键更新方向
                if (dir != DOWN) dir = UP;
                break;
            // ... 其他按键处理
        }
    }
}
```

## 工作流程

1. `_kbhit()` 检测是否有按键被按下
2. 如果有按键，则用 `_getch()` 获取具体按键
3. 根据按键值更新游戏方向
4. 如果没有按键，程序继续运行而不等待

## 平台兼容性

- **Windows 专用**：这两个函数都在 `<conio.h>` 中定义
- **跨平台替代方案**：在 Linux/macOS 上需要使用其他库（如 `ncurses`）实现类似功能

## 与常规输入的区别

- `cin >>` 或 `getchar()`：需要按回车键确认，有缓冲
- `_getch()`：按任意键即响应，无缓冲，适合游戏控制


