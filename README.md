# C++ 17

## 头文件
`__has_include("filename")`和`__has_include(<filename>)`预处理器常量用于判断头文件是否存在。
```cpp
#if __has_include(<optional>)
    #include <optional>
#elif __has_include(<experimental/optional>)
    #include <experimental/optional>
#endif
```
## 强类型枚举
```cpp
enum class PieceType : unsigned char {
    King = 1,
    Queen,
    Rook = 10,
    Pawn
};
PieceType piece = PieceType::King;
```
## if 和 switch 初始化器
```cpp
int i = 10;
if (int j = 0; j < i) {
    // do something
}

PieceType getPiece() {
    cout << __func__ << endl;
    return PieceType::King;
}

switch (auto piece = getPiece(); piece) {
    default: break;
}
```
## 类型别名
类型别名和`typedef`相似。
```cpp
using IntPtr = int*;
using StringVector = vector<string>;
using MatchFunction = bool (*)(int, int); // 函数指针别名
  
// 成员函数指针
using PtrToGet = int (Empolyee::*)() const;
PtrToGet methodPtr = &Empolyee::getSalary;
Empolyee empolyee;
(empolyee.*methodPtr)()
```
## 数组
### 指针

1. 指针数组：`int *p[10]`为一个数组，元素为`int`类型的指针
1. 数组指针：`int (*p)[10]`为一个指针，元素为`int[10]`的数组
```cpp
char a[2][3];
char (*p)[3] = a;
```
### 传递数组大小
按引用传递长度已知的数组（只能用于栈中分配的数组）。
```cpp
template<size_t N>
void doubleIntsStack(int (&theArray)[N]) {
    for (size_t i = 0; i < N; i++) {
        theArray[i] *= 2;
    }
}
```
### 获取数组大小
```cpp
int arr[] = {1, 2, 3};
cout << size(arr) << endl;
```
## 结构化绑定
```cpp
array<int, 3> arr = {1, 2, 3};
auto [a, b, c] = arr;

struct Point { float x; float y; };
Point pt = {
    .x = 1.0,
    .y = 2.0
};
auto [x, y] = pt;
```
## 初始化列表
```cpp
int sum(initializer_list<int> lst) {
    int total = 0;
    for (auto val : lst) {
        total += val;
    }
    return total;
}

sum({1, 2, 3, 4, 5});
```
## 字面量
### 原始字符串字面量
```cpp
const char *str1 = R"(Hello World!)";

// 起始位置和结束位置必须相同, 此处为 "-"
const char *str2 = R"-(Hello World!)-";
```
### 标准字面量
| **后缀** | **说明** |
| --- | --- |
| `s` | `string` |
| `sv` | `string_view`，需要引入命名空间，`using namespace std::string_view_literals`或`using namespace std` |
| `h`、`min`、`s`、`ms`、`us`、`ns` | `std::chrono::duration`，时间段，需要引入命名空间，`using namespace std::chrono_literals` |

### 自定义字面量
```cpp
// 数值->字符串
string operator"" _s(long double d) {
    return to_string(d);  // 1.23_s
}

// 字符串->数值
int operator"" _ss(const char *str, size_t len) {
    return stoi(str, nullptr); // "1232"_ss
}
```
## constexpr 常量表达式
```cpp
constexpr int arrsize() { return 32; }
int arr[arrsize()];
int arr1[arrsize() + 1]; // 在编译时运算

class Rect {
public:
    constexpr Rect(size_t width, size_t height) 
        : mWidth(width), mHeight(height) {}
    constexpr size_t area() const { return mWidth * mHeight; }
private:
    size_t mWidth, mHeight;
};
constexpr Rect r(8, 2);
int arr2[r.area()];
```
> 不允许包含跳转，异常处理，未初始化变量，非字面量变量定义，也不允许抛出异常，所有成员都应该用常量表达式初始化（可以调用其他`constexpr`表达式）。

## 字符串
```cpp
auto string str{"Hello World!"};

// c++14 以前与 c_str 一样返回 const char *
const char *str1 = str.data();

// c++17, const 引用返回 const char *, 非 const 引用返回 char *
char *str2 = str.data();
```
### 数值转换
#### to_string
基本类型转换为字符串。
#### to_chars
```cpp
#include <charconv>
#include <string>
#include <cstring>

string str(5, ' ');
to_chars_result rs = to_chars(str.data(), str.data() + str.size(), 1234);
if (rs.ec == errc()) {
    cout << str << endl;
    // rs.ptr 指向写入字符串末尾的下一个字符
} else {
    cout << strerror(int(rs.ec)) << endl;
}
```
#### stoi、stol、stof、stod
函数原型：
```cpp
stoi/stol(const string &str, size_t *idx=0, int base=10);
stof/stod(const string &str, size_t *idx=0)
```
`idx` 输出参数指向第一个未转换的字符的索引。
#### from_chars
### string_view
用来替代`const string &`，避免字符串字面量创建临时对象。
```cpp
string_view extractExtension(string_view fileName) {
    return fileName.substr(fileName.rfind('.'));
}

const char *raw = "test.txt";
size_t length = strlen(raw);

auto view = extractExtension(raw);
string str{view.data()};

// Error: str = view, 不能隐式转换
cout << view << endl;
cout << extractExtension(string(raw)) << endl;
cout << extractExtension(string_view(raw, length)) << endl;
```
## 内存管理
### 运算符重载
```cpp
class Foo {
public:
    void *operator new(size_t size) {
        return ::operator new(size);
    }
    void operator delete(void *p) {
        ::operator delete(p);
    }

    // 带有参数的 new, 如 new ("abc") Foo();
    void *operator new(size_t size, string_view str) {
        return ::operator new(size);
    }

    void *operator new[](size_t) = delete;
    void operator delete[](void *) = delete;

    // new (nothrow) Foo
    void *operator new(size_t size, const nothrow_t &nothrow) noexcept {
        return ::operator new(size, nothrow);
    }
    void operator delete(void *p, const nothrow_t &nothrow) noexcept {
        ::operator delete(p, nothrow);
    }

    void *operator new[](size_t, const nothrow_t &) noexcept = delete;
    void operator delete[](void *, const nothrow_t &) noexcept = delete;
};
```
### 智能指针
#### unique_ptr
```cpp
unique_ptr<int> ptr1{new int};
auto ptr2 = make_unique<int[]>(10); // c++14

// decltype(free) 为函数类型, * 表示此为函数指针
auto ptr3 = unique_ptr<int, decltype(::free)*>((int *)malloc(sizeof(int) * 10), ::free);
```
#### shared_ptr
```cpp
struct Foo {
    Foo(int value) : data{value} {}
    int data;
};

shared_ptr<Foo> ptr1{new Foo(10)}; // 引用计数, 线程安全
shared_ptr<int[]> ptr2{new int[10]}; // c++17 支持数组, 但不能使用 make_shared
auto ptr3 = shared_ptr<int>((int *)malloc(sizeof(int) * 10), ::free);
auto ptr4 = make_shared<Foo>(10);
auto ptr5 = shared_ptr<int>(ptr4, &ptr4->data); // 别名, 引用成员变量, ptr4 和 ptr5 都销毁才会释放对象
```
#### 转换函数

- `static_pointer_cast`
- `dynamic_pointer_cast`
- `const_pointer_cast`
```cpp
shared_ptr<std::exception> ex(new bad_exception);
auto ex1 = dynamic_pointer_cast<bad_exception>(ex);
```
#### weak_ptr
关联一个`shared_ptr`，可用于判断资源是否已经被释放。

1. 调用`lock`返回一个`shared_ptr`，如果资源已被释放, 返回的`shared_ptr`为`nullptr`。
1. 通过`weak_ptr`创建一个`shared_ptr`，如果资源已被释放，抛出`bad_weak_ptr`异常。
#### enable_shared_from_this 混入类
```cpp
class Bar : public enable_shared_from_this<Bar> {};

auto ptr1 = make_shared<Bar>();
auto ptr2 = ptr1->shared_from_this();  // 仅当指针为 shared_ptr 时, 才能使用该方法
auto ptr3 = ptr1->weak_from_this();

auto ptr4 = new Bar();
// Error: auto ptr4 = ptr4->shared_from_this();
```
> _可以使用 Valgrind_ 工具查找并修复内存泄漏。

## 类和对象
### 构造函数
```cpp
class Foo {
public:
      Foo() = default; // c++ 编译器显式提供默认构造函数, 无需实现
      Foo(const Foo &o) = default; // 使用默认拷贝构造
      Foo &operator=(const Foo &o) = default; // 使用默认赋值函数
      Foo(int value) : data{value} {}
      Foo(initializer_list<int> args) {  // 初始化列表构造
          if (args.size() != 1) {
              throw invalid_argument("initializer_list should contain at least one element");
          }
          data = args[0];
      }
      virtual ~Foo() = default; // 可以在继承中使用默认析构函数
      int data = 10; // c++11
};

class Bar {
public:
      Bar() = delete; // c++ 编译器显式删除默认构造函数, 无需实现
      Bar(const Bar &o) = delete;  // 显式删除拷贝构造
      Bar &operator=(const Bar &o) = delete; // 显式删除赋值函数
      Bar(int value) : data{value} {}
      void setData(int value) { data = value; }
      void setData(double value) = delete; // 禁用 double 类型的重载

      int data = 10;
};
```
### 移动语义
右值引用，能够引用临时对象。
```cpp
void handleMessage(string &message) {
  // do something
}
void handleMessage(string &&message) {
  // 此时, 参数 message 为具名参数, 为左值 
}

string a{"Hello"};
string b{"World"};
handleMessage(a);     // 左值
handleMessage(a + b); // 右值, 临时对象
handleMessage(move(b)); // 右值, move 将左值转换为右值
```
#### 移动构造和移动赋值

1. 需要指定`noexcept`限定符，告诉编译器不会抛出异常。
1. `move`函数用于执行移动操作，如果接受方为左值引用，则会调用移动构造或移动赋值。
```cpp
class Foo {
public:
    Foo(Foo &&o) noexcept;
    Foo &operator=(Foo &&o) noexcept;
    int data = 10; // c++11
};
```
### mutable
被`mutable`修饰的数据成员告诉编译器允许在`const`函数中修改，如在访问器中加锁。
### as_const
转换为`const`类型，`as_const(*this)`可以在`const`方法中调用非`const`方法。
### 静态成员
```cpp
static const size_t max_value = 0; // 静态常量数据成员
static inline size_t counter = 0;  // c++17, 静态内联数据成员, 不需要在类外定义
```
### 引用数据成员
```cpp
class Foo {};
class Bar {
public:
    Bar(Foo &foo, const Foo &foo1) : foo(foo), foo1(foo1) {};
    Foo &foo;
    const Foo &foo1;
};
```
### 嵌套类
嵌套类能访问外围类的所有成员，但外围类需要按照访问权限访问嵌套类成员。
```cpp
class Foo {
public:
    class Bar;
};
class Foo::Bar {};
```
### 禁止继承
```cpp
class Foo final {};
class Bar {
public:
    virtual void method() final; // 禁止派生类再次重写
};
```
### 重写方法
```cpp
class Foo1 {
public:
    Foo1() = default;
    Foo1(int value) {}
    Foo1(const char *value) {}
    Foo1(string_view value) {}
    virtual void method() {}
};

class Foo2 {
public:
    Foo2() = default;
    Foo2(const char *value) {} 
};

class Bar : public Foo1, public Foo2 {
public:
    using Foo::Foo;  // 继承构造函数, 多重继承时, 如果基类拥有相同参数的构造函数, 必须要在派生类中进行重新定义
    Bar(double value) {}
    Bar(int value) {}  // 屏蔽基类 int 参数类型的构造
    Bar(const char *value) {} // 避免歧义, 重新定义基类相同参数的构造

    using Foo::method;  // method(int i) 会隐藏基类实现, 使用 using 指令则可以完成重载
    virtual void method(int i) {}
};

Bar bar1(10); // Bar(int)
Bar bar2("a"); // Foo(string_view)
```

1. 基类的指针或引用指向派生对象时，派生类保留其重写方法，但通过类型转换将派生类对象转换为基类对象时，就会丢失其独有特征。
1. 在构造函数中，基类会调用自身的虚方法实现，而不是派生类重写的方法。
1. _C++ _根据表达式的编译时类型绑定默认参数，因此默认参数不具有多态性。
## 特性
| **特性** | **说明** |
| --- | --- |
| `[[noreturn]]` | 函数永远不会返回 |
| `[[deprecated]]` | 用于标记某个对象已被废弃 |
| `[[fallthrough]]` | 用于告诉编译器某个 _case _的 _fall through_ 是有意安排的 |
| `[[nodiscard]]` | 用于有返回值的函数，如果函数返回值未被使用，编译器将会发出警告 |
| `[[maybe_unused]]` | 用于修饰函数参数，当参数未被使用时阻止编译器发出警告 |

```cpp
[[deprecated("Unsafe method, please use xyz")]] void func();

switch (1) {
case 0:
    // do something
    [[fallthrough]]
case 1:
    // do something
    break;
default: break;
}

int func(int p, [[maybe_unused]] int q) { return p; }
```
## lambda
格式：
```cpp
[capture_block](parameters) mutable constexpr 
    noexcept_specifier attributes
    -> return_type { body }
```
```cpp
auto fn1 = [](int a, int b) -> int { return a + b; }
auto fn2 = [](int a, int b){ return a + b; } // 自动推断返回类型
auto fn3 = []{ cout << "hello world!" << endl; } // 无参数时可以省略括号
```
### 变量捕捉
_lambda_ 表达式中的`[]`用于捕捉作用域中的变量，之后在表达式中使用。

| **表达式** | **说明** |
| --- | --- |
| `[=]` | 按值捕捉所有变量 |
| `[&]` | 按引用捕捉所有变量 |
| `[x]` | 按值捕捉 _x _变量 |
| `[&x]` | 按值捕捉 _x _变量 |
| `[=, &x]` | 按引用捕捉 _x _变量，其余变量按值捕捉 |
| `[&, x]` | 按值捕捉 _x _变量，其余变量按引用捕捉 |
| `[this]` | 捕获当前对象，可在表达式中使用 _this _指针 |
| `[*this]` | 捕获当前对象的副本，此时表达式中的 _this _指针是只读的 |

### 捕捉表达式
```cpp
double pi = 3.14;
auto fn1 = [myCap = "Pi: ", pi]{ cout << myCap << pi << endl; };

auto ptr = make_unique<double>(3.14);
auto fn2 = [ptr = move(ptr)]{ cout << *ptr << endl; }; // 允许使用与所在作用域相同的名称
```
### 范型 lambda
可以将_ lambda_ 表达式的参数类型指定为`auto`，由编译器进行推断。
```cpp
auto fn = [](auto i) { return i > 2; }

vector<int> ivec{1, 2, 3};
find_if(cbegin(ivec), cend(ivec), fn);

vector<double> dvec{1.2, 2.4, 3.6};
find_if(cbegin(dvec), cend(dvec), fn);
```
### 参数与返回值
_lambda _表达式作为函数参数或返回值时会被转换为对应的`function`对象。
```cpp
// void 可以省略
function<int(void)> fn = []{ return 10; };

// 使用类型推断
auto exp(int a) {
  return [&a]{ return a * a; };
}
```
## 模版
### 模版推导
可以根据构造函数的实参自动推导模版参数（_C++17_）。

- 智能指针不允许类型推导
- 虚函数和析构函数不允许是模版
### 模版类型转换
```cpp
template<typename T>
class Foo {
public:
    template<E> Foo(const Foo<E> &o) {}
    template<E> Foo<T> &operator=(const Foo<E> &o);
};
template<typename T, typename E> Foo<T> &Foo<T>::operator=(const Foo<E> &o) {}
```
### 模版特化
特化和模版继承不同，继承可以复用基类的模版代码，但特化后的模版仍重写需要的所有代码。
```cpp
template<typename T, typename E> class Foo {};
template<typename T> class Foo<string, T> {};
template<> class Foo<const char *, int> {};
class Bar : public Foo<int, int> {}
```
> 对函数进行特化是不允许的，但是可以重载。

### 模版别名
```cpp
template<typename T, typename E> class Foo {};
using Foo1 = Foo<int, int>;
template<T> using Foo2 = Foo<T, int>; // typedef 无法完成
```
### 类模版的友元函数模版
```cpp
template<typename T> class Foo; // 前置声明
template<typename T> Foo<T> operator+(const Foo<T> &lhs, const Foo<T> &rhs); // 函数原型

template<typename T> class Foo {
public:
    // 类模版和函数实例之间一一对应, 函数后的 <T> 表明该函数本身也是模版
    friend Foo<T> operator+ <T>(const Foo<T> &lhs, const Foo<T> &rhs);
};
```
### 类型推导
`auto`关键字会去除引用和`const`限定符, 因此在需要时要自己添加`const auto &`，`decltype`不会修改类型。
```cpp
const string message = "helle";
const string &getString() {
  return message;
}
auto s1 = getString();  // 去除引用和 const 限定符
s1[0] = 'H';

decltype(auto) s2 = getString();
// Error: s2[0] = 'H';

// 函数返回类型推导
auto add(int a, int b) {  // c++14
    return a + b;
}

// 模版返回类型推导(c++14之前)
template<typename T1, typename T2>
auto add(const T1 &t1, const T2 &t2) -> decltype(t1+t2) {
  return t1 + t2;
}

// c++14
template<typename T1, typename T2>
decltype(auto) add(const T1 &t1, const T2 &t2) {
  return t1 + t2;
}
```
### 可变模版
可以通过不同的模版参数返回不同类型，也可以对其进行特化。
```cpp
template<typename T> constexpr T Pi = T(3.141592657);
float fpi = Pi<float>;
double dpi = Pi<double>;
```
### template template 参数
模板参数列表里面可以存在模板。
```cpp
template<typename T, typename Container> class Grid {
public:
    Container mCells;
};
```
对于上述的模板定义，只能使用`Grid<int, vector<optional<int>>>`来定义对象，必须传入两个`int`。
```cpp
template<typename T, 
template<typename E, typename Allocator = std::allocator<E>> class Container = std::vector> 
class Grid {
public:
    Container<optional<T>> mCells;
};
```
此时可以使用`Grid<int, vector>`或者`Grid<int>`来定义对象。
### 非类型参数
将前面的类型参数的值作为参数，可以是指针、常量、引用，也可以为其指定默认值。
```cpp
template<typename T, T DEFAULT = T()>
class Grid {
public:
    const T val = DEFAULT;
};

Grid<int> grid1; // grid1.val: 0
Grid<int, 10> grid2; // grid2.val: 10
```
### 模板递归
```cpp
template<typename T, size_t N>
class Grid {
public:
    Grid() { eles.resize(N); }
    Grid<T, N-1> &operator[](size_t i) { return eles[i]; }

    vector<Grid<T, N-1>> eles;
};
```
和普通递归一样，必须对模板递归的终点进行特化。
```cpp
template<typename T>
class Grid<T, 1> {
public:
    Grid() { eles.resize(N); }
    T &operator[](size_t i) { return eles[i]; }

    vector<int> eles;
};

Grid<int, 3> grid;
grid[0][1][2] = 5;
```
### 可变参数模板
```cpp
template<typename... Types> class Foo {};

Foo<> foo1;
Foo<int> foo2;
Foo<int, double> foo3;
```
为了避免零个模板参数的情况，一般使用如下的方式编写模板：
```cpp
template<typename T1, typename... Types> class Foo {};
```
#### 类型安全的变长参数列表
```cpp
void handle(int value) {}
void handle(double value) {}
void handle(string_view value) {}

void process() {} // 终止函数
template<typename T1, typename... Tn> void process(T1 arg1, Tn... args) {
    handle(arg1);
    process(args...);
}
```
前面的实现在进行递归调用的时候会对参数进行复制，但如果使用引用参数，则将无法使用字面量参数，为了解决这个问题，可以使用转发引用，`std::forward`可以完美转发所有参数，如果参数是左值则转发左值，如果参数是右值则转发右值。
```cpp
template<typename T1, typename... Tn> void process(T1 &&arg1, Tn&&... args) {
    handle(forward<T1>(arg1));
    // 解开参数包, 在每个参数上调用 forward, 并用逗号分隔
    // 如果有 a1 和 a2 两个参数, 类型分别为 A1 和 A2, 则会转换后为 
    // process(forward<A1>(a1), forward<A2>(a2));
    process(forward<Tn>(args)...);
    // 在使用了参数解包的函数体中, 可以通过如下方式获得参数包中参数的个数
    int numOfArgs = sizeof...(args);
}
```
参数包几乎可以用在任何地方，如下混入类：
```cpp
class Mixin1 {
public: 
    Mixin1(int v) {}
    virtual ~Mixin1() {} 
};
class Mixin2 { 
  Mixin2() = default;
  virtual ~Mixin2() {} 
};

template<typename... Mixins>
class Foo : public Mixins... {
public:
    Foo(const Mixins&... mixins) : Mixins(mixins)... {}; 
    virtual ~Foo() = default;
};

// 将构造函数传入的参数在初始化列表展开(Mixin1(Mixin1(1)), Mixin2(Mixin2())), 会调用 Mixin 的拷贝构造函数
Foo<Mixin1, Mixin2> foo1(Mixin1(1), Mixin2());

// 如果不使用统一初始化, Mixin 构造函数必须带有参数, 否则编译器会将其当做函数指针声明, 导致编译失败
Foo<Mixin2> foo2(Mixin2()); // Error!

Mixin2 mixin2;
Foo<Mixin2> foo3(mixin2);
```
#### 折叠表达式
_C++17_ 加入了折叠表达式的支持，可以更容易的处理参数包。

| **名称** | **表达式** | **含义** |
| --- | --- | --- |
| 一元右折叠 | `(pack op ...)` | `pack_0 op (... op (pack_n-1 op pack_n))` |
| 一元左折叠 | `(... op pack)` | `((pack_0 op pack_1) op ...) op pack_n` |
| 二元右折叠 | `(pack op ... op init)` | `pack_0 op (... op (pack_n-1 op (pack_n op init)))` |
| 二元左折叠 | `(init op ... op pack)` | `(((init op pack_0) op pack_1) op ...) op pack_n` |

_表达式中的 op_ 可以是`+`、`-`、`*`、`/`、`%`、`^`、`&`、`|`、`<<`、`>>`、`+=`、`-=`、`*=`、`/=`、`%=`、`^=`、`&=`、`/=`、`<<=`、`=>>`、`=`、`==`、`<`、`>`、`<=`、`>=`、`&&`、`||`、`,`、`.*`、`->*`。
```cpp
template<typename... Tn>
void process(const Tn... args) {
    // 展开后为 (handle(a1), handle(a2), handle(a3))
    (handle(args), ...);
}

template<typename... Tn>
void print(const Tn... args) {
    ((cout << args << endl), ...);
}

template<typename T, typename... Tn>
int sum(const T &init, const Tn... args) {
    (init + ... + args);
}
```
### 模板元编程
#### 打印元组
```cpp
template<typename TupleType, int n>
class tuple_print_helper {
public:
    tuple_print_helper(const TupleType &t) {
        tuple_print_helper<TupleType, n-1> tp(t);
        cout << get<n-1>(t) << endl;
    }
};

template<typename TupleType, 0>
class tuple_print_helper {
public:
    tuple_print_helper(const TupleType &) {}
};

template<typename T>
void tuple_print(const T& t) {
    tuple_print_helper<T, tuple_size<T>::value> tph(t);
}

auto t = make_tuple(1, 1.5f, "test");
tuple_print(t);
```
_C++17_ 引入了`constexpr if`，在编译时执行`if`语句，如果`if`从未到达，则不会进行编译。
```cpp
template<typename TupleType, int n = tuple_sise<TupleType>::value>
void tuple_print_helper(const TupleType &t) {
    if constexpr(n > 1) {
        tuple_print_helper<TupleType, n-1> tp(t);
    }
    cout << get<n-1>(t) << endl;
}
```
#### 编译时整数序列
| **序列** | **说明** |
| --- | --- |
| `index_sequence` | 辅助用的序列索引 |
| `index_sequence_for` | 生成给定的参数包等长的索引序列 |

```cpp
#include <utility>

template<typename TupleType, size_t... Indices>
void tuple_print_helper(const TupleType &t, index_sequence<Indices...>) {
    (cout << get<Indices>(t) << endl, ...)
}

template<typename... args>
void tuple_print(const tuple<args...> &t) {
    tuple_print_helper(t, index_sequence_for<args...>());
}
```
#### 类型 trait
类型 _trait_ 相关的功能都定义在`<type_traits>`头文件中，用于在编译时根据类型做出决策。

- `is_void`
- `is_reference`
- `is_object`
- `is_same`
- `is_base_of`
- `is_const`
```cpp
if (is_integral<int>::value) {}
if (is_class<string>::value) {}

// 对于取 value 成员的的 trait, C++17 都添加了一个变量模板, 名称以 _v 结尾
if (is_integral_v<int>) {}
```
##### enable_if
第一个参数为`true`时，内部将有一个由第二个参数给定的`type`嵌套类型, 第一个参数为`false`，将没有此类型，可以在需要解析重载歧义的时候使用。
```cpp
template<typename T>
enable_if_t<is_base_of_v<IsDoable, T>, void>
call_doit(const T &t) {
    t.doit();
}
template<typename T>
enable_if_t<!is_base_if_v<IsDoable, T>, void>
call_doit(const T&) {
    cout << "Cannot call doit()!" << endl;
}

Derived d;
call_doit(d); // doit
call_doit(1); // cannot call
```
可以使用`constexpr if`简化。
```cpp
template<typename T>
void call_doit(const T& [[maybe_unused]] t) {
    if constexpr(is_base_of_v<IsDoable, T>) {
        t.doit();
    } else {
        cout << "Cannot call doit()!" << endl;
    }
}
```
##### 逻辑运算符

- `conjunction`: 串联 
- `disjunction`: 分离
- `negation`:  否定
```cpp
cout << conjunction_v<is_integral_v<int>, is_integral_v<short>> << endl;
cout << negation_v<is_integral_v<int>> << endl;
```
## 流
| **流** | **说明** |
| --- | --- |
| `cin` | 标准输入 |
| `cout` | 标准输出，缓冲 |
| `cerr` | 标准错误，无缓冲 |
| `clog` | 带缓冲的`cerr` |

### 通用方法
| **方法** | **说明** |
| --- | --- |
| `good` | 判断流是否处于正常状态 |
| `bad` | 判断流是否发生致命错误 |
| `fail` | 最近一次操作失败，但无法判断下一次操作是否也会失败，相当于`!cout`或者`!cin` |
| `exceptions` | 要求流发生故障时抛出`ios_base::failure`异常，`cout.exceptions(ios::failbit &#124; ios::badbit &#124; ios::eofbit)` |
| `clear` | 重置错误状态 |
| `eof` | 文件末尾 |

### otream
| **方法** | **说明** |
| --- | --- |
| `put` | 输出单个字符 |
| `write` | 输出字符串，如`cout.write("123", 3)` |
| `flush` | 刷新流，将缓冲立即写出 |

```cpp
// 布尔值
cout << true << " " << boolalpha << true << endl;
cout << false << " " <<  noboolalpha << false << endl;

// 进制
cout << hex << 16 << dec << 10 << endl;

// 设置宽度, 填充和对齐(仅对紧跟的输出有效)
#include <iomanip>
cout << setiosflags(ios::left) << setfill('0') << setw(6) << 10 << endl;
```
### istream
| **方法** | **说明** |
| --- | --- |
| `get` | 从流中返回一个字符（可能会返回`eof`） |
| `unget` | 将流回退一个字符 |
| `putback` | 将指定的字符放回流中 |
| `peek` | 从流中查看一个字符 |
| `getline` | 从流中读入一行 |

```cpp
char buf[BUFSIZ] = {0};
cin.getline(buf, BUFSIZ);

string str;
getline(cin, str);
```
#### 操作
用于控制流输入。

| **操作** | **说明** |
| --- | --- |
| `skipws`、`noskipws` | 跳过（默认）或读入空表字符作为标记 |
| `ws` | 跳过流中当前位置的一串空白符 |

### ofstream、ifstream、fstream
#### 模式
使用或运算组合模式，`ostream`默认包含`out`，`istream`默认包含`in`。

| **常量** | **说明** |
| --- | --- |
| `ios_base::app` | 追加 |
| `ios_base::ate` | 打开文件时将文件指针移动到文件尾 |
| `ios_base::binary` | 二进制文件 |
| `ios_base::in` | 读 |
| `ios_base::out` | 写 |
| `ios_base::trunc` | 截断 |

```cpp
#include <fstream>

ofstream oStream("test.txt", ios_base::trunc);
```
#### 移动文件指针
| **方法** | **说明** |
| --- | --- |
| `seekg`、`seekp` | 设置输入或输出文件偏移 |
| `tellg`、`tellp` | 查询输入或输出流当前位置 |

文件位置常量：

| **位置** | **说明** |
| --- | --- |
| `ios_base::beg` | 流开始位置 |
| `ios_base::end` | 流结束位置 |
| `ios_base::cur` | 流当前位置 |

```cpp
oStream2.seekp(ios_base::beg);  // 移动到文件头

oStream1.seekp(0, ios_base::beg);
iStream.seekg(-2, ios_base::end);
```
由于文件是双向_ I/O_，因此在流中需要维护输入和输出两个方向的位置，才会使用不同的函数。
### 流适配器
允许将输入和输出流视为迭代器，通过这些迭代器对输入流和输出流进行适配，将它们应用在标准库算法中。
#### ostream_iterator
参数说明：

1. 输出流
1. 写入每个元素后写入流的分隔字符串
```cpp
vector<int> vec(10);
iota(begin(vec), end(vec), 1);

copy(begin(vec), end(vec), ostream_iterator<int>(cout, " "));
```
#### istream_iterator
```cpp
istream_iterator<int> numbersIter(cin);
istream_iterator<int> endIter;
int sum = accumulate(numbersIter, endIter, 0);
```
## 异常处理
`exception`为标准异常的基类，`what()`函数用于返回异常的描述字符串。
```cpp
class MyException : public exception {
public:
    virtual const char *what() const noexcept override {
        return "my exception";
    }
}

try {
  // do something
  // throw 可以抛出任何类型的异常
  throw MyException();
} catch (const MyException &exp) {
} catch (...) {
}
```
通常会设置终止处理器函数，用于在进程结束前创建崩溃转储，提供给调试器调试。
```cpp
#include <exception>

void myTerminate() {}
set_terminate(myTerminate);
```
### noexcept
可以使用`noexcept`标记此函数不应该抛出异常。
### throw
异常列表用于标识当前函数可能抛出的异常。
```cpp
void func() throw(invalid_argument, runtime_error) {}
```
> _C++11_ 之后，已不建议使用异常规范，_C++17_ 之后, 该规范已被废弃，不再支持。

### new
默认情况下内存分配失败时抛出`bad_allc`异常，可以使用`nothrow`指明失败时返回`nullptr`，而不是抛出异常。
```cpp
auto ptr = new (nothrow) int;
```
#### 自定义内存分配失败行为
`set_new_handle()`用于设置内存分配失败时的处理函数。
```cpp
class terminate_application : public bad_alloc {};
void newHandler() {
    throw terminate_application(); // 处理函数只能抛出 bad_alloc 异常及其派生类异常
}
new_handler old = set_new_handler(newHandler);
```
如果`new_handler`成功返回，则会重新尝试分配内存，在`new_handler`中可以再次设置`handler`。
### 构造函数异常
如果构造函数抛出为处理的异常，该对象的析构函数将不会被调用, 但会调用已经构造完整的对象的析构函数。
### 析构函数异常
析构函数会被隐式的标记为`noexcept`，除非添加`noexcept(false)`，如果在栈回退的过程中，由于析构函数的调用而再次引发异常，此时进程将被强行终止，为防止这种情况，应在所有可能抛出异常的析构函数中使用`uncaught_exception`方法判断当前是否正在进行栈回退（存在一个未捕获的异常），如果存在，则应抑制异常的再次抛出。
### 嵌套异常
可以在抛出的异常中嵌套当前环境中的异常，在处理异常时，可以使用`dynamic_cast<const nested_exception *>(&e)`得到嵌套的异常指针，通过`rethrow_nested`再次抛出异常，捕获后可获得嵌套的异常对象。
```cpp
void doSomething() {
    try {
        throw runtime_error("runtime_error");
    } catch (const runtime_error &e) {
        throw_with_nested(MyException(__func__));
    }
}

try {
    doSomething();
} catch (const MyException &e) {
    cout << "Exception: " << e.what() << endl;
    const auto *pNested = dynamic_cast<const nested_exception *>(&e);
    if (pNested) {
        try {
            pNested->rethrow_nested();
        } catch (const runtime_error &e) {
            cout << "Nested exception: " << e.what() << endl;
        }
    }
}
```
可以使用`rethrow_if_nested`辅助函数重新抛出异常。
```cpp
try {
    doSomething();
} catch (const MyException &e) {
    try {
        rethrow_if_nested(e);
    } catch (const runtime_error &e) {
        cout << "Nested exception: " << e.what() << endl;
    }
}
```
### 重新抛出异常
```cpp
try {
    // do something
} catch (const MyException &e) {
    throw;  // 不要试图使用 throw e;
}
```
## 标准库
### byte 类型
```cpp
#include <cstddef>
byte b{15};
cout << to_integer<int>(b) << endl;
```
### 容器
`size()`和`empty()`可以用于获取容器的大小和判断容器是否为空。
#### 顺序容器
| **方法** | **说明** |
| --- | --- |
| `data` | 获取数据指针，可用于`array`、`vector`、字符串、静态分配的C风格数组（不通过指针访问）和 `initializer_lists` |
| `emplace` | 通过传入的参数构造元素对象并插入 |
| `erase` | 删除迭代器指向的元素，返回被删除元素之后的迭代器 |

##### array
封装了数组，固定长度，提供迭代器支持，可用于算法。
```cpp
array<int, 3> arr{1, 2, 3};
cout << arr.size() << endl;
cout << arr[0] << endl;
```
##### vector
```cpp
class Foo{
public:
    explict Foo(string_view str) {}
};
vector<Foo> vec;
auto &e = vec.emplace_back("abc"); // C++17, 返回插入元素的引用
```
> 标准库对`vector<bool>`进行了特化，但不推荐使用，建议使用`bitset`。

##### deque
##### list
`splice()`方法用于将源容器的元素插入目标容器，插入的元素将会从源容器中删除。
```cpp
list<string> li1{"123", "456", "789"};
list<string> li2;
// 执行完该操作后, li1 将为空
li2.splice(li2.begin(), li1, li1.begin(), li1.end());
```
##### forward_list
#### 关联容器、无序关联容器
##### pair
_C++17_ 可以直接使用`pair(1, 2)`进行参数推导，因此不再需要`make_pair`这类工具函数。
```cpp
// 结合结构化绑定
auto [fisrt, second] = pair(1, 2);
```
##### 关联容器
关联容器是被称为基于节点的数据结构，C++17 可以对节点进行直接访问。
```cpp
map<string, int> map1;
map<string, int> map2;
map2["abc"] = 1;
auto node = map2.extract("abc");
map1.insert(move(node)); // 将 map1 的 abc 节点移动到 map2 上

for (auto &[key, value] : map1) {
    cout << "(" << key << ", " << value << ")" << endl;
}
```
###### map
可以结合`initializer_lists`和结构化绑定插入元素。
```cpp
map<int, int> map;
auto [iter, success] = map.insert({1, 3});
```
| **方法** | **说明** |
| --- | --- |
| `try_emplace` | 元素不存在则插入，如果存在则忽略 |
| `upper_bound`、`lower_bound` | 获取范围 |

`map`是有序的，可以获取指定范围内的迭代器。
```cpp
map<int, int> map{{{1, 1}, {2, 2}, {3, 3}, {4, 4}}};
auto beg = map.lower_bound(2);
auto end = map.upper_bound(3);
// 2, 3
for (auto iter = beg; iter != end; ++iter) {
    cout << iter->second << endl;
}
```
###### multimap
一个键可以对应多个值，因此可以通过`equal_range`获取指定键值的范围内的迭代器，`map`也有该函数，但意义不大。
```cpp
multimap<int, int> map{{{1, 1}, {1, 2}, {1, 3}, {4, 4}}};
auto [beg, end] = map.equal_range(1);
// 1, 2, 3
for (auto iter = beg; iter != end; ++iter) {
    cout << iter->second << endl;
}
```
###### set
###### multiset
##### 无序容器
无序容器的元素需要提供`hash`函数，标准库提供了 _hash _模版，并对大多数类型做了特化，用于计算 _hash _值。
###### unordered_map
| **方法** | **说明** |
| --- | --- |
| `load_factor` | 返回每一个桶的平均元素数 |
| `bucket_cout` | 返回容器中桶的数量 |
| `bucket(key)` | 返回指定键的桶索引 |
| `begin(n)`、`end(n)` | 返回指定索引的桶的迭代器 |
| `local_iterator`、`const_iterator` | 遍历桶中元素的迭代器 |

###### unordered_multimap
###### unordered_set
###### unordered_multiset
#### 容器适配器
##### queue
##### priority_queue
##### stack
#### 分配器
任何提供了`allocate`、`deallocate`和其他需要的方法和类型别名的类都可以替代默认的`allocator`类。
两个采用不同分配器的`vector<int>`是不同的，不能进行赋值操作，_C++17_ 在`<memory_resource>`的`std::pmr`命名空间中引入了多态内存分配器的概念，有助于解决此问题，`std::pmr::polymorphic_allocator`的分配行为取决于构建期间的`memory_resource`，而非取决于模板类型参数，虽然具有相同类型，但行为可以不同，_C++ _也提供了一些内置的内存资源，可供初始化了多态内存分配器。

- `synchronized_pool_resource`
- `unsynchronized_pool_resource`
- `monotonic_buffer_resource`
#### 迭代器
除了使用容器成员函数获取，还可以通过`begin`、`end`、`rbegin`、`rend`、`crbegin`和`crend`来获取迭代器，也可用于 _C _风格的数组。
##### 迭代器适配器
| **迭代器适配器** | **说明** |
| --- | --- |
| `reverse_iterator` | 以反向遍历双向迭代器或随机访问迭代器 |
| `insert_iterator`、`back_insert_iterator`、`front_insert_iterator` | 插入迭代器 |
| `back_inserter` | 返回`back_insert_iterator`的辅助函数 |
| `move_iterator` | 移动迭代器, 可以通过`make_move_iterator`来创建 |

##### iterator_traits
迭代器萃取器。

| **别名** | **说明** |
| --- | --- |
| `value_type` | 元素类型 |
| `difference_type` | 迭代器之间的差值类型 |
| `iterator_category` | 迭代器类型
- `bidirectional_iterator_tag`，双向迭代器
- `input_iterator_tag`
- `output_iterator_tag`
- `forward_iterator_tag`
- `random_access_iterator_tag`，随机访问迭代器
 |
| `pointer` | 元素类型的指针 |
| `reference` | 元素类型的引用 |

```cpp
#include <iterator>

iterator_traits<vector<int>::iterator>::value_type // int
```
#### 自定义标准容器
如果要将自己的容器实现为标准容器，顺序容器必须提供的别名：

- `value_type`
- `reference`或`const_reference`
- `iterator`或`const_iterator`
- `size_type`
- `difference_type`

无序容器必须提供的别名：

- `key_type`：键类型
- `mapped_type`: 值类型
- `value_type`: `pair<const Key, T>`
- `hasher`: 哈希类型
- `key_equal`: 使用的 _equality_ 谓词
- `local_iterator`或`const_local_iterator`: 迭代单个桶使用的迭代器，不能跨桶迭代
- `node_type`: 节点类型
## 算法
如果容器提供了相关算法的特殊实现，应该优先使用容器提供的方法。
### accumulate
元素汇总（不支持并行）。
```cpp
#include <numeric>

vector<int> vec{1, 2, 3, 4};
// 0 为初始值, a 为上次计算结果, b 为当前元素值
accumulate(begin(vec), end(vec), 0, [](int a, int b){return a + b;});
```
### function
可以指向任何可调用的对象，称为多态函数包装器。
```cpp
#include <functional>

void func(int num, const string &str) {}
function<void(int, const string &)> fn = func;
fn(1, "abc");
function<void(int, const string &)> fn1 = [&fn](int num, const string &str) {
  fn(num, str);
};
```
> 也可以使用`auto`关键字，但推断的类型是函数指针`void (*fn)(int, const string &)`，而不是`function`。

### count_if
计算满足特定条件的元素个数。
### generate
生成元素值。
```cpp
vector<int> vec(5);
int value = 0;
generate(begin(vec), end(vec), [&value]{ value += 1; return value; });
// vec: 1, 2, 3, 4, 5
```
### 函数对象

- 算数运算 
   - `plus`
   - `minus`
   - `multiplies`
   - `divides`
   - `modulus`
- 比较运算 
   - `equal_to`
   - `not_equal_to`
   - `less`
   - `greater`
   - `less_equal`
   - `greater_equal`
- 逻辑运算 
   - `logical_not`
   - `logical_and`
```cpp
plus<int> myPlus;
int res = myPlus(1, 2);

vector<int> vec{1, 2, 3, 4};
accumulate(begin(vec), end(vec), 0, plus<int>());

// 透明运算符仿函数允许忽略模版的类型参数, 建议使用这种方式的仿函数
accumulate(begin(vec), end(vec), 0, plus<>());
```
### bind
绑定器用于实现函数的柯里化，即将接受多个参数的函数变换成另一个接受一个或多个参数的函数。
```cpp
int func1(int a, int b, int c, int d) {
  return a + b + c + d;
}

// 将 func1 的参数 b 绑定为 10, 参数 d 绑定为 32, placeholders 为参数的占位符
auto fn1 = bind(func1, placeholders::_1, 10, 32, placeholders::_2);
auto res = fn1(1, 2);

void func2(int a, const string &str) {}
// 交换 func2 的两个参数的调用顺序
auto fn2 = bind(func2, placeholders::_2, placeholders::_1);
fn2("abc", 1);

// 修改参数, 传递引用
void func3(int &a) { a++; }
int i = 1;
auto fn3 = bind(func3, ref(i));
fn3(); // i: 2
```
### not_fn
取反器参数和原函数一致，但将函数的返回值取反。
```cpp
int func(int a, int b) { return a + b; }
auto fn1 = not_if(func);
auto res1 = fn1(1, 1); // 0
auto res2 = fn1(0, 0); // 1
```
### mem_fn
调用成员函数。
```cpp
vector<string> vec{"abc", "", "efg"};
auto iter = find_if(vec.begin(), vec.end(), mem_fn(&string::empty));
```
### invoke
通过一组参数调用任何可调用对象。
```cpp
void func(int a, int b) {}
invoke(func, 1, 2);
invoke([](int a) -> void {}, 10);
```
### 搜索

- `find` 
- `find_if`
- `find_if_not`
- `find_first_of`
- `adjacent_find`
- `search_n`
```cpp
vector<int> vec{5, 6, 9, 8, 8, 3};
vector<int> target{6, 9, 8};

auto it1 = find(vec.begin(), vec.end(), 9); // 9
auto it2 = find_if(vec.begin(), vec.end(), [](auto v) { return v == 9; });  // 9
auto it3 = find_if_not(vec.begin(), vec.end(), [](auto v) { return v != 9; });  // 9
auto it4 = find_first_of(vec.begin(), vec.end(), target.begin(), target.end()); // 5, 查找 vec 中第一个存在于 target 中的元素

auto it5 = adjacent_find(vec.begin(), vec.end()); // 8, 查找 vec 中第一个重复的元素

auto it7 = search_n(vec.begin(), vec.end(), 2, 8); // 8, 查找 vec 中连续出现 2 个 8 的位置
```
#### search
提供了一些额外的选项，允许使用专用的搜索算法，时间复杂度_ O(N+M)_，最坏情况为_O(NM)。_

- `default_searcher` 
- `boyer_moore_searcher`
- `boyer_moore_horspool_searcher`
```cpp
vector<int> vec{5, 6, 9, 8, 8, 3};
vector<int> target{6, 9, 8};

auto it1 = search(vec.begin(), vec.end(), target.begin(), target.end()); // 6, 在 vec 查找 target 子序列(查找子串)
auto it2 = find_end(vec.begin(), vec.end(), target.begin(), target.end()); // 6, 从 vec 尾部开始查找 target 子序列(查找子串)

string text = "this is the haystack to search a needle in.";
string toSearchFor = "needle";
auto searcher = boyer_moore_searcher(toSearchFor.begin(), toSearchFor.end());
auto result = search(text.begin(), text.end(), searcher); // n
```
#### 二分查找

- `lower_bound`、`upper_bound`：针对已排序集合进行搜索，返回小于或大于指定值的第一个元素的迭代器
- `binary_search`：二分查找指定值
### 比较

- `equal`：元素个数相同切每个对应元素都相等 
- `mismatch`：查找第一个不匹配的元素, 返回一个包含迭代器的 `piar`
- `lexicographical_comppare`：扩展字符串匹配规则, 应用于任意对象
```cpp
vector<int> vec{6, 9, 8, 3};
vector<int> target{6, 9, 8};

auto res1 = equal(vec.begin(), vec.begin()+3, target.begin(), target.end()); // true
auto res2 = mismatch(vec.begin(), vec.end(), target.begin(), target.end()); // res2.first: 3
```
### 计数
返回`bool`值，用于标识值在集合中的存在情况。

- `all_of`：集合中所有元素都等于指定的值 
- `any_of`：集合中任意一个元素等于指定的值
- `none_of`：集合中没有一个元素等于指定的值
```cpp
vector<int> vec{1, 1, 0, 1};
auto res = all_of(vec.begin(), vec.end(), [](auto i){ return i == 1; }); // true
```
### 复制

- `copy` 
- `copy_backward`
- `copy_if`
- `copy_n`
- `partition_copy`：将值分别拷贝到两个容器中，参数的回调函数返回`true`拷贝到第一个容器，返回`false`则拷贝到第二个容器
```cpp
vector<int> vec{1, 2, 3, 4};
vector<int> target1(vec.size());
vector<int> target2(vec.size());

// 将 vec 中的元素复制到 target 中
copy(vec.bedin(), vec.end(), target1.begin());

// 将 vec 中前两个元素复制到 target 中
copy_n(vec.begin(), 2, target1.begin());

partition_copy(vec.begin(), vec.end(), target1.begin(), target2.begin(), [](const auto i){ return i % 2 == 0; }); // target: [2, 4], target2: [1, 3]
```
### 移动

- `move` 
- `move_backward`
```cpp
vector<int> vec{1, 2, 3, 4};
vector<int> target(vec.size());

// 将 vec 中前两个元素移动到 target 中, 移动后的元素将从 vec 中移除
move(vec.bedin(), vec.end(), target.begin());
```
### 替换

- `replace` 
- `replace_if`
```cpp
vector<int> vec{1, 2, 3, 4};
vector<int> target(vec.size());

// 将 vec 中的元素 2 替换为 10
replace_if(vec.begin(), vec.end(), [](auto i){ return i == 2; }, 10);
```
### 删除

- `remove_if` 
- `unique`：移除重复元素
### transform
```cpp
vector<int> vec1{1, 2, 3, 4};
vector<int> vec2{5, 6, 7, 8, 9};
vector<int> target(vec.size());

// 将 vec1 中的每个元素加 1, 放入 target 中, taget 也可以是原集合用来转换自身
transform(vec1.begin(), vec1.end(), target.begin(), [](auto i) { return i + 1; );

// 将 vec1 中的每个元素与 vec2 中的元素相加, 放入 target 中
transform(vec1.begin(), vec1.end(), vec2.begin(), target.begin(), [](int i, intj) { return i + j; });
```
### 抽样

- `sample` 
- `shuffle`
```cpp
vector<int> vec{1, 2, 3, 4, 5, 6, 7, 8};
vector<int> target;

random_device seeder;
const auto seed = seeder.entropy() ? seeder() : time(nullptr);
default_random_engine engin(static_cast<default_random_engine::result_type>(seed));
// 从 vec 中随机选择 4 个元素到 target 中
sample(vec.begin(), vec.end(), back_iterator(target), 4, engine);
```
### 遍历

- `for_each` 
- `for_earch_n`：遍历 _n_ 个元素
```cpp
vector<int> vec{1, 2, 3, 4, 5, 6, 7, 8};
for_earch(vec.begin(), vec.end(), [](const auto &i){ cout << i << endl; });
for_earch_n(vec.begin(), 2, [](const auto &i){ cout << i << endl; });
```
### 交换

- `swap`：交换两个变量的值 
- `exchange`：用新值替换旧值
```cpp
#include <utility>

int a = 10, b = 20;
swap(a, b);

a = 10, b = 20;
int c = exchange(a, b); // 相当于 c = a, a = b, 结果为 c: 10, a: 20, b: 20
```
### 排序

- `sort` 
- `merge`：将两个已序集合合并，可以传入一个可选的比较函数
```cpp
vector<int> vec1{6, 4, 2, 1};
vector<int> vec2{5, 3, 1}

sort(vec1.begin(), vec1.end(), greater<>());
merge(vec1.begin(), vec1.end(), vec2.begin(), vec2.end());
```
### 集合算法

- `includes`：包含 
- `set_union`：并集
- `set_intersection`：交集
- `set_diference`：差集
- `set_aymmetric_diference`：对称差集
```cpp
vector<int> vec1{1, 2, 3, 4};
vector<int> vec2{2, 3};
vector<int> target(vec1.size() + vec2.size());

// vec1 是否包含 vec2 中的元素
auto res1 = includes(vec1.begin(), vec1.end(), vec2.begin(), vec2.end());

set_union(vec1.begin(), vec1.end(), vec2.begin(), vec2.end(), target.begin());
```
### 最小、最大值

- `max` 
- `min`
- `minmax`：返回一个 `pair`
- `clamp(v, lo, hi)`：若 _v_ 小于 _lo_，则返回 _lo_；若 _v_ 大于 _hi_，则返回 _hi_；否则返回 _v_，可以传入一个可选的比较函数
```cpp
auto imax1 = max(1, 2);
auto imax2 = max({3, 7, 4, 2});
```
### 并行
_C++17 _中针对标准库的大部分算法, 包括`for_each`、`all_of`、`find`、`search`、`transform`等都添加了并行执行以提高性能。

| 策略 | 全局实例 | 描述 |
| --- | --- | --- |
| `sequenced_policy` | `execution::seq` | 不允许并行执行 |
| `parallel_policy` | `execution::par` | 允许并行执行 |
| `parallel_unsequenced_policy` | `execution::par_unseq` | 允许并行执行和矢量化，还允许线程间的迁移执行 |

```cpp
#include <execution>

vector<int> vec{5, 3, 7, 5, 8, 1};

sort(execution::par, vec.begin(), vec.end());
for_each(execution::par, vec.begin(), vec.end(), [](const auto i) {
    cout << i << endl;
  });
```
### 数值算法

- `inner_product`：内积（不支持并行）
- `iota`：生产序列值
- `gcd`/`lcm`：返回两个数的最大公约数/最小公倍数
- `reduce`：汇总函数
```cpp
#include <numeric>

vector<int> vec1{5, 3, 7, 5, 8, 1};
vector<int> vec2{5, 3, 7, 5, 8, 1};
vector<int> vec3(5);

// 0 为初始值
auto res1 = inner_product(vec1.begin(), vec1.end(), vec2.begin(), 0);

// 以 5 为初始值, 生成序列
iota(vec3.begin(), vec3.end(), 5); // [5, 6, 7, 8, 9]

auto res2 = gcd(4, 6); // 2

// 求和
auto res3 = reduce(execution::par_unseq, vec1.begin(), vec1.end(), 0);
```
## 正则表达式
类型模板说明：

| **类型** | **说明** |
| --- | --- |
| `basc_regex` | 正则表达式对象 |
| `match_results` | 捕捉组，`sub_match`的集合
- `prefix()`、`suffix()`：返回为匹配子串之前或之后的字符 |
| `sub_match` | 包含输入序列的一个迭代器对象<br>&nbsp;&nbsp;&#9670&nbsp;`str()`返回匹配到的字符串 |
| `regex_iterator` | 遍历一个模式在源字符串中出现的所有位置 |
| `regex_token_iterator` | 遍历一个模式在源字符串中出现的所有捕捉组 |

上述模板均提供了类型别名。
```cpp
using regex = basic_regex<char>;
using wregex = basic_regex<wchar_t>;

using csub_match = basic_regex<const char *>;
using wcsub_match = basic_regex<const wchar_t *>;

using cmatch = match_results<const char *>;
using wcmatch = match_results<const wchar_t *>;
using smatch = match_results<string::const_iterator>;
using wsmatch = match_results<wstring::const_iterator>;

using cregex_iterator = regex_iterator<const char *>;
using wcregex_iterator = regex_iterator<const wchar_t *>;
using sregex_iterator = regex_iterator<string::const_iterator>;
using wsregex_iterator = regex_iterator<wstring::const_iterator>;

using cregex_token_iterator = regex_token_iterator<const char *>;
using wcregex_token_iterator = regex_token_iterator<const wchar_t *>;
using sregex_token_iterator = regex_token_iterator<string::const_iterator>;
using wsregex_token_iterator = regex_token_iterator<wstring::const_iterator>;
```
### regex_match
正则表达式需要 **完整匹配** 源字符串才会返回`true`，该函数提供了多种形式的重载，函数原型如下：
```cpp
bool regex_match(input [, match_results], regex [, flags])
```
```cpp
regex r(R"((\d{4})/(0?[1-9]|1[0-2])/(0?[1-9]|[1-2][0-9]|3[0-1]))");
  
if (regex_match("1991/1/1-abc", r)) { // false
    cout << "found" << endl;
}

// match_results 类型要和源字符串类型对应
string str = "1991/1/1";
smatch m;
if (regex_match(str, m, r)) {
    cout << stoi(m[1]) << "-" << stoi(m[2]) << "-" << stoi(m[3]) << endl;
}
```
### regex_search
在源字符串中搜索匹配特定模式的字字符串，如果找到则返回`true`，函数原型如下：
```cpp
bool regex_search(input [, match_results], regex [, flags])
```
> 不要在循环中通过操作迭代器的方式调用`regex_search`来搜索所有匹配模式的子串，可能出现空匹配导致无限循环，应该使用`regex_iterator`或`regex_token_iterator`。

```cpp
regex r(R"((\d{4})/(0?[1-9]|1[0-2])/(0?[1-9]|[1-2][0-9]|3[0-1]))");
  
if (regex_search("1991/1/1-abc", r)) {  // true
    cout << "found" << endl;
}

// match_results 类型要和源字符串类型对应
string str = "1991/1/1";
smatch m;
if (regex_search(str, m, r)) {
    cout << stoi(m[1]) << "-" << stoi(m[2]) << "-" << stoi(m[3]) << endl;
}
```
### regex_iterator
遍历所有的匹配。
```cpp
string str("this is a simple text");
regex reg(R"([\w]+)");

const sregex_iterator end;
for (sregex_iterator iter(str.begin(), str.end(), reg); iter != end; ++iter) {
    cout << (*iter)[0] << endl;
}
```
> 由于内部包含了一个指向正则表达式的指针，因此不能传入`regex` 的右值。

### regex_token_iterator
用于遍历指定索引的捕捉组，索引参数可以为如下值：

| **索引** | **说明** |
| --- | --- |
| _无_ | 遍历索引为 _0_ 的捕捉组 |
| _n_ | 遍历第 _n _个捕捉组 |
| `initializer_list`、`vector` | 遍历指定索引的捕捉组 |
| _-1_ | 用于激活执行使用模式分割字段和标记的任务 |

```cpp
regex reg(R"((\d{1})-(\d{2})-(\d{3}))");
string str("1-02-003");
const sregex_token_iterator end;

// 遍历索引为 0 的捕捉组, 结果为 1-02-003
for (sregex_token_iterator iter(str.begin(), str.end(), reg); iter != end; ++iter) {
    cout << *iter << endl;
}

// 遍历索引为 2, 3 的捕捉组, 即 (\d{2})-(\d{3}), 结果为 02, 003
vector<int> indices{2, 3};
for (sregex_token_iterator iter(str.begin(), str.end(), reg); iter != end; ++iter) {
    cout << *iter << endl;
}

// 分割字符串, 类似 split
string s("this is a simple text");
regex r(R"(\s+)");
for (sregex_token_iterator iter(s.begin(), s.end(), r, -1); iter != end; ++iter) {
    cout << *iter << endl;
}
```
### regex_replace
替换字符串，可以在替换字符串中引用源字符串中匹配到的子串，格式化字符如下：

| **转义** | **描述** |
| --- | --- |
| _$n_ | 匹配到的第 _n_ 个捕捉组的字符串 |
| _$&_ | 匹配的整个正则表达式字符串 |
| _$`_ | 匹配到的子串左边的文本 |
| _$’_ | 匹配到的子串右边的文本 |
| _$$_ | 美元符号 |

工作方式 _regex_constants_ 如下：

| **标志** | **描述** |
| --- | --- |
| `format_default` | 替换所有实例，并复制所有未匹配的内容到结果字符串 |
| `format_no_copy` | 替换所有实例，不复制未匹配的内容到结果字符串 |
| `format_first_only` | 只替换模式的第一个实例 |

```cpp
string str{R"(<body><h1>Header</h1><p>some text</p></body>)"};
regex reg(R"(<h1>(.*)</h1><p>(.*)</p>)");
const string format{"H1=$1 P=$2"};

string res1 = regex_replace(str, reg, format); // <body>H1=Header P=some text</body>
string res2 = regex_replace(str, reg, format, regex_constants::format_no_copy); // H1=Header P=some text
```
## ratio
用于精确的表示任何可以在编译时使用的有限有理数（分数），在`chrono::duration`中被使用，分子分母的类型为`intmax_t`编译时常量。
```cpp
// 由于在编译期确定值, 必须使用常量表达式, den 的默认参数为 1
const intmax_t n = 1;
const intmax_t d = 60;
using r1 = ratio<n, d>;

// 取值
intmax_t num = r1::num;
intmax_t den = r1::den;
```
### 数学运算

- `ratio_add` 
- `ratio_subtract`
- `ratio_multiply`
- `ratio_divide`
```cpp
using r1 = ratio<1, 60>;
using r2 = ratio<1, 30>;
using result = ratio_add<r1, r2>::type;
```
### 比较
返回值`bool_constant`为模板`integral_constant`，保存了一种类型和一个编译时常量，`integral_constant<int, 15>`保存一个值为 _15_ 的整型值，`true`则表示为`integral_constant<bool, true>`。

- `ratio_less` 
- `ratio_equal`
- `ratio_not_equal`
- `ratio_less_equal`
- `ratio_greater`
- `ratio_greater_equal`
```cpp
using r1 = ratio<1, 60>;
using r2 = ratio<1, 30>;
using res = ratio_less<r1, r2>;
cout << res::value << endl;
```
## chrono
时间相关的类和函数均在`std::chrono`命名空间中。
### duration
持续时间（_duration_）表示两个时间点之间的时间间隔，`ratio<1>`为默认的滴答周期，表示 _1 _秒。
标准 _duration _类型：

- `nanoseconds`
- `microseconds`
- `milliseconds`
- `seconds`
- `minites`
- `hours`
```cpp
auto t = hours(24);
```
_duration _的运算：

- `count`：以滴答数返回 _duration_ 值，返回值为 _duration_ 模板中指定的类型参数
- `zero`：返回持续时间等于 _0_ 的 _duration_
- `max`、`min`：返回最大或最小持续时间的 _duration_ 值，为 _duration_ 模板中指定的类型参数
- `floor`、`ceil`、`round`
- `abs`
```cpp
duration<long> d1(10); // 10 秒
duration<long, ratio<60>> d2(10); // 10 分钟
// 在 ratio 头文件中预定义了常用的单位
duration<long, micro> d3(10); // 10 毫秒, micro 为 ratio 头文件中预定义的单位

duration<long, ratio<1>> d4 = d1 + d2;
d3 *= 2;

cout << d4.count() << endl;

// 由于使用的是 long 类型, 可能会出现不能整除
// Error: duration<long, ratio<60>> d5(d1);
auto d5 = duration_cast<duration<long, ratio<60>>>(d1);
```
### time_point
`time_point`总是和某个特定的`clock`关联，表示时间中的某个时点，同 `duration `一样提供了数值运算，可以使用`time_point_cast`进行类型转换。
```cpp
time_point<steady_clock> tp1;
tp1 += minites(10);

time_point<steady_clock, seconds> tp2(42s);
time_point<steady_clock, milliseconds> tp3(tp2);
time_point<steady_clock, seconds> tp4(time_point_cast<seconds>(tp3));
```
### 时钟
`clock`由`time_point`和`duration`组成。

| **时钟** | **描述** |
| --- | --- |
| `system_clock` | 系统实时时钟的真实时间 |
| `steady_clock` | 一个能保证其`time_point`绝不递减的时钟 |
| `high_resolution_clock` | 滴答周期达到了最小值的高精度时钟 |

静态方法：

| **方法** | **描述** |
| --- | --- |
| `now` | 用于把当前时间作为`time_point`返回 |
| `to_time_t`、`from_time_t` | 用于在`time_t`类型之间转换（`system_clock`） |

```cpp
auto point = system_clock::now();
auto tt = system_clock::to_time_t(point);

struct tm *t = localtime(&tt);
char buf[BUFSIZ] = {0};
strftime(buf, sizeof(buf), "%H:%M:%S", t);
```
## 随机数
### 随机数引擎
| **引擎** | **描述** |
| --- | --- |
| `random_device` | 这种引擎比较特殊，要求计算机连接真正生成不确定随机数的硬件，通常比伪随机数更慢
- `entropy()`返回随机数的熵，决定随机数生成器的质量
 |
| `linear_congruential_engine` | 线性同余引擎，所需内存最少 |
| `mersenne_twister_engine` | 梅森旋转算法，随机数质量最高 |
| `subtract_with_carry_engine` | 带进位减法，质量不如梅森旋转算法 |

> 一般使用 `random_device`，这个引擎使用简单且不需要参数，其他几个引擎需要指定数学参数，较为复杂。

### 随机数引擎适配器
修改相关联的随机数引擎生成的结果，关联的随机数引擎成为基引擎。

| **适配器** | **描述** |
| --- | --- |
| `discard_block_engine` | 丢弃一些基引擎生成的值，以生成随机数 |
| `independent_bits_engine` | 组合由基引擎生成的随机数，生成具有给定位数的随机数 |
| `shuffle_order_engine` | 和基引擎生成的随机数一致 |

预定义随机数引擎和引擎适配器：

| **名称** | **模板** |
| --- | --- |
| `minstd_rand0` | `linear_congruential_engine` |
| `minstd_rand` | `linear_congruential_engine` |
| `mt19937` | `mersenne_twister_engine` |
| `mt19937_64` | `mersenne_twister_engine` |
| `ranlux24_base` | `linear_congruential_engine` |
| `ranlux48_base` | `subtract_with_carry_engine` |
| `ranlux24` | `discard_block_engine` |
| `ranlux48` | `discard_block_engine` |
| `knuth_b` | `shuffle_order_engine` |
| `default_random_engine` | 与编译器相关 |

### 分布
一个描述数字在特定范围内分布的数学公式。

| **分布** | **类型** |
| --- | --- |
| 均匀分布 | 
- `uniform_int_distribution`
- `uniform_real_distribution`
 |
| 伯努利分布  | 
- `bernoulli_distribution`
- `binomial_distribution`
- `geometric_distribution`
- `negative_binomial_distribution`
 |
| 泊松分布 | 
- `poisson_distribution`
- `exponential_distribution`
- `gamma_distribution`
- `weibull_distribution`
- `extreme_value_distribution`
 |
| 正态分布  | 
- `normal_distribution`
- `lognormal_distribution`
- `chi_squared_distribution`
- `cauchy_distribution`
- `fisher_f_distribution`
- `student_t_distribution`
 |
| 采样分布  | 
- `discrete_distribution`
- `piecewise_constant_distribution`
- `piecewise_linear_distribution`
 |

```cpp
random_device seeder;
const auto seed = seeder.entropy() ? seeder() : time(nullptr);
mt19937 eng(static_cast<mt19937::result_type>(seed));
uniform_int_distribution<int> dist(1, 99);

auto gen = bind(dist, eng);
vector<int> vec(10);
generate(begin(vec), end(vec), gen);
```
## optional
用于保存特定类型的值，或者什么也不保存。

| **方法** | **描述** |
| --- | --- |
| `has_value` | 是否包含值 |
| `value` | 获取包含的值，如果为空将抛出`bad_optional_access`异常 |
| `value_or` | 返回包含的值，如果为空则返回参数传入的值 |

不能在`optional`中存储引用，可以存储指针，`reference_wrapper<T>`，`reference_wrapper<const T>`，分别用`ref`或`cref`函数创建。
## variant
用于保存给定类型集合中的一个值。

| **方法** | **描述** |
| --- | --- |
| `index` | 获取当前存储的值的类型在类型集合中的索引 |

全局函数：

| **函数** | **描述** |
| --- | --- |
| `holds_alternative` | 确定当前`variant`是否包含特定类型的值 |
| `get<index>`、`get<T>` | 获取当前索引或类型的值，如果不匹配则抛出`bad_variant_access`异常 |
| `get_if<index>`、`get_if<T>` | 传入`variant`指针, 返回当前包含值的指针，如果不匹配则返回`nullptr` |
| `visit` | 应用 _visitor _模式，调用重载的函数调用运算符 |

`get<index>`中的 _index _必须是编译时已确定的。
```cpp
variant<int, float, string> v;
v = 10;
v = 12.5f;
v = "text";

auto res1 = v.index(); // 2
auto res2 = holds_alternative<string>(v); // true

string str1 = get<2>(v);
string *str2 = get_if<string>(&v);

// 如果类型没有默认构造函数, 可以使用 std::monostate 作为第一个参数
class Foo { Foo() = delete; Foo(int){} };
class Bar { Bar() = delete; Bar(int){} };
variant<std::monostate, Foo, Bar> v1;

class Visitor {
public:
    void operator()(int) {}
    void operator()(float){}
    void operator()(const string&) {}
};
visit(Visitor(), v);
```
## any
可以包含任意类型的值，使用`any_cast`获取当前的值, 如果失败则抛出`bad_any_cast`异常。

| **方法** | **描述** |
| --- | --- |
| `has_value` | 判断当前是否有值 |

```cpp
any intAny(10);
int i = any_cast<int>(intAny);
```
`any`可以当做容器的元素，用于存储任意类型的值。
## 元组
`tuple`是`pair`的泛化，允许存储任意数量的值，每个值都有自己特定的类型。
### 构造元组
可以使用构造函数创建，也可以使用辅助函数`make_tuple`。
```cpp
using MyTuple = tuple<int, float, bool>;

MyTuple t1(1, 2.0f, true);
tuple t2(1, 2.0f, true); // 类型推导

auto siz1 = tuple_size<MyTuple>::value; // 获取 tuple 的元素数
auto siz2 = tuple_size<decltype(t1)>::value;

string str = "123";
tuple t3(ref(str)); // 推导类型为 string&, 并非 reference_wrapper<T>

get<1>(t1) = 10.0f; // 赋值
```
### 分解元组
全局函数：

| **方法** | **描述** |
| --- | --- |
| `get<index>` | 获取指定索引的值 |
| `get<T>` | 获取指定类型的值，如果`tuple`中有多个 _T_ 类型，则会编译出错 |
| `tie` | 分解元组，可以使用`ignore`忽略不需要的值 |

`get<index>`中的 _index _必须是编译时已确定的。
```cpp
tuple t1(1, 2.0f, true);

// 解构赋值
auto [i1, f1, b1] = t1;

int i2;
bool b2;
tie(i2, ignore, b2) = t1;
```
### 串联元组
`tuple_cat`将两个元组串联为一个元组。
### 比较
`tuple`重载了比较运算符，可以按字典顺序比较对应元素的大小，需要各元素也支持比较操作，也可以使用`tie`和元组比较。
```cpp
tuple t1(1, "123");
tuple t2(2, "234");
cout << (t1 < t2) << endl;

int i1 = 1, i2 = 2;
string str1("123"), str2("234");
cout << (tie(i1, str1) < tie(i2, str2)) << endl;
```
### 构建对象
`make_from_tuple`将元素作为参数传递给 _T _类型的构造函数创建对象。
```cpp
class Foo { Foo(int, float) {} };
tuple t(10, 20.0f);

auto foo = make_from_tuple<Foo>(t);
```
使用此函数构造对象不一定必须使用`tuple`，但必须支持`std::get<>`，`std::tuple_size`操作，`std::array`和`std::pair`也能满足。
### 调用函数
```cpp
int add(int a, int b) { return a + b; }
cout << apply(add, make_tuple(1, 2)) << endl;
```
## 文件系统
_filesystem _的类和函数位于`std::filesystem`命名空间中。
### path
文件路径，可以是绝对路径，也可以是相对路径。
全局函数：

- `remove_filename` 
- `replace_filename`
- `replace_extension`
- `root_name`
- `parent_name`
- `extension`
- `has_extension`
- `is_absolute`
- `is_relative`
```cpp
path p1(LR"(./Foo)");
cout << p1 << endl;

p.append("Bar") // ./Foo/Bar
p /= "Bar"; // ./Foo/Bar/Bar

p.concat("bar"); // ./Foo/Bar/Barbar
p += "bar"; // ./Foo/Bar/Barbarbar

// 迭代路径, 输出: ., Foo, Bar, Barbarbar
for (const auto &component : p) {
    cout << component << endl;
}
```
### diectory_entry
表示文件系统的目录或文件（一定存在）。

- `is_directory`
- `is_regular_file`
- `is_socket`
- `is_symlink`
- `file_size`
- `last_write_time`
### 辅助函数
| **函数** | **描述** |
| --- | --- |
| `create_directory` | 创建目录 |
| `exists` | 文件是否存在 |
| `file_size` | 文件大小 |
| `last_write_time` | 文件最近一次修改的时间 |
| `remove` | 删除文件 |
| `temp_directory_path` | 获取保存临时文件的目录 |
| `space` | 查询文件系统中的可用空间 |

### 遍历目录
使用`recursive_directory_iterator`递归遍历目录：
```cpp
void process(const path & p) {
    if (!exists(p)) {
        return;
    }
    auto degin = recursive_directory_iterator(p);
    auto end = recursive_directory_iterator();
    for (auto iter = begin; iter != end; ++iter) {
        const string spacer(iter.depth() * 2, ' ');
        auto &entry = *iter;
        if (is_regular_file(entry)) {
            cout << spacer << "File: " << entry << " (" << file_size(entry) << " bytes" << endl;
        } else if (is_directory(entry)) {
            cout << spacer << "Dir: " << entry << endl;
        }
    }
}
```
使用`directory_iterator`迭代目录内容，并自行实现递归：
```cpp
void process(const path & p, size_t level = 0) {
    if (!exists(p)) {
        return;
    }

    const string spacer(level * 2, ' ');

    if (is_regular_file(entry)) {
        cout << spacer << "File: " << entry << " (" << file_size(entry) << " bytes" << endl;
    } else if (is_directory(entry)) {
        cout << spacer << "Dir: " << entry << endl;
        for (auto &entry : directory_iterator(p)) {
            process(entry, level + 1);
        }
    }
}
```
## 多线程
### 伪共享
大多数缓存都是用所谓的“缓存行（cache line）”，现代 _CPU _通常为 _64 _字节，如果需要写入缓存行，需要锁定整行数据，_C++17_ 在`<new>`中引入了`hardware_destructive_interference_size`常量返回两个并发访问的对象之间的建议偏移量，可将这个值与`alignas`关键字结合使用，以合理的对齐数据，避免共享缓存行。
### thread
创建线程：
```cpp
void counter(int id) {}
thread t1(counter, 1);

class Func {
public:
    Func() {}
    void operator()() {}

    void func() {}
};

Func fn;

thread t2(Func{});
thread t3{Func()};
thread t4(fn);

thread t5(&Func::func, &fn); // 成员函数
thread t6([](int i){}, 1); // lambda

// 运行错误, 编译器会将其当做函数指针声明 
thread t7(Func()); // Error!

this_thread::sleep_for(100ms); // 当前线程睡眠 100ms
```
参数为一个函数对象，还有任意个执行所需的参数。
如果一个线程对象表示系统当前或过去的某个活动线程，则认为它是可结合的（_joinable_），在销毁线程对象前，必须调用其`join`或`detach`方法，将会使线程变得不可结合，如果一个仍可结合的线程对象被销毁，析构函数会调用`terminate`，这会导致终止所有线程以及应用程序本身。

| **方法** | **描述** |
| --- | --- |
| `join` | 阻塞等待线程结束 |
| `detach` | 将线程对象与底层 _OS _线程分离 |

> `cout`是线程安全的，除非调用了`cout.sync_with_stdio(false)`。

### 线程本地存储
通过`thread_local`关键字修饰的变量，可使该变量在每个线程中都有一个独立副本，且在该线程生命周期中持续存在，且只初始化一次。
```cpp
thread_local int n;
```
### 异常处理
如果一个线程中抛出异常，如果不在该线程中处理，则很难将其捕获，标准线程库提供了解决该问题的方法。
```cpp
exception_ptr current_exception() noexcept;
```
返回一个引用当前正在处理的异常或其副本的`exception_ptr`对象。
```cpp
[[noreturn]] void rethrow_exception(exception_ptr);
```
可以跨线程将`exception_ptr`引用的异常重新抛出.
```cpp
template<class E> exception_ptr make_exception(E e) noexcept;
```
创建一个`exception_ptr`对象，引用给定异常的副本。
```cpp
void doSomeWork() {
    throw runtime_error("exception from thread");
}
void threadFunc(exception_ptr &err) {
    try {
        doSomeWork();
    } catch (...) {
        err = current_exception();
    }
}

exception_ptr err;
thread t(threadFunc, ref(err));
t.join();

if (err) {
    try {
        rethrow_exception(err);
    } catch (const exception &e) {
        cout << "main thread: " << e.what() << endl;
    }
}
```
### 原子操作
`atomic`可以和任何类型一起使用，但该类型必须具有`is_trivially_copy`特点，对于基本类型，`atomic`提供了相关的命名原子类型，只需在该类型前加上`atomic_*`前缀。
```cpp
atomic<int> i1(10);
i1++;

atomic_int i2(10);
i2++;
```
```cpp
compare_exchange_stong(T &expected, T desired);
```
如果当前`atomic`等于 _expected_，则将 _desired _赋值给当前`atomic`，否则将当前`atomic`赋值给 _expected。_
`fetch_add()`函数将参数加到当前`atomic`中，返回未递增的旧值。
### 互斥
#### 非定时互斥体
| **互斥体** | **描述** |
| --- | --- |
| `mutex` | 互斥体 |
| `recursive_mutex` | 可以在同一线程中多次获得所锁，但同时需要调用相同次数的解锁操作以释放锁 |
| `shared_mutex` | 共享锁（读写锁），在`<shared_mutex>`中定义<br>&nbsp;&nbsp;&#9670;&nbsp;`try_lock_shared()`：尝试获取读锁<br>&nbsp;&nbsp;&#9670;&nbsp;`lock_shared()` ：获取读锁（阻塞）
 |

| **方法** | **描述** |
| --- | --- |
| `lock` | 获取锁（阻塞） |
| `try_lock` | 尝试获得锁，成功返回`true`，否则返回`false` |
| `unlock` | 解锁 |

#### 定时互斥体

- `timed_mutex`
- `recursive_timed_mutex`
- `shared_timed_mutex`
   - `try_lock_shared_for()`
   - `try_lock_shared_until()`

和定时互斥体支持相同的方法，但还提供了尝试在给定时间内获得锁的方法。

| **方法** | **描述** |
| --- | --- |
| `try_lock_for` | 相对时间，类型为`chrono::duration` |
| `try_lock_until` | 绝对时间，类型为`chrono::time_point` |

### 锁
锁均为 _RAII_ 类，析构函数会自动释放关联的锁。

- `lock_guard`
- `unique_lock`
   - `owns_lock()`：是否获得了锁
- `shared_lock`：获得共享锁

上述锁的构造函数存在多种重载形式，特别是`unique_lock`，第一个参数始终未互斥体的引用，第二个参数说明如下：

| **参数** | **描述** |
| --- | --- |
| `adopt_lock_t` | 参数的互斥体已经上锁，用于管理互斥体，在销毁时释放锁, 全局实例`std::adopt_lock` |
| `defer_lock_t` | 不立即获得锁，全局实例`std::defer_lock` |
| `try_to_lock_t` | 尝试获得锁，即使未能获得也不阻塞，全局实例`std::try_to_lock` |
| `time_point` | 尝试获得锁，阻塞等待直到获得锁或系统时间超过该绝对时间 |
| `duration` | 尝试获得锁，阻塞等待直到获得锁或者超时 |

#### 同时获得多个锁
全局函数：

| **函数** | **描述** |
| --- | --- |
| `lock` | 获得参数中的所有锁 |
| `try_lock` | 调用所有锁的`try_lock`，成功返回 _-1_，如果有一个失败则会释放已获得的锁，返回调用失败的锁的索引 |

```cpp
mutex m1, m2;
unique_lock lock1(m1, defer_lock);
unique_lock lock2(m2, defer_lock);
lock(m1, m2);
```
##### scoped_lock
与`lock_guard`类似，只是可以接受任意数量的互斥体。
```cpp
mutex m1, m2;
scoped_lock<mutex, mutex> lock1(m1, m2);
scoped_lock lock2(m1, m2); // 模板推导, C++17
```
### std::call_once
结合使用`call_once`和`once_flag`可确保某个函数只会调用一次，对于同一个`once_flag`，多个线程竞争调用时，只有一个线程会有效调用，但在有效调用结束前，其他线程将阻塞。
```cpp
once_flag flag;

void initShared() {}
void process() {
    call_once(flag, initShared);
}

vector<thread> threads(3);
for (auto &t : threads) {
    t = thread{process};
}
for (auto &t : threads) {
    t.join();
}
```
### 条件变量
条件变量包含在`<condition_variable>`头文件中。

| **条件变量** | **描述** |
| --- | --- |
| `condition_variable` | 只能等待`unique_lock<mutex>`，在特定平台效率最高 |
| `condition_variable_any` | 可等待任何对象的条件变量，包括自定义类型 |

| **方法** | **描述** |
| --- | --- |
| `notify_one` | 只唤醒一个等待的线程 |
| `notify_all` | 唤醒所有等待的线程 |
| `wait` | 该函数调用前，`unique_lock<mutex>`应该已经获得锁，调用时会原子的解锁并阻塞等待，直到被唤醒并重新加锁返回 |
| `wait_for` | 同`wait`，可以指定等待的相对超时时间 |
| `wait_until` | 同`wait`，可以指定等待的绝对超时时间 |

```cpp
queue<string> que;
mutex mu;
condition_variable cond;

// 线程1
unique_lock lock(mu);
que.push("1");
cond.notify_all();

// 线程2
unique_lock lock(mu);
while (true) {
    cond.wait(lock, [&que](){ return !que.empty(); });
}
```
### future 和 promise
使用`futrue`可以方便的从线程获得结果，其将结果存储在`promise`中，也就是说，`promise`是输入端，而`future`是输出端。
#### future
| **方法** | **描述** |
| --- | --- |
| `get` | 检索结果，当结果还未准备好时，调用将阻塞，且只能获取一次，多次调用结果不确定 |
| `wait` | 等待结果 |
| `wait_for` | 等待结果直到超时，返回`true`表示结果已经转备好 |

#### promise
| **方法** | **描述** |
| --- | --- |
| `get_future` | 获取关联的`future`对象 |
| `set_value` | 存储结果 |
| `set_exception` | 存储异常 |

只允许调用上述方法中的一个，且只能调用一次，多次调用将抛出`future_error`异常。
```cpp
void doWork(promise<int> thePromise) {
    thePromise.set_value(42);
}

promise<int> myPromise;
auto theFuture = myPromise.get_future();
// promise 对象不可复制
thread t(doWork, move(myPromise));

int res = theFuture.get();
t.join();
```
`packaged_task`会自动创建`promise`，不需要显式的存储结果，自动存储函数的返回值或抛出的异常。
```cpp
int calc(int a, int b) { return a + b; }

packaged_task<int(int, int)> task(calc);
auto theFuture = task.get_future();
// packaged_task 对象不可复制
thread t(move(task), 1, 2);

int res = theFuture.get();
t.join();
```
`sync`可以让 _C++_ 运行时更多的控制是否创建一个线程来进行计算。

| **参数** | **描述** |
| --- | --- |
| `launch::async` | 在不同的线程中异步执行函数 |
| `launch::deferred` | 调用`get`时，在主调线程同步执行函数 |
| `launch::any` | 允许 _C++_ 运行时进行选择 |

```cpp
int calc(int a, int b) {
   if (a == 0 || b == 0) {
       throw runtime_error("Cannot be zero!");
   }
   return a + b;
}

auto myFuture = async(launch::deferred, calc, 0, 0);
try {
    auto res = theFuture.get();
}  catch (const exception &e) {
    cout << e.what() << endl;
}
```
#### shared_future
`shared_future`允许多次调用`get`，但要求结果参数是可复制构建的，可通过两种方式创建：

- `future`作为参数调用构造函数
- `future`对象调用`share`返回
```cpp
promise<void> thread1Started, thread2Started;
promise<int> signalPromise;
auto signalFuture = signalPromise.get_future().share();
// shared_future<int> signalFuture(signalPromise.get_future());

auto fn1 = [&thread1Started, signalFuture] {
    thread1Started.set_value();
    int param = signalFuture.get();
}
auto fn2 = [&thread2Started, signalFuture] {
    thread2Started.set_value();
    int param = signalFuture.get();
}

auto res1 = async(launch::async, fn1);
auto res2 = async(launch::async, fn2);
thread1Started.get_future().wait();
thread2Started.get_future().wait();
signalPromise.set_value(1);
```
