# 简介
参考书籍[现代c++核心语言特性](https://book.douban.com/subject/35602582/)
> 默认：C++11

# 新基础类型

```cpp
long long x = 1LL;
// 也能运用于枚举类型和位域
enum longlong_enum : long long {
    x1,
};
struct longlong_struct {
    long long x1 : 8;
};
#define LLONG_MAX 9223372036854775807LL // long long的最大值
// 使用类模板方法
std::cout << "std::numeric_limits<long long>::max() = "
    << std::numeric_limits<long long>::max() << std::endl;
std::printf("LLONG_MAX = %lld\n", LLONG_MAX);

char16_t utf16[] = u"你好世界"; // char8_t、char32_t
using u16string = basic_string;  // u32string、wstring
using u32string = basic_string;
```

# 内联和嵌套命名空间
内联命名空间特性可以帮助库作者无缝升级库代码，让客户不用修改任何代码也能够自由选择新老库代码
```cpp
namespace Parent {
    // c++14: deprecated
    namespace [[deprecated( 字符字面量 )]] Child1  { void foo() { } }
    inline namespace Child2 { void foo() { } }
}
Parent::foo(); // 默认 Child2

// c++17: 嵌套命名空间的简化语法
namespace A::B::C { int foo() {  }}
std::vector<int>::iterator it;
// C++20: 
namespace A::B::inline C {}
```

# auto
C++11标准赋予了auto新的含义：声明变量时根据初始化表达式自动推断该变量的类型、声明函数时函数返回值的占位符。

注意事项：
+ 当用一个auto关键字声明多个变量的时候，编译器遵从由左往右的推导规则，以最左边的表达式推断auto的具体类型
+ 当使用条件表达式初始化auto声明的变量时，编译器总是使用表达能力更强的类型
+ C++11 不支持声明非静态成员变量初始化
+ 按照C++20之前的标准，无法在函数形参列表中使用auto声明形参
```cpp
auto *pn = &n, m = 10; // auto被推导为int
auto i = true ? 5 : 8.0; // i的数据类型为double
struct sometype {
    static const auto i = 5; // c++17 支持可以不用const
};
```

推导规则
+ 如果auto声明的变量是按值初始化（既没有使用引用，也没有使用指针），则推导出的类型会忽略cv（const和volatile）限定符
+ 使用auto声明变量初始化时，目标对象如果是引用，则引用属性会被忽略
+ 使用auto和万能引用声明变量时，对于左值会将auto推导为引用类型
+ 使用auto声明变量，如果目标对象是一个数组或者函数，则auto会被推导为对应的指针类型
+ 当auto关键字与列表初始化组合时，直接使用列表初始化，列表中必须为单元素，用等号加列表初始化，列表中可以包含单个或者多个元
素
```cpp
const int i = 5;
auto j = i; // auto推导类型为int，而非const int
auto &m = i; // auto推导类型为const int，m推导类型为const int&
auto *k = i; // auto推导类型为const int，k推导类型为const int*

int i = 5;
int &j = i;
auto m = j; // auto推导类型为int，而非int&

int i = 5;
auto&& m = i; // auto推导类型为int& （引用折叠）
auto&& j = 5; // 5是一个右值，因此j的类型被推导为int&&，auto被推导为int

int i[5];
auto m = i; // auto推导类型为int*

auto x1 = { 1, 2 }; // x1类型为 std::initializer_list<int>
auto x2 = { 1, 2.0 }; // 编译失败，花括号中元素类型不同
auto x3{ 1, 2 }; // 编译失败，不是单个元素
auto x4 = { 3 }; // x4类型为std::initializer_list<int>
auto x5{ 3 }; // x5类型为int
```

auto的使用规则。
+ 当一眼就能看出声明变量的初始化类型的时候可以使用auto
+ 对于复杂的类型，例如lambda表达式、bind等直接使用auto

```cpp
for (auto &it : map) {}

auto l = [](int a1, int a2) { return a1 + a2; }; // l的类型较复杂
auto b = std::bind(sum, 5, std::placeholders::_1); // b的类型较复杂
```

C++14标准支持对返回类型声明为auto的推导

```cpp
auto sum(int a1, int a2) { return a1 + a2; }

// 后置返回类型：这是唯一让lambda表达式通过推导返回引用类型的方法
auto l = [](int &i)->auto& { return i; };
auto x1 = 5;
auto &x2 = l(x1);
assert(&x1 == &x2); // 有相同的内存地址

// 非类型模板形参占位符
template<auto N>f(){};
f<5.0>(); // 编译失败，模板参数不能为double
```

# decltype
decltype(e)（其中e的类型为T）的推导规则
+ 如果e是一个未加括号的标识符表达式（结构化绑定除外）或者未加括号的类成员访问，则decltype(e)推断出的类型是e的类型T。如果并不存在这样的类型，或者e是一组重载函数，则无法进行推导。
+ 如果e是一个函数调用或者仿函数调用，那么decltype(e)推断出的类型是其返回值的类型。
+ 如果e是一个类型为T的左值，则decltype(e)是T&。
+ 如果e是一个类型为T的将亡值，则decltype(e)是T&&。
+ 除去以上情况，则decltype(e)是T

通常情况下，decltype(e)所推导的类型会同步e的cv限定符， 当e是未加括号的成员变量时，父对象表达式的cv限定符会被忽略
```cpp
const int&& foo();
int i;
struct A {double x;};
const A* a = new A();
decltype(foo()); // 推导类型为const int&&
decltype(i); // 推导类型为int
decltype(a->x); //推导类型为double
decltype((a->x)); //加括号，是左值，则a是const, 推导类型为const double&

int i;
int *j;
int n[10];
const int&& foo();
decltype(static_cast<short>(i)); // short
decltype(n); // decltype(n)推导类型为int[10]
decltype(foo); // decltype(foo)推导类型为int const && (void)
struct A { int operator() () { return 0; }};
A a;
decltype(a()); // decltype(a())推导类型为int

decltype(i=0); // 返回左值i, 推导类型为int&
decltype(0,i); // 返回左值i, 推导类型为int&
decltype(i,0); // 返回0, 推导类型为int
decltype(n[5]); // 左值, 推导类型为int&
decltype(*j); // 左值, 推导类型为int&
decltype(static_cast<int&&>(i)); // 将亡值类型, 推导类型为int&&
decltype(i++); //右值, 推导类型为int
decltype(++i); // 左值，推导类型为int&
decltype("hello world"); // 常量数组的左值， 推导类型为const char(&)[12]
```


## typeid
typeid运算符来获取与目标操作数类型有关的信息。获取的类型信息会包含在一个类型为std::type_info的对象里
+ typeid的返回值是一个左值，且其生命周期一直被扩展到程序生命周期结束
+ typeid返回的std::type_info删除了复制构造函数，若想保存std::type_info，只能获取其引用或者指针
+ typeid的返回值总是忽略类型的 cv 限定符

typeid 不能再编译器确定对象类型
```cpp
// GCC扩展
typeof(a) b = 5;

std::cout << typeid(int).name() << std::endl;
auto t1 = typeid(int); // 编译失败，没有复制构造函数无法编译
auto &t2 = typeid(int); // 编译成功，t2推导为const std::type_info&
auto t3 = &typeid(int); // 编译成功，t3推导为const std::type_info*
```

## 获取对象或者表达式的类型
decltype能在非静态成员变量中使用
```cpp
decltype(x1 + x3) x4 = x1 + x3;

decltype(x1) sum(decltype(x1) a1) { return a1;}
```

decltype的优势: 需要用到函数返回类型后置

```cpp
// 如何需要支持各种参数，则需要用到模板
auto sum(int a1, int a2)->int { return a1+a2;}

// c++14: 支持对auto声明的返回类型进行推导, 但在返回引用则存在问题
template<class T1, class T2>
auto sum(T1 a1, T2 a2)->decltype(a1 + a2) {return a1 + a2;}

template<class T>
auto return_ref(T& t)->decltype(t) {return t;}
static_assert(std::is_reference_v<decltype(return_ref(x1))); // 编译成功
```

## decltype(auto)
C++14: 告诉编译器用decltype的推导表达式规则来推导auto, 必须单独声明，也就是它不能结合指针、引用以及cv限定符

```cpp
int i;
int&& f();
auto x1a = i; // x1a推导类型为int
decltype(auto) x1d = i; // x1d推导类型为int
auto x2a = (i); // x2a推导类型为int
decltype(auto) x2d = (i); // x2d推导类型为int&
auto x3a = f(); // x3a推导类型为int
decltype(auto) x3d = f(); // x3d推导类型为int&&
auto x4a = { 1, 2 }; // x4a推导类型为std::initializer_list<int>
decltype(auto) x4d = { 1, 2 }; // 编译失败, {1, 2}不是表达式
auto *x5a = &i; // x5a推导类型为int*
decltype(auto)*x5d = &i; // 编译失败，decltype(auto)必须单独声明

template<class T>
decltype(auto) return_ref(T& t) { return t;}

// C++17: 作为非类型模板形参的占位符
template<decltype(auto) N>
static int y = 7;
f<y>(); // 编译错误, 因为y不是一个常量，所以编译器无法对函数模板进行实例化
f<(y)>(); // N为int&类型, 静态对象而言内存地址是固定的
```

# 函数返回类型后置

在无法使用C++14以及更新标准的情况下，通过返回类型后置语法来推导函数模板的返回类型无疑是最便捷的方法
```cpp
// 传统函数声明语法的foo1无法将函数指针类型作为返回类型直接使用，需要结合typedef
auto foo2()->int(*)(int) { return ;}
// 推导函数模板的返回类型
auto sum1(T1 t1, T2 t2)->decltype(t1 + t2) {}
```

# 右值引用
右值引用是C++11标准提出的一个非常重要的概念，它的出现不仅完善了C++的语法，改善了C++在数据转移时的执行效率，同时还增强了C++模板的能力

左值一般是指一个指向特定内存的具有名称的值（具名对象），它有一个相对稳定的内存地址，并且有一段较长的生命周期。
右值则是不指向稳定内存地址的匿名值（不具名对象），它的生命周期很短，通常是暂时性的。
可以用取地址符&来判断左值和右值，能取到内存地址的值为左值，否则为右值

```cpp
int *q = &++x; // x++是右值,返回临时复制内容, ++x 是左值
int get_val() { return x;} // 函数返回的是临时内容，所以是右值
// 通常字面量都是一个右值，除字符串字面量以外
auto p = &"c"; // 程序加载的时候会为其开辟内存空间
```

## 左值引用
左值引用，可以免去创建临时对象的操作
```cpp
const int &x = 11; // 编译成功, 常量左值引用还能够引用右值

class X{
    X(const X&) {}
    X& operator = (const X&) { return *this; }
};
X make_x() { return X();}
// 为非常量左值引用无法绑定到make_x()产生的右值
X x3(make_x());
x3 = make_x();
```

## 右值引用
右值引用是一种引用右值且只能引用右值的方法, 在左值引用声明中，需要在类型后添加&，而右值引用则是在类型后添加&&
右值引用延长临时对象生命周期, 从而减少对象复制

> `-fno-elide-constructors`用于关闭函数返回值优化（RVO）
```cpp
X &&x2 = make_x();
```

## 移动语义
复制构造函数而言形参是一个左值引用，往往进行的是深复制，即在不能破坏实参对象的前提下复制目标对象。
移动构造函数接受的是一个右值，其核心思想是通过转移实参对象的数据以达成构造目标对象的目的，也就是说实参对象是会被修改的

对于右值编译器会优先选择使用移动构造函数去构造目标对象
```cpp
// 移动构造函数
X(X&& other) {
    pool_ = other.pool_;
    other.pool_ = nullptr;
}
// 移动赋值运算符函数
X& operator=(X&& other) {
    pool_ = other.pool_;
    other.pool_ = nullptr;
    return *this;
}
```

在一个移动构造函数中，如果当一个对象的资源移动到另一个对象时发生了异常，也就是说对象的一部分发生了转移而另一部分没有，这就会造成源对象和目标对象都不完整的情况发生，这种情况的后果是无法预测的。
在编写移动语义的函数时建议确保函数不会抛出异常，与此同时，如果无法保证移动构造函数不会抛出异常，可以使用`noexcept`说明符限制该函数。这样当函数抛出异常的时候，程序不会再继续执行而是调用`std::terminate`中止执行以免造成其他不良影响。

## 值类别
值类别: 是表达式的一种属性，分为3个类别:左值（lvalue）、纯右值（prvalue）和将亡值（xvalue）
左值是指非将亡值的泛左值，而右值则包含了纯右值和将亡值

表达式首先被分为了泛左值（glvalue）和右值（rvalue），其中泛左值被进一步划分为左值和将亡值，右值又被划分为将亡值和纯右值。
+ 泛左值是指一个通过评估能够确定对象、位域或函数的标识的表达式。简单来说，它确定了对象或者函数的标识（具名对象）。
+ 纯右值是指一个通过评估能够用于初始化对象和位域，或者能够计算运算符操作数的值的表达式。
+ 将亡值属于泛左值的一种，它表示资源可以被重用的对象和位域，通常这是因为它们接近其生命周期的末尾，另外也可能是经过右值引用的转换产生的。

将亡值的途径有两种
+ 使用类型转换将泛左值转换为该类型的右值引用
+ c++17, 临时量实质化，指的是纯右值转换到临时对象的过程

```cpp
// 显式地将左值通过static_cast转换为将亡值
int &&k = static_cast<int&&>(i);
// 正确的使用场景是在一个右值被转换为左值后需要再次转换为右值，最典型的例子是一个右值作为实参传递到函数中

void move_pool(X &&pool) { X my_pool(pool); }
move_pool(make_pool());
// 函数模板std::move 将左值转换为右值，内部也是用static_cast做类型转换
X my_pool(std::move(pool));
```

## 万能引用和引用折叠
万能引用既可以绑定左值也可以绑定右值，甚至const和volatile的值都可以绑定
万能引用是因为发生了类型推导，在T&&和auto&&的初始化过程中都会发生类型的推导
原理： 引用折叠

|类模板型 | T实际类型  | 最终类型 |
| --- | --- | ---- |
|T& |R   | R& |
|T& |R&  | R& |
|T& |R&& | R& |
|T&&|R   | R&& |
|T&&|R&  | R& | 
|T&&|R&& | R&& |

只要有左值引用参与进来，最后推导的结果就是一个左值引用。只有实际类型是一个非引用类型或者右值引用类型时，最后推导出来的才是一个右值引用


```cpp
template<class T>
void foo(T &&t) {} // t为万能引用
auto &&y = get_val(); // y为万能引用

// 必须在初始化的时候被直接推导出来，如果在推导中出现中间过程，则不是一个万能引用
```

## 完美转发
万能引用最典型的用途被称为完美转发

`std::move`: 一定会将实参转换为一个右值引用，并且使用不需要指定模板实参，模板实参是由函数调用推导出来的
`std::forward`: 会根据左值和右值的实际情况进行转发，在使用的时候需要指定模板实参
```cpp
template<class T>
void show_type(T t) { std::cout << typeid(t).name() << std::endl;}
// 完美转发
template<class T>
void perfect_forwarding(T &&t) { 
    // 是因为作为形参的t是左值。为了让转发将左右值的属性也带到目标函数中，这里需要进行类型转换
    show_type(static_cast<T&&>(t)); 
    show_type(std::forward<T>(t));
}

std::string get_string(){return "c";}
std::string s = "c";
perfect_forwarding(s);
perfect_forwarding(get_string());
```

## 针对局部变量和右值引用的隐式移动操作
C++20: 可隐式移动的对象必须是一个非易失或一个右值引用的非易失自动存储对象，在以下情况下可以使用移动代替复制
+ return或者co_return语句中的返回对象是函数或者lambda表达式中的对象或形参
+ throw语句中抛出的对象是函数或try代码块中的对象

# lambda表达式

```cpp
[ captures ] ( params ) specifiers exception -> ret { body }

int x = 1;
int main() {
    int y = 2;
    static int z = 3;
    auto foo = [y] { return x + y + z; }; // 直接使用x, z
}

// 无状态lambda表达式可以转换为函数指针
void g() { f([] {}); } // 隐式转换为void(*)()类型的函数指针
void g() { f(*[] {}); } // 隐式转换为 void(&)()

// 在STL中使用lambda表达式
*std::find_if(x.cbegin(),x.cend(),[](int i) { return (i % 3) == 0; });
```

`[ captures ]` —— 捕获列表，它可以捕获当前函数作用域的零个或多个变量，变量之间用逗号分隔。
+ 两个作用域: lambda表达式定义的函数作用域以及lambda表达式函数体的作用域
+ 非静态的局部变量
+ 捕获方式分为捕获值和捕获引用
+ 捕获的变量默认为常量，或者说lambda是一个常量函数
+ 捕获值的变量在lambda表达式定义的时候已经固定下来了，无论函数在lambda表达式定义后如何修改外部变量的值，lambda表达式捕获的值都不会变化
+ `[this]` —— 捕获this指针，捕获this指针可以使用this类型的成员变量和函数
+ `[=]` —— 捕获lambda表达式定义作用域的全部变量的值，包括this
+ `[&]` —— 捕获lambda表达式定义作用域的全部变量的引用，包括this

`( params )` —— 可选参数列表，语法和普通函数的参数列表一样，在不需要参数的时候可以忽略参数列表

specifiers —— 可选限定符
+ mutable，在lambda表达式函数体内改变按值捕获的变量，或者调用非const的成员函数

`exception` —— 可选异常说明符，可以使用noexcept来指明lambda是否会抛出异常

`ret` —— 可选返回值类型，，如果没有返回值（void类型），可以忽略包括->在内的整个部分

`{ body }` —— lambda表达式的函数体，这个部分和普通函数的函数体一样

## lambda表达式的实现原理

lambda表达式在编译期会由编译器自动生成一个闭包类，在运行时由这个闭包类产生一个对象，称它为闭包。
在C++中，所谓的闭包可以简单地理解为一个匿名且可以包含定义时作用域上下文的函数对象。

```cpp
int main()
{
    int x = 5, y = 8;
    auto foo = [=] { return x * y; };
    int z = foo();
}

// 用GCC输出其GIMPLE的中间代码
main () {
    int D.39253;
    {
        int x;
        int y;
        struct __lambda0 foo;
        // __lambda0是一个拥有operator()自定义运算符的结构体，这也正是函数对象类型的特性
        typedef struct __lambda0 __lambda0;
        int z;
        try {
            x = 5;
            y = 8;
            foo.__x = x;
            foo.__y = y;
            z = main()::<lambda()>::operator() (&foo);
        } finally  {
            foo = {CLOBBER};
        }
    }
    D.39253 = 0;
    return D.39253;
}

main()::<lambda()>::operator() (const struct __lambda0 * const __closure)
{
    int D.39255;
    const int x [value-expr: __closure->__x];
    const int y [value-expr: __closure->__y];
    _1 = __closure->__x;
    _2 = __closure->__y;
    D.39255 = _1 * _2;
    return D.39255;
}
```

## 广义捕获
c++14: 简单捕获(只能捕获lambda表达式定义上下文的变量), 初始化捕获(捕获表达式结果以及自定义捕获变量名)
```cpp
auto foo = [x = x + 1]{ return x; };
// 1) 使用移动操作减少代码运行的开销
auto foo = [x = std::move(x)]{ return x + "world"; };

class Work
{
private: int value;
public:
    Work() : value(42) {}
    // 2) 在异步调用时复制this对象，防止lambda表达式被调用时因原始this对象被析构造成未定义的行为
    std::future<int> spawn() { return std::async([[=, tmp = *this]]() -> int { return tmp.value; });}
};
std::future<int> foo() { Work tmp; return tmp.spawn();}
std::future<int> f = foo();
f.wait();
```

## 增强
C++17标准对lambda表达式同样有两处增强，一处是常量lambda表达式（constexpr），另一处是对捕获*this的增强。

```cpp
// c++14： 泛型lambda表达式
auto foo = [](auto a) { return a; };
// [*this]的语法让程序生成了一个*this对象的副本并存储在lambda表达式内
return std::async([=, *this]() -> int { return value; });
// c++20: 使语义更加明确，区分它与[=,*this]的不同（捕获this对象的副本）
// 强调了要用[=, this]代替[=]
[=, this]{};
// 在C++20中添加模板对lambda的支持
auto f = []<typename T>(T const& x) {
    T copy = x;
    using Iterator = typename T::iterator;
};

// C++20标准允许了无状态lambda表达式类型的构造和赋值
auto greater = [](auto x, auto y) { return x > y; };
std::map<std::string, int, decltype(greater)> mymap;
```

## 递归

```cpp
// 1) std::function可以把lambda包装起来，相当于赋予了其一个函数名，在通过引用捕获并实现递归调用
std::function<int(const int&)>sum1 = [&](const int& n) {
	return n == 1 ? 1 : n + s(n - 1);
};

// 2) 将lambda作为参数
const auto& sum2 = [](const int& n) {
	const auto& s = [&](auto&& self, const int& x) -> int{
		return x == 1 ? 1 : x + self(self, x - 1);
	};
	return s(s,n);
};

// 3) 基于Y不动点组合子, 科里化
auto T = [&](auto x) { return [&](auto y) { return y([&](auto z) {return x(x)(y)(z); }); }; };
auto X = T(T);
auto fib = X([](auto f) { return [&](auto n)->int { return n < 2 ? n : f(n - 1) + f(n - 2); }; });
```

# 非静态数据成员默认初始化
注意：
+ 不要使用括号()对非静态数据成员进行初始化，因为这样会造成解析问题，所以会编译错误
+ 不要用auto来声明和初始化非静态数据成员
```cpp
class X {
public:
    X() {}
    // 每个构造函数只需要专注于特殊成员的初始化，而其他的数据成员则默认使用声明时初始化的值。
    X(int a) : a_(a) {}
    X(double b) : b_(b) {}
private:
    // 即在声明非静态数据成员的同时直接对其使用=或者{}
    int a_ = 0;
    double b_{ 0. };
};
// C++20: 对数据成员的位域进行默认初始化
struct S2 {
    int y : (true ? 8 : a) = 42;
    int z : (1 || new int){ 0 };
};
```

# 列表初始化
列表初始化：使用大括号{}对变量进行初始化，和传统变量初始化的规则一样，它也区分为直接初始化和拷贝初始化

```cpp
// 传统：new运算符和类构造函数的初始化列表就属于直接初始化，而函数传参和return返回则是拷贝初始化
int x = 5; // 拷贝初始化（复制初始化），注意是隐式调用直接初始化
int x1(8); // 直接初始化, 类似构造函数

struct C {
    C(std::string a, int b) {}
    C(int a) {}
};
void foo(C) {}
C bar() { return {"world", 5};}
int main() {
    int x = {5}; // 拷贝初始化
    int x1{8}; // 直接初始化
    C x2 = {4}; // 拷贝初始化
    C x3{2}; // 直接初始化
    foo({8}); // 拷贝初始化
    foo({"hello", 8}); // 拷贝初始化
    C x4 = bar(); // 拷贝初始化
    C *x5 = new C{ "hi", 42 }; // 直接初始化
    std::vector<int> x2{ 1,2,3,4,5 };
    std::vector<int> x3 = { 1,2,3,4,5 };
    std::map<std::string, int> x8{ {"bear",4} };
}
```

## std::initializer_list
标准容器之所以能够支持列表初始化，离不开编译器支持的同时，它们自己也必须满足一个条件：支持`std::initializer_lis`t为形参的构造函数。
即一个一个支持begin、end以及size成员函数的类模板， 返回是一个常量对象指针`const T*`

## 注意事项
隐式缩窄转换
+ 从浮点类型转换整数类型
+ 从long double转换到double或float，或从double转换到float，除非转换源是常量表达式以及转换后的实际值在目标可以表示的值范围内
+ 从整数类型或非强枚举类型转换到浮点类型，除非转换源是常量表达式，转换后的实际值适合目标类型并且能够将生成目标类型的目
标值转换回原始类型的原始值。
+ 从整数类型或非强枚举类型转换到不能代表所有原始类型值的整数类型，除非源是一个常量表达式，其值在转换之后能够适合目标类型

优先级问题
+ 如果有一个类同时拥有满足列表初始化的构造函数，且其中一个是以 `std::initializer_list`为参数，那么编译器该构造函数。

## 指定初始化
C++20标准中引入了指定初始化的特性。该特性允许指定初始化数据成员的名称
+ 要求对象必须是一个聚合类型, 没有构造函数
+ 指定的数据成员必须是非静态数据成员
+ 指定的数据成员必须是非静态数据成员
+ 非静态数据成员的初始化必须按照声明的顺序进行
+ 针对联合体中的数据成员只能初始化一次，不能同时指定
+ 不能嵌套指定初始化数据成员
+ 不能混用其他方法对数据成员初始化了
```cpp
Point p{ .x = 4, .y = 2 };
```

# 默认和删除函数
在没有自定义构造函数的情况下，编译器会为类添加默认的构造函数
1．默认构造函数。
2．析构函数。
3．复制构造函数。
4．复制赋值运算符函数。
5．移动构造函数（C++11新增）。
6．移动赋值运算符函数（C++11新增）。

C++11标准提供了一种方法能够简单有效又精确地控制默认特殊成员函数的添加和删除，叫作显式默认和显式删除

```cpp
struct type {
    // 类就从非平凡类型恢复到平凡类型了
    type() = default;
    // 像自动变量、静态变量或者全局变量这种会隐式调用析构函数的对象就无法创建了
    virtual ~type() = delete; //delete 必须添加在类内部的函数声明中
    type(const type &);
    void * operator new(std::size_t) = delete; // 阻止该类在堆上动态创建对象
};
type::type(const type &) = default;
```

# 非受限联合类型
当面对一个可能被滥用的功能时，语言的设计者往往有两条路可走，一是为了语言的安全性禁止此功能，另外则是为了语言的能力和灵活性允许这个功能，C++的设计者一般会采用后者。

过去的C++标准规定，联合类型的成员变量的类型不能是一个非平凡类型，也就是说它的成员类型不能有自定义构造函数
在C++11标准中解除了大部分限制，联合类型的成员可以是除了引用类型外的所有类型

C++17： 则大部分情况下可以使用std:: variant来代替联合体
```cpp
union U {
    U() {} // 存在非平凡类型成员，必须提供构造函数
    ~U() {} // 存在非平凡类型成员，必须提供析构函数
    std::string x3;
};
// 用了placement new的技巧来初始化, 保证了联合类型使用的灵活性和正确性
new(&u.x3) std::string("c");
u.x3.~basic_string(); // 手动释放
```

# 委托构造函数
放在函数里并不是一个初始化过程，而是一个赋值过程

注意：
+ 每个构造函数都可以委托另一个构造函数为代理
+ 不要递归循环委托
+ 如果一个构造函数为委托构造函数，那么其初始化列表里就不能对数据成员和基类进行初始化
+ 委托构造函数的执行顺序是先执行代理构造函数的初始化列表，然后执行代理构造函数的主体，最后执行委托构造函数的主体
+ 如果在代理构造函数执行完成后，委托构造函数主体抛出了异常，则自动调用该类型的析构函数

```cpp
struct X {
    X() : X(0, 0.) {}
    X(int a) : X(a, 0.) {}
    X(double b) : X(0, b) {}
    X(int a, double b) : a_(a), b_(b) {}
}
```

## 委托模板构造函数

```cpp
class X {
    template<class T> X(T first, T last) : l_(first, last) { }
    std::list<int> l_;
public:
    X(std::vector<short>&);
    X(std::deque<int>&);
};
X::X(std::vector<short>& v) : X(v.begin(), v.end()) { }
X::X(std::deque<int>& v) : X(v.begin(), v.end()) { }
```

## 捕获委托构造函数的异常
当使用Function-try-block去捕获委托构造函数异常时，其过程和捕获初始化列表异常如出一辙。如果一个异常在代理构造函数的初始
化列表或者主体中被抛出，那么委托构造函数的主体将不再被执行，与之相对的，控制权会交到异常捕获的catch代码块中：

```cpp
class X {
public:
    X() try : X(0) {} catch (int e) {throw 3;}
    X(int a) try : X(a, 0.) {}catch (int e) {throw 2;}
    X(double b) : X(0, b) {}
    X(int a, double b) : a_(a), b_(b) { throw 1; }
private:
    int a_; double b_;
};
X x; // throw 1 2 3
```

# 继承构造函数
使用using关键字将基类的函数引入派生类

规则:
+ 派生类是隐式继承基类的构造函数，所以只有在程序中使用了这些构造函数，编译器才会为派生类生成继承构造函数的代码
+ 派生类不会继承基类的默认构造函数和复制构造函数
+ 不会影响派生类默认构造函数的隐式声明
+ 在派生类中声明签名相同的构造函数会禁止继承相应的构造函数
+ 派生类继承多个签名相同的构造函数会导致编译失败
+ 继承构造函数的基类构造函数不能为私有

```cpp
class Derived : public Base {
public:
    using Base::foo; // 导入函数
    using Base::Base; // 让编译器为自己生成转发到基类的构造函数
};
```

# 强枚举类型
C++是一门类型安全的强类型语言，但是枚举类型在一定程度上却是一个例外
+ 枚举类型存在一定的安全检查功能，一个枚举类型不允许分配到另外一种枚举类型，而且整型也无法隐式转换成枚举类型
+ 枚举类型却可以隐式转换为整型
+ 枚举类型会把其内部的枚举标识符导出到枚举被定义的作用域

强枚举类型具备以下3个新特性：
+ 枚举标识符属于强枚举类型的作用域
+ 枚举标识符不会隐式转换为整型。
+ 能指定强枚举类型的底层类型，底层类型默认为int类型

```cpp
// class: 强枚举类型
enum class type: unsigned int {};
// C++17： 列表初始化有底层类型枚举对象
type c{ 5 };
// C++20: 打开强枚举类型的命名空间
using enum type; 
```

# 扩展的聚合类型
C++17标准对聚合类型的定义做出了大幅修改，即从基类公开且非虚继承的类也可能是一个聚合
+ 没有用户提供的构造函数
+ 没有私有和受保护的非静态数据成员
+ 没有虚函数
+ 必须是公开的基类，不能是私有或者受保护的基类
+ 必须是非虚继承

在标准库 `<type_traits>` 中提供了一个聚合类型的甄别办法 `is_aggregate`

```cpp
std::is_aggregate_v<std::string>;
```

# override和final说明符
1．重写（override）的意思更接近覆盖，在C++中是**指派生类覆盖了基类的虚函数**，这里的覆盖必须满足有相同的函数签名和返回类型，也就是说有相同的函数名、形参列表以及返回类型。

2．重载（overload），它通常是**指在同一个类中有两个或者两个以上函数，它们的函数名相同，但是函数签名不同**，也就是说有不同的形参。这种情况在类的构造函数中最容易看到，为了让类更方便使用，我们经常会重载多个构造函数。

3．隐藏（overwrite）的概念也十分容易与上面的概念混淆。隐藏是指**基类成员函数，无论它是否为虚函数，当派生类出现同名函数时，如果派生类函数签名不同于基类函数，则基类函数会被隐藏**。如果派生类函数签名与基类函数相同，则需要确定基类函数是否为虚函数，如果是虚函数，则这里的概念就是重写；否则基类函数也会被隐藏。另外，如果还想使用基类函数，可以使用using关键字将其引入派生类。

>final说明符可以修饰最底层基类的虚函数而override则不行
```cpp
// 明确告诉编译器这个虚函数需要覆盖基类的虚函数
virtual void bar() override {}
// 告诉编译器该虚函数不能被派生类重写
void foo(int x) final {};
virtual void log(const char *) const override final {…}
```

# 基于范围的for循环
范围表达式可以是数组或对象，对象必须满足以下2个条件中的任意一个
+ 对象类型定义了begin和end成员函数
+ 定义了以对象类型为参数的begin和end普通函数

如果不会在循环过程中修改引用对象，那么推荐在范围声明中加上const限定符以帮助编译器生成更加高效的代码
```cpp
std::map<int, std::string> index_map{ {1, "c"}};
void print(std::map<int, std::string>::const_reference e){}
std::for_each(index_map.begin(), index_map.end(), print);

for ( range_declaration : range_expression ) {}
for (const auto &e : index_map) { e.first, e.second;}

// c++11 伪代码： 要求begin_expr和end_expr返回的必须是同类型的对象
{
    auto && __range = range_expression;
    for (auto __begin = begin_expr, __end = end_expr; __begin != __end;
        ++__begin) {
        range_declaration = *__begin;
        loop_statement
    }
}
// c++17 伪代码
{
    auto && __range = range_expression;
    auto __begin = begin_expr;
    auto __end = end_expr;
    for (; __begin != __end; ++__begin) {
        range_declaration = *__begin;
        loop_statement
    }
}

// 如果range_ expression是一个泛左值，那结果可就不确定了
class T {
std::vector<int> data_;
public:
    std::vector<int>& items() { return data_; }
};
T foo() { T t; return t;}
for (auto& x : foo().items()) {} //返回的是一个泛左值类型std::vector<int>& 未定义行为
T thing = foo();
for (auto & x :thing.items()) {} // 正确
// 在C++20标准中，基于范围的for循环增加了对初始化语句的支持
for (T thing = foo(); auto & x :thing.items()) {}
```

# 支持初始化语句的if和 switch

```cpp
// c++17:
if (init; condition) {}
// 因为if初始化语句声明的变量会贯穿整个if结构, 加锁：
if (std::lock_guard<std::mutex> lock(mx); shared_flag) {shared_flag = false;}
```

# static_assert声明
运行时断言, 无法模板实例化的时候对模板实参进行约束

对静态断言的要求：
1．所有处理必须在编译期间执行，不允许有空间或时间上的运行时成本。
2．它必须具有简单的语法。
3．断言失败可以显示丰富的错误诊断信息。
4．它可以在命名空间、类或代码块内使用。
5．失败的断言会在编译阶段报错。

```cpp
static_assert(常量表达式,"诊断消息字符串");

template<class T>
class E {
    static_assert(std::is_base_of<A, T>::value, "T is not base of A");
};
// 让常量表达式作为诊断信息字符串参数的默认值是非常理想的
#define LAZY_STATIC_ASSERT(B) static_assert(B, #B)
// C++17: 忽略第二个参数
static_assert(std::is_base_of<A, T>::value);
```

# 结构化绑定
C++17标准中新引入的特性——结构化绑定。指将一个或者多个名称绑定到初始化对象中的一个或者多个子对象（或者元素）上，相当于给初始化对象的子对象（或者元素）起了别名，**别名不同于引用**

在结构化绑定中编译器会根据限定符生成一个等号右边对象的匿名副本，而绑定的对象正是这个副本而非原对象本身
别名的类型和绑定目标对象的子对象类型相同，而引用类型本身就是一种和非引用类型不同的类型

结构化绑定可以作用于3种类型: 包括原生数组、结构体和类对象、元组和类元组的对象

绑定元组和类元组有一系列抽象的条件：对于元组或者类元组类型T
+ 需要满足`std::tuple_size<T>::value`是一个符合语法的表达式，并且该表达式获得的整数值与标识符列表中的别名个数相同
+ 类型T还需要保证 `std::tuple_element<i, T>::type` 也是一个符合语法的表达式，其中i是小于 `std::tuple_size<T>::value` 的整数，表达式代表了类型T中第i个元素的类型
+ 类型T必须存在合法的成员函数模板`get<i>()`或者函数模板`get<i>(t)`，其中i是小于`std::tuple_size<T>::value`的整数，t是类型T的实例，`get<i>()`和`get<i>(t)`返回的是实例t中第i个元素的值

`std::pair` 和 `std::array`也能作为结构化绑定的目标，其原因就是它们是满足上述条件的类元组

C++20标准规定结构化绑定的限制不再强调必须为公开数据成员，编译器会根据当前操作的上下文来判断是否允许结构化绑定
```cpp
std::tuple<int, int> return_multiple_values() { return std::make_tuple(1, 2);}
// 使用函数模板std::tie将x和y通过引用绑定到std::tuple<int&, int&>上
std::tie(x, y) = return_multiple_values();

auto return_multiple_values(){return std::make_tuple(11, 7);}
auto[x, y] = return_multiple_values();
// 直接绑定到结构体上
for (const auto& [x, y] : bt) {}

struct BindBase3 {int a = 42;};
struct BindTest3 : BindBase3 { double b = 11.7;};
namespace std {
    template<> struct tuple_size<BindTest3> { static constexpr size_t value = 2;};
    template<> struct tuple_element<0, BindTest3> {using type = int; }; // 要明确的是每个子对象和元素的类型
    template<> struct tuple_element<1, BindTest3> {using type = double;};
}
// 可以明确地告知编译器不要生成除了特化版本以外的函数实例以防止get函数模板被滥用
template<std::size_t Idx> auto& get(BindTest3 &bt) = delete;
template<>auto& get<0>(BindTest3 &bt) { return bt.a; }
template<>auto& get<1>(BindTest3 &bt) { return bt.b;}

BindTest3 bt3;
auto& [x3, y3] = bt3;
x3 = 78;
std::cout << bt3.a << std::endl;
```

# noexcept关键字
移动构造函数中包含着一个严重的异常陷阱: 如果数据移动的途中发生异常，那么原始容器也将无法继续使用

noexcept是一个与异常相关的关键字，它既是一个说明符，也是一个运算符
+ 作为说明符，它能够用来说明函数是否会抛出异常, 指示编译器是不会抛出异常的，编译器可以根据声明优化代码, 一种承诺, 当抛出异常时，程序会调用`std::terminate`去结束程序的生命周期,还能接受一个返回布尔的常量表达式，为false的时候，则表示该函数有可能会抛出异常
+ noexcept运算符接受表达式参数并返回true或false。因为该过程是在编译阶段进行，所以表达式本身并不会被执行。而表达式的结果取决于编译器是否在表达式中找到潜在异常

```cpp
void g() noexcept {}
// 通过std::is_fundamental来判断T是否为基础类型
template <class T> T copy(const T &o) noexcept(std::is_fundamental<T>::value) {}
// noexcept(T(o)) 是运算符
template <class T> T copy(const T &o) noexcept(noexcept(T(o))) {}

// 如果没有抛出异常的可能，那么函数可以选择进行移动操作；否则将使用传统的复制操作
template<class T>
void swap(T& a, T& b) noexcept(noexcept(T(std::move(a))) && noexcept(a.operator=(std::move(b))))
{
    T tmp(std::move(a));
    a = std::move(b);
    b = std::move(tmp);
}
```

C++11标准规定下面几种函数会默认带有noexcept声明:
+ 默认构造函数、默认复制构造函数、默认赋值函数、默认移动构造函数和默认移动赋值函数, 同时对应的函数在类型的基类和成员中也具有noexcept声明
+ 类型的析构函数以及delete运算符默认带有noexcept声明， 自定义实现的析构函数也会默认带有noexcept声明

# 类型别名和别名模板

```cpp
typedef type-id identifier;
using identifier = type-id; // type-id是已有的类型名

typedef void(*func1)(int, int);
using func2 = void(*)(int, int);
// 别名模板
template<class T>
using int_map = std::map<int, T>;
```

# 指针字面量nullptr
关键字nullptr表示空指针的字面量，它是一个 `std::nullptr_t` 类型的纯右值。就是用来指示空指针，它不允许运用在算术表达式中或者与非指针类型进行比较（除了空指针常量0）

使用nullptr的另一个好处是，可以为函数模板或者类设计一些空指针类型的特化版本

```cpp
// 0依然保留着可以代表整数和空指针常量的特殊能力
assert(nullptr == 0);
```

# 三向比较（C++20）
根据标准三向比较会返回3种类型，分别为`std::strong_ordering`、`std::weak_ordering` 以及 `std::partial_ordering`
该特性的引入为实现比较运算提供了方便。只需要实现`==`和`<=>`两个运算符函数，剩下的4个运算符函数就可以交给编译器自动生成了

```cpp
// 运算符<=>的返回值只能与0和自身类型来比较
bool b = 7 <=> 11 < 0; // b == true
```

# 线程局部存储
thread_local说明符可以用来声明线程生命周期的对象，它能与static或extern结合，分别指定内部或外部链接，不过额外的static并不影响对象的生命周期

使用取地址运算符&取到的线程局部存储变量的地址是运行时被计算出来的，它不是一个常量，也就是说无法和`constexpr`结合

在同一个线程中，一个线程局部存储对象只会初始化一次，即使在某个函数中被多次调用
```cpp
thread_local static int errno;
```

# 扩展的inline说明符（C++17）
在C++17标准之前, 定义类的非常量静态成员变量的声明和定义必须分开进行

```cpp
struct X {
    // 即使将类X的定义作为头文件包含在多个CPP中也不会有任何问题
    inline static std::string text{"c"};
};
```

# 常量表达式
constexpr值即常量表达式值，是一个用constexpr说明符声明的变量或者数据成员，它要求该值必须在编译期计算。另外，常量表达式值必须被常量表达式初始化

constexpr函数要求：
+ 函数必须返回一个值，所以它的返回值类型不能是void
+ 函数体必须只有一条语句：return expr，其中expr必须也是一个常量表达式
+ 函数使用之前必须有定义
+ 函数必须用constexpr声明

当带形参的常量表达式函数接受了一个非常量实参时，常量表达式函数可能会退化为普通函数

使用constexpr声明自定义类型的变量，必须确保这个自定义类型的析构函数是平凡的
1．自定义类型中不能有用户自定义的析构函数。
2．析构函数不能是虚函数。
3．基类和成员的析构函数必须都是平凡的。

constexpr 支持声明浮点类型的常量表达式值，而且标准还规定其精度必须至少和运行时的精度相同
```cpp
constexpr int x = 42;
constexpr int max_unsigned_char() {return 0xff;}
constexpr double sum(double x) { return x > 0 ? x + sum(x - 1) : 0;}
struct X {
    constexpr X() : x1(5) {}
    constexpr int get() const
}
constexpr X x;
char buffer[x.get()] = { 0 };
```

C++14标准对常量表达式函数的改进如下
+ 函数体允许声明变量，除了没有初始化、static和thread_local变量
+ 函数允许出现if和switch语句，不能使用go语句
+ 函数允许所有的循环语句
+ 函数可以修改生命周期和常量表达式相同的对象
+ 函数的返回值可以声明为void
+ constexpr声明的成员函数不再具有const属性

从C++17开始，lambda表达式在条件允许的情况下都会隐式声明为constexpr
在C++17标准中，constexpr声明静态成员变量时，也被赋予了该变量的内联属性
`if constexpr`是C++17标准提出的一个非常有用的特性，可以用于编写紧凑的模板代码，让代码能够根据编译时的条件进行实例化
if constexpr不支持短路规则
```cpp
auto get_size = [](int i) constexpr -> int { return i * 2; };
char buffer2[get_size(5)] = { 0 };

#include <type_traits>
template<class T> bool is_same_value(T a, T b)
{
    if constexpr (std::is_same<T, double>::value) { return std::abs(a - b) < 0.0001 }
    else {  return a == b;} // 如果不是 double, 则上面直接被优化掉
}
```

c++20: 允许constexpr虚函数
c++20: 允许在constexpr函数中出现Try-catch
c++20: 允许在constexpr中进行平凡的默认初始化
C++20: 允许在constexpr中更改联合类型的有效成员

## 立即函数
希望确保函数在编译期就执行计算，对于无法在编译期执行计算的情况则让编译器直接报错

```cpp
consteval int sqr(int n) {return n*n;}
auto sqr = [](int n) consteval { return n * n; }
```

## 使用constinit检查常量初始化
“Static Initialization Order Fiasco”，指的是因为静态初始化顺序错误导致的问题。
c++20: constinit说明符主要用于具有静态存储持续时间的变量声明上，它要求变量具有常量初始化程序

```cpp
constinit int x = 11; // 编译成功，全局变量具有静态存储持续
int main() {
    constinit static int y = 42; // 编译成功，静态变量具有静态存储持续
    constinit int z = 7; // 编译失败，局部变量是动态分配的
}
```

## 判断常量求值环境
`std::is_constant_evaluated` 是C++20新加入标准库的函数，它用于检查当前表达式是否是一个常量求值环境
```cpp
constexpr inline bool is_constant_evaluated() noexcept
{
    return __builtin_is_constant_evaluated();
}
```

# 字面量优化
从C++11开始，标准库中引入了 `std::hexfloat` 将浮点数格式化为十六进制的字符串，而`std::defaultfloat`可以将格式还原到十进制

```cpp
std::cout << std::hexfloat << 13.14 << " = " << std::defaultfloat << 13.14 << std::endl;
// C++17: 十六进制浮点数的表示方法
double a = 0x1.7p+2;
// c++14: 二进制整数字面量, 还增加了一个用单引号作为整数分隔符的特性
auto x = 0b11001101L + 0xcdl + 077LL + 42 + 123'456;
// C++11: 原生字符串字面量, 说保留了字符串里的格式和特殊字符，同时它也会忽略转移字符，所见即所得
prefixR"delimiter(raw_ characters)delimiter"; // prefix和delimiter是可选部分
// delimiter可以是由除括号、反斜杠和空格以外的任何源字符构成的字符序列，长度至多为16个字符, 改变默认判定
```

## 用户自定义字面量
C++11： 可以通过自定义后缀将整数、浮点数、字符和字符串转化为特定的对象

```cpp

template<int scale, char … unit_char>
struct LengthUnit {
    constexpr static int value = scale;
    constexpr static char unit_str[sizeof…(unit_char) + 1] = { unit_char…,'\0' };
};
template<class T>
class LengthWithUnit {
public:
    LengthWithUnit() : length_unit_(0) {}
    LengthWithUnit(unsigned long long length) : length_unit_(length *T::value) {}
    template<class U>
    LengthWithUnit<std::conditional_t<(T::value > U::value), U, T>> operator+(const LengthWithUnit<U> &rhs)
    {
        using unit_type = std::conditional_t<(T::value > U::value), U, T>;
        return LengthWithUnit<unit_type>((length_unit_ + rhs.get_length()) / unit_type::value);
    }
    unsigned long long get_length() const { return length_unit_; }
    constexpr static const char* get_unit_str() { return T::unit_str; }
private:
    unsigned long long length_unit_;
};
template<class T>
std::ostream& operator<< (std::ostream& out, const LengthWithUnit<T>&unit)
{
    out << unit.get_length() / T::value <<
    LengthWithUnit<T>::get_unit_str();
    return out;
}
using MMUnit = LengthUnit<1, 'm', 'm'>;
using CMUnit = LengthUnit<10, 'c', 'm'>;
using LengthWithMMUnit = LengthWithUnit<MMUnit>;
using LengthWithCMUnit = LengthWithUnit<CMUnit>;

auto total_length = LengthWithCMUnit(1) + LengthWithMUnit(2) +LengthWithMMUnit(4);
std::cout << total_length;

// 使用用户自定义字面量来简化代码
retrun_type operator "" identifier (params){}

LengthWithMMUnit operator "" _mm(unsigned long long length)
{
    return LengthWithMMUnit(length);
}
LengthWithCMUnit operator "" _cm(unsigned long long length)
{
    return LengthWithCMUnit(length);
}
auto total_length = 1_cm  + 4_mm;
```

# alignas和alignof
C++11中新增了alignof和alignas两个关键字， 解决了长期以来C++标准中无法对数据对齐进行处理的问题
+ alignof运算符可以用于获取类型的对齐字节长度
+ alignas说明符可以用来改变类型的默认对齐字节长度

C++11定义了 `std::max_align_t`，它是一个平凡的标准布局类型，其对齐字节长度要求至少与每个标量类型一样严格。还规定，诸如new和malloc之类的分配函数返回的指针需要适合于任何对象

如果将alignas用于结构体类型，那么该结构体整体就会以alignas声明的对齐字节长度进行对齐


```cpp
// offsetof来间接实现alignof的功能
#define ALIGNOF(type, result) \
    struct type##_alignof_trick{ char c; type member; }; \
    result = offsetof(type##_alignof_trick, member)

auto x1 = alignof(int);
auto x2 = alignof(void(*)());
auto x3 = alignof(a); // *C++标准不支持这种用法
auto x3 = alignof(decltype(a));

// 可以接受类型或者常量表达式, 必须2的幂值
alignas(8) int a = 0;
alignas(double) int a2;
auto x3 = alignof(decltype(a)); // 错误的返回4，而并非设置的8

std::cout << std::alignment_of<int>::value << std::endl; // 输出4
std::cout << std::alignment_of<int>() << std::endl; // 输出4

std::aligned_storage<128, 16>::type buffer; // 用来分配一块指定对齐字节长度和大小的内存
std::aligned_union<64, double, int, char>::type buffer; // 获取这些类型中对齐字节长度最严格的
// std::align函数模板，该函数接受一个指定大小的缓冲区空间的指针和一个对齐字节长度，返回一个该缓冲区中最近的能找到符合指定对齐字节长度的指针

// 在C++17: 拥有了根据对齐字节长度分配对象的能力
// 编译器会自动从类型对齐字节长度的属性中获取这个参数并且传参
void* operator new(std::size_t, std::align_val_t);
void* operator new[](std::size_t, std::align_val_t);
```

# 属性说明符和标准属性
GCC从2.9.3版本开始支持GCC手册的属性语法，后来一些编译器为了兼容以GCC为基础编写的代码也纷纷支持了GCC的属性语法
GCC的属性语法十分灵活，它能够用于结构体、类、联合类型、枚举类型、变量或者函数

```cpp
_attribute__((attribute-list))

// C++11标准的属性表示方法:
[[attr]] [[attr1, attr2, attr3(args)]] [[namespace::attr(args)]]
```

## 标准属性
从C++11到C++20标准一共只定义了9种标准属性, 一方面他们需要考虑一个语言特性应该定义为关键字还是定义为属性，另一方面还需要谨慎考虑该属性是否是平台通用

carries_dependency是C++11标准引入的属性，该属性允许跨函数传递内存依赖项，它通常用于弱内存顺序架构平台上多线程程序的优化，避免编译器生成不必要的内存栅栏指令。所谓弱内存顺序架构，简单来说是指在多核心的情况下，一个核心看到共享内存中的值的变化与另一个核心写入它们的顺序不同。IBM的PowerPC就是这样的架构

只有当被声明为nodiscard属性的类或者枚举类型被当作函数返回值的时候才发挥作用
+ 防止资源泄露，malloc或者new
+ 对于工厂函数而言，真正有意义的是回返的对象而不是工厂函数
+ 对于返回值会影响程序运行流程的函数而言

```cpp
// c++11: 编译器会认为在这个函数中执行流会被中断，函数不会返回到其调用者
[[noreturn]] void foo() {}
// c++14: 带有此属性的实体被声明为弃用, 编译器通常会给出警告
[[deprecated("xx")]] void foo() {}
class [[deprecated]] X {};
// c++17: 提示编译器直落行为是有意的，并不需要给出警告 
case 1:
bar();
[[fallthrough]];
case 2:

// C++17: 该属性声明函数的返回值不应该被舍弃，否则鼓励编译器给出警告提示
class [[nodiscard("Memory leak!")]] X {};
[[nodiscard]] int foo() { return 1; }
// c++17: 消除编译器警告
int foo(int a [[maybe_unused]]) {}
// c++20: likely和unlikely是C++20标准引入的属性
[[unlikely]] case 2: return 2;
// c++20: 该属性指示编译器该数据成员不需要唯一的地址
struct X {
    int i;
    [[no_unique_address ]]Empty e;
};
```

# 新增预处理器和宏

```cpp
// c++17: __has_includ, 用于判断某个头文件是否能够被包含进来
#if __has_include(<optional>)
// c++20: 属性特性测试宏
__has_cpp_attribute(deprecated);
// 语言功能特性测试宏, 扩展到添加到标准时的年份和月份
__cpp_variadic_templates
// 标准库功能特性测试宏
__cpp_lib_addressof_constexpr

#define LOG(msg, …) printf("[" __FILE__ ":%d] " msg, __LINE__,##__VA_ARGS__)
// c++20: __VA_OPT__令可变参数宏更易于在可变参数为空的情况下使用
#define LOG(msg, …) printf("[" __FILE__ ":%d] " msg, __LINE__ __VA_OPT__(,) __VA_ARGS__)
```

# 协程（C++20）
2019年3月的C++标准委员会上才通过投票加入C++20标准中，并且最终采用了微软的实现方案
包含 co_await、co_return和co_yield 任意关键字一个的函数就是协程。但main函数不能为协程
通常情况下，建议将协程和标准库中的future、generator一起使用，因为协程辅助代码较为复杂，所以应该尽量避免自定义它们

```cpp
using namespace std::chrono_literals;
std::future<int> foo()
{
    std::cout << "call foo\n";
    std::this_thread::sleep_for(3s);
    co_return 5;
}
std::future<std::future<int>> bar()
{
    std::cout << "call bar\n";
    std::cout << "before foo\n";
    auto n = co_await std::async(foo); // 挂起点
    std::cout << "after foo\n";
    co_return n;
}
int main()
{
    std::cout << "before bar\n";
    auto i = bar();
    std::cout << "after bar\n";
    i.wait();
    std::cout << "result = " << i.get().get();
}
```

## 协程的实现原理

### co_await
co_await运算符可以创建一个挂起点将协程挂起并等待协程恢复
可等待体: 该对象是可以被等待的
等待器: 包含下面3个成员函数

```cpp
auto n = co_await std::async(foo);
// 等价于
std::future<std::future<int>> expr = std::async(foo);
auto n = co_await expr; // 该对象是可以被等待的
```
目标对象可被等待需要实现await_resume、await_ready和await_suspen这3个成员函数
+ await_ready: 用于判定可等待体是否已经准备好
+ await_suspend: 调度协程的执行流程，比如异步等待可等待体的结果、恢复协程以及将执行的控制权返回调用者
+ await_resume: 用于接收异步执行结果

coroutine_handle: 是协程的句柄，可以用于控制协程的运行流程, 包含协程挂起和恢复的上下文信息,有operator()和resume()
函数，它们可以执行挂起点之后的代码

await_suspend 返回类型：
1）返回void类型表示协程需要将执行流的控制权交给调用者，协程保持挂起状态。
2）返回bool类型则又会出现两种情况，当返回值为true时，效果和返回类型与void相同；当返回false的时候，则恢复当前协程运行。
3）返回coroutine_handle类型的时候，则会恢复该句柄对应的协程

```cpp
class file_io_string {
public:
    file_io_string(const char* file_name) {
        t_ = std::thread{ [file_name, this]() mutable {
            std::ifstream f(file_name);
            std::string str((std::istreambuf_iterator<char>(f)),
            std::istreambuf_iterator<char>());
            result_ = str;
            ready_ = true;
        } };
    }
    bool await_ready() const { return ready_; }
    void await_suspend(std::experimental::coroutine_handle<> h) {
        std::thread r{ [h, t = std::move(t_)] () mutable {
            t.join();   h();  }
        };
        r.detach();
    }
    std::string await_resume() const { return result_; }
private:
    bool ready_ = false;
    std::thread t_;
    std::string result_;
};
std::future<std::string> foo()
{
    auto str = co_await file_io_string{ "test.txt" };
    co_return str;
}

// 重载co_await运算符，让它从可等待体转换为等待器
awaitable_string operator co_await(std::string&& str)
{
    return awaitable_string{ str };
}
```

### co_yield
promise_type可以用于自定义协程自身行为，代码的编写者可以自定义协程的多种状态以及自定义协程中任
何co_await、co_return或co_yield表达式的行为，比如挂起前和恢复后的处理、如何返回最终结果等

通常情况下promise_type会作为函数的嵌套类型存在，比如`std::experimental:: generator`类模板
但不是所有的满足，另一种 `std::experimental::coroutine_traits<T>` 比如 `std::future`

```cpp
#include <experimental/resumable>
using namespace std::experimental;
struct my_int_generator {
    struct promise_type {
        int *value_ = nullptr;
        my_int_generator get_return_object() { return my_int_generator{*this}; }
        auto initial_suspend() const noexcept{  return suspend_always{};}
        auto final_suspend() const noexcept { return suspend_always{}; }
        auto yield_value(int &value){
            value_ = &value;
            return suspend_always{};
        }
        void return_void() {}
    };
    explicit my_int_generator(promise_type &p)  : handle_(coroutine_handle<promise_type>::from_promise(p)) {}
    ~my_int_generator() { if (handle_) handle_.destroy();}
    int next() {
        if (!handle_) return -1;
        handle_();
        if (handle_.done()) {
            handle_.destroy();
            handle_ = nullptr;
            return -1;
        }
        return handle_.promise().value_;
    }
    coroutine_handle<promise_type> handle_;
};
my_int_generator foo() {
    for (int i = 0; i < 10; i++) co_yield i;
}
auto obj = foo();
std::cout << obj.next() << std::endl;
```

### co_return
co_return也需要promise_type的支持

```cpp
struct my_int_return{
    struct promise_type {
        int value_ = 0;
        my_int_return get_return_object(){return my_int_return{*this};}
        auto initial_suspend() const noexcept {  return suspend_never{}; }
        auto final_suspend() const noexcept{  return suspend_always{};}
        void return_value(int value){    value_ = value;}
    };
    explicit my_int_return(promise_type &p) : handle_(coroutine_handle<promise_type>::from_promise(p)) {}
    ~my_int_return(){ if (handle_) handle_.destroy();}
    int get(){
        if (!ready_){
            value_ = handle_.promise().value_;
            ready_ = true;
            if (handle_.done()) {
                handle_.destroy();
                handle_ = nullptr;
            }
        }
        return value_;
    }
    coroutine_handle<promise_type> handle_;
    int value_ = 0;
    bool ready_ = false;
};
my_int_return foo() {
    co_return 5;
}

auto obj = foo();
std::cout << obj.get();
```

### promise_type

```cpp
// 可对co_await的操作数进行转换处理
awaitable await_transform(expr e) {return awaitable(e);}
void unhandled_exception() { eptr_ = std::current_exception();}
```

# 基础特性的其他优化

在某些期待上下文为bool类型的语境中，可以隐式执行布尔转换，即使这个转换被声明为显式。这些语境包括以下几种。
+ if、while、for的控制表达式
+ 内建逻辑运算符!、&&和||的操作数
+ 条件运算符?:的首个操作数
+ static_assert声明中的bool常量表达式
+ noexcept说明符中的表达式

```cpp
// 显式自定义类型转换
explicit operator bool() const { return !data_.empty(); }
static_cast <bool>(s1); // 显式地转换
```

如果新的对象在已被某个对象占用的内存上进行构建，那么原始对象的指针、引用以及对象名都会自动转向新的对象，除非对象是一个常量类型或对象中有常量数据成员或者引用类型

```cpp
struct X { const int n; };
union U { X x; float f; };
U u = {{ 1 }};
X *p = new (&u.x) X {2}; // replace new, 未定义行为：

// c++17: 是一个有定义的行为，而且获取n的值也保证为2
assert(*std::launder(&u.x.n) == 2);
```

## 返回值优化
允许编译器将函数返回的对象直接构造到它们本来要存储的变量空间中而不产生临时对象。严格来说返回值优化分为RVO（Return Value Optimization）和
NRVO（Named Return Value Optimization）

RVO: 当返回语句的操作数为临时对象时, NRVO: 当返回语句的操作数为具名对象时
c++11: 复制消除（copy elision）gcc 默认开启（取消`-fno-elide- constructors`）

C++14标准对返回值优化做了进一步的规定，规定中明确了对于常量表达式和常量初始化而言，编译器应该保证RVO，但是禁止NRVO

在C++17标准中提到了确保复制消除的新特性: 在传递临时对象或者从函数返回临时对象的情况下，编译器应该省略对象的复制和移动构造函数，即使这些复制和移动构造还有一些额外的作用，最终还是直接将对象构造到目标的存储变量上，从而避免临时对象的产生。标准还强调，这里的复制和移动构造函数甚至可以是不存在或者不可访问的

## c++20
类的默认比较规则要求类C可以有一个参数为const C&的非静态成员函数，或者有两个参数为const C&的友元函数
C++20标准对这一条规则做了适度的放宽，它规定类的默认比较运算符函数可以是一个参数为const C&的非静态成员函数，或是两个参数为const C&或C的友元函数

通常情况下delete一个对象，编译器会先调用该对象的析构函数，之后才会调用delete运算符删除内存
完善了调用伪析构函数结束对象声明周期的规则：总是会结束对象的生命周期，即使对象是一个平凡类型

修复const和默认复制构造函数不匹配造成无法编译的问题
不推荐在下标表达式中使用逗号运算符， 希望这种表达方式能用在矩阵、视图、几何实体、图形API中
```cpp
friend bool operator==(C, C) = default;
// 支持new表达式推导数组长度
int *x = new int[]{ 1, 2, 3 };
// 允许数组转换为未知范围的数组
void f(int(&)[]) {}

// 允许控制行为： 第一个参数类型由void *修改为X *；增加了一个形参
void operator delete(X* ptr, std::destroying_delete_t)
{
    ptr->~X(); // 这时候需要手动调用析构函数
    ::operator delete(ptr);
}

std::cout << a[(x, y)];
```

## 不推荐使用volatile的情况
volatile：用于表达易失性。它能够让编译器不要对代码做过多的优化，保证数据的加载和存储操作被多次执行
volatile限定符的意义已经不大了：并不能保证数据的同步，无法保证内存操作不被中断，也不能代替原子操作，对于现代CPU而言这种操作执行顺序也是无法保证的

C++20标准在部分情况中不推荐volatile的使用, 给出警告
+ 不推荐算术类型++,--
+ 不推荐非类类型左操作数的赋值
+ 不推荐函数形参和返回类型
+ 不推荐结构化绑定

## 模块
使用模块也能大幅提升编译效率，因为编译后的模块信息会存储在一个二进制文件中，编译器对于它的处理速度要远快于单纯使用文本替换的头文件方法
目前为止并没有主流编译器完全支持该特性

```cpp
// helloworld.ixx
export module helloworld;
import std.core;
export void hello() {
           std::cout << "Hello world!\n";
}
// modules_test.cpp
import helloworld;
hello();
```

# 可变参数模板

```cpp
// 只要保证后续的形参类型能够通过实参推导或者具有默认参数即可
template<class …Args, class T, class U = double>
void foo(T, U, Args …args) {}
// 在类模板中，模板形参包必须是模板形参列表的最后一个形参
template<classT, class …Args>
class bar { // 类型模板形参包，它可以接受零个或者多个类型的模板实参
public:
    bar(Args …args) {foo(args…); } // args…是形参包展开，通常简称包展开, 解包
};
```

## 形参包展开
允许包展开的场景包括以下几种
1．表达式列表
2．初始化列表
3．基类描述
4．成员初始化列表
5．函数参数列表
6．模板参数列表
7．动态异常列表
8．lambda表达式捕获列表
9．Sizeof…运算符
10．对其运算符
11．属性列表

```cpp
bar<int, double, unsigned int> b(1, 5.0, 8);
```

# SFINAE
SFINAE（Substitution Failure Is Not An Error，替换失败不是错误）主要是指在函数模板重载时，当模板形参替换为指定的实参或由函数实参推导出模板形参的过程中出现了失败，则放弃这个重载而不是抛出一个编译失败
