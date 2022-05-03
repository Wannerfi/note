# Primer c++第五版笔记7(到第19章完)

~~这是笔记，所以别问为什么写的跟书上一摸一样了~~

[toc]

## 19 特殊工具与技术

### 19.1 控制内存分配

```cpp
string* sp = new string("a value");
string* arr = new string[10];
```

当使用new表达式时，实际执行了三步操作：

- new表达式调用operator new的标准库函数。该函数分配一块足够的、原始并未命名的内存空间以存储特定类型的对象。
- 编辑器运行相应的构造函数以构造这些对象，并传入初始值。
- 对象被被分配空间并构造完成，返回一个指向该对象的指针。

```cpp
delete sp;
delete[] arr;
```

使用delete表达式删除动态分配的对象时，实际执行了两步：

- 对sp所指的对象或者arr所指的数组中的元素执行相应的析构函数
- 编译器调用名为operator delete（或者operator delete[]）的标准库函数释放内存空间。

如果应用程序希望控制内存分配的过程，需要自定义operator new函数和operator delete函数。

当编译器发现一条new表达式或delete表达式后，将在程序中查找可调用的operator new或operator delete函数。如果该对象是类类型，编译器会先在类及其基类的作用域中查找，然后在全局作用域中查找。

可以使用作用域运算符执行相应作用域中的operator new 或 operator delete函数。

**operator new接口和operator delete接口**

标准库提供了不会抛出异常的接口

```cpp
// size_t 参数表示对象所需的字节数
void* operator new(size_t, nothrow_t&) noexcept;
```

类型nothrow_t 在头文件new中。该头文件还定义了nothrow的const对象，用户可以通过这个对象请求new 的非抛出版本。

当使用new 或delete 运算符函数定义成类的成员时，它们是隐式静态的。因为new用在对象构造前，delete用在对象销毁后，所以这两个成员必须是静态的，而且不能操纵类的任何数据成员。

```cpp
// 该函数只供标准库使用，不会被重载
void* operator new(size_t, void*);
```

如果基类有虚析构函数，则传递给operator delete的字节数因待删除指针所指向对象的动态类型不同而有区别。而且实际运行的operator delete函数版本也由对象的动态类型决定。

> new表达式的执行过程总是先调用operator new函数以获取内存空间，然后在得到的内存空间中构造对象。delete表达式的执行过程总是先销毁对象，然后调用operator delte函数释放对象所占空间。
>
> 无论如何，都不能改变new 运算符和delete运算符的基本含义

malloc 和 free函数定义在头文件stdlib中。

```cpp
// 简单示例
void* operator new(size_t size) {
    if(void* mem = malloc(size))
        return mem;
    else 
        throw bad_alloc();
}
void operator delete(void* mem) noexcept {free(mem);}
```

**定位 new表达式**

operator new和operator delete 函数的行为与allocate 和deallocate 类似，负责分配和释放内存空间，但是不会构造或销毁对象。不同的是，对于operator new分配的内存空间，用户无法使用construct 函数构造对象。应该使用定位new （placement new）形式构造对象。

placement new的形式如下

```cpp
new (place_address) type;
new (place_address) type (initializers);
new (place_address) type [size];
new (place_address) type [size] {braced initializer list};
```

placement new 不分配内存，只是简单地返回指针实参。它允许在一个特定的、预先分配的内存上构造函数。

placement new与allocator的construct成员的区别：传给construct的指针必须指向同一个allocator 对象分配的空间，但是传给placement new的指针无须指向operator new分配的内存，甚至不需要指向动态内存。

```cpp
// 调用析构函数会销毁对象，但是不会释放内存
string* sp = new string("a value");
sp->~string();
```

### 19.2 运行时类型识别

运行时类型识别（RTTI）功能由两个运算符实现：

- typeid运算符，用于返回表达式的类型
- dynamic_cast运算符，用于将基类的指针或引用安全转换为派生类的指针或引用

这两个运算符用于某种类型的指针或引用，并该类型含有虚函数时，运算符将使用指针或引用所绑定对象的动态类型。

**dynamic_cast运算符**

```cpp
dynamic_cast<type*>(e)
dynamic_cast<type&>(e)
dynamic_cast<type&&>(e)
```

e的类型必须符合以下条件中的任意一条：

- e的类型是目标type的公有派生类
- e的类型是目标type的公有有基类或就是类型type

如果指针类型并转换失败，则返回0，如果引用类型且失败，则抛出bad_cast异常

> 空指针执行dynamic_cast，返回空指针

**typeid运算符**

typeid运算符可以作用于任何类型的表达式，忽略顶层const，对数组或函数不会执行向指针的标准类型转换。

当运算对象不属于类类型或不包含任何虚函数的类时，typeid运算符指示的说运算对象的静态类型。当运算对象是定义了至少一个虚函数的类的左值时，typeid的结果直到运行时才会确定。

```cpp
Derived* dp = new Derived;
Base* bp = dp;
if(typeid(*bp) == typeid(*dp)) {
    // 相同对象
}
if(typeid(*bp) == typeid(Derived)) {
    // bp实际为为Derived对象
}
```

> 当typeid作用于指针本身时，返回该指针的静态编译时类型

只有当类型含有虚函数时，编译器才会对表达式求值，否则typeid返回表达式的静态类型；编译器无需对表达式求值也能知道表达式的静态类型。

### 19.3 枚举类型

```cpp
// 限定作用域的枚举类型
enum class open_modes {input, ouput, append};
// 不限定作用域的枚举类型
enum color {red, yellow, green};
// 未命名、不限定作用域的枚举类型
enum {floatPrec = 6, doublePrec = 10, double_doublePrec = 10};
```

不限定作用域的枚举类型中，枚举成员的作用域与枚举类型本身的作用域相同。

默认情况下，枚举值从0开始，一次加1，也可以执行专门的值。

枚举成员是const。不限定作用域的枚举类型的枚举成员可以隐式转换成int

```cpp
// 指定enum 的大小
enum intValues: unsigned long long {
    charTyp = 255;
}
```

对不限定作用域的枚举类型，其成员不存在默认类型，只知道成员的潜在类型足够大，能容纳枚举值。

```cpp
// 前置声明enum必须指定其成员的大小
enum intValues: unsigned long long;
// 限定作用域的枚举类型可以使用默认成员类型int
enum class open_modes;
```

### 19.4 类成员指针

成员指针是指可以指向类的非静态成员的指针。类的静态成员不属于任何对象。

```cpp
// 指向Screen类的const string成员的指针，取地址符作用域类的成员
const string Screen::*pdata = &Screen::contoents;
// 初始化成员指针或赋值时，该指针并没有指向任何数据。只有当解引用成员指针时才提供对象的信息。
Screen myScreen, *pScreen = &myScreen;
// *pdata 获得对象的contents成员
auto s = myScreen.*pdata;
s = pScreen->*pdata;
```

```cpp
// data 是静态成员，返回成员指针
class Screen {
public:
    statis const std::string Screen::* data() {
        return &Screen::contents;
    }
}
```

**成员函数指针**

成员函数和指向该成员的指针之间不存在自动转换规则

```cpp
// 需要显示使用&
auto pmf = &Screen::get;
```

指向成员的指针形参可以有默认实参

```cpp
// 可以对Action赋值格式相同的函数
using Action  = Screen& (Screen::*)();
```

与普通的函数指针不同，指向成员函数的指针要调用时需要将该指针绑定到特定的对象上，所以成员指针不是可调用对象，不支持函数调用运算符。

**使用function 生成可调用对象**

```cpp
function<bool (const string&)> fcn = &string::empty;
find_if(svec.begin(), svec.end(), fcn);
```

如果可调用对象是成员函数，则第一个形参必须表示该成员是在那个（一般是隐式的）对象上执行的。

```cpp
vector<string*> pvec;
function<bool (const string*)> fp = &string::empty;
find_if(pvec.begin(), pvec.end(), fp);
```

**使用mem_fn生成可调用对象**

mem_fn定义在functional头文件中，并且可以从成员指针生成一个可调用对象，mem_fn可以根据成员指针的类型推断可调用对象的类型，无须用户显示指定。

```cpp
find_if(svec.begin(), svec.end(), mem_fun(&string::empty));
```

可以认为mem_fn生成的可调用对象含有一对重载的函数调用运算符：一个接受string*，另一个接受string&。

**使用bind生成可调用对象**

```cpp
find_if(svec.begin(), svec.end(), bind(&string::empty, _1));
```

### 19.5 嵌套类

外层类的对象和嵌套类的对象是相互独立的。嵌套类的名字在外层类作用域中是可见的，在外层类作用域之外不可见。

嵌套类在其外层类中定义了一个类型成员。

嵌套类可以直接使用外层类的成员。

和之前一样，返回类型不在类的作用域中。

### 19.6 union: 一种节省空间的类

union可以有多个数据成员，但是在任意时刻只有一个数据成员可以有值。当给union的某个成员赋值后，该union的其他成员就变成未定义的状态。分配给一个union对象的存储空间至少要能容纳它的最大的数据成员。

union不能含有引用类型的成员。union可以为其成员指定public, protected, private等保护标记。默认情况下，union的成员都是公有的。union可以定义包括构造函数和析构函数在内的成员函数。但是union不能继承自其他类也不能作为基类所用，所以union不能含有虚函数。

```cpp
union Token{
    char cval;
    int ival;
    double dval;
}

Token t = {'a'};
```

**匿名union**

```cpp
union {
    char cval;
    int ival;
    double dval;
};
// 使用匿名union
cval = 'c'; 
ival = 42;
```

匿名union只能包含public成员，也不能定义成员函数。

**含有类类型成员的union**

当union包含的是内置类型的成员时，编译器将按照成员的次序依次合成默认构造函数或拷贝控制成员。如果union含有类类型的成员，并且该类型自定义了默认构造函数或拷贝控制成员，则编译器将为union合成对应的版本并将其声明为删除的。

**使用类管理union成员**

为了追踪union中存储了什么类型的值，通常会定义一个独立的对象，该对象被称为union的判别式。

```cpp
class Token {
public:
    // union含有stirng成员，所以Token必须定义拷贝控制成员
    Token(): tok(INT), ival{0} {};
    Token(const Token& t): tok(t.tok) { copyUnion(t);}
    Token& operator = (const Token&);
    
    // 如果union含有string成员，必须销毁它
    ~Token() {if (tok == STR) sval.~string();}
    
    Token& operator = (const string&);
    Token& operator = (char);
    Token& operator = (int);
    Token& operator = (double);
private:
    enum {INT, CHAR, DBL, STR} tok; // 判别式
    union {
        char cval;
        int ival;
        double dval;
        string sval;
    };
    // 检查判别式
    void copyUnion(const Token&);
};
```

**管理判别式并销毁string**

```cpp
Token& Token::operator=(int i) {
    if(tok == STR) sval.~string();
    ival = i;
    tok = INT;
    return *this;
}
```

**管理需要拷贝控制的联合成员**

```cpp
void Token::copyUnion(const Token &t) {
    switch(t.tok) {
        case Token::INT: ival = t.ival; break;
        case Token::CHAR: ival = t.cval; break;
        case Token::DBL: ival = t.dval; break;
        // 使用placement new
        case Token::STR: new(&sval) string(t.sval); break;
    }
}
```

```cpp
Token& Token::operator=(const Token &t) {
    if(tok == STR && t.tok != STR) sval.~string();
    if(tok == STR && t.tok == STR)
        sval = t.sval;
    else
        copyUnion(t); // 如果t.tok是STR，则需要构造一个string
    tok = t.tok;
    return *this;
}
```

### 19.7 局部类

定义在函数内部的类叫局部类。局部类的所有成员都必须完整定义在类的内部。局部类中也不允许声明静态数据成员。

局部类不能访问所在函数的普通局部变量。

### 19.8 固有的不可移植的特性

不可移植的特性是指因机器而异的特性。比如算数类型的大小在不同机器上不一样。

**位域**

位域在内存中的布局是与机器相关的带符号位域的行为是由具体实现确定的。

```cpp
typedef unsigned int Bit;
class File {
    Bit mode: 2; // 2 bit
    Bit modified: 1; // 1 bit
    Bit pro_owner: 3; // 3 bit
};

void File::write() {
    modified = 1;
}
void File::close() {
    if(modified) {}
}
```

可能的话，类内连续定义的位域压缩在同一整数的相邻位，能否压缩和如何压缩取决于机器。

取地址符不能作用于位域，任何指针都无法指向类的位域。

**volatile限定符**

volatile的确切含义与机器相关。

当对象的值可能在程序的控制或检测之外被改变时，应该将该对象声明为volatile，告诉编译器不应对这样的对象进行优化。

const和volatile 限定符互相没什么影响。只有volatile的成员函数才能被volatile的对象调用。与const 相同，volatile 的地址只能赋予指向 volatile 指针，引用类似。

```cpp
volatile int display_register;
int *volatile vip; // volatile 指针，指向int
```

const 和 volatile 的一个重要区别是，不能使用合成的拷贝/移动构造函数及赋值运算符初始化volatile 对象或从volatile对象赋值。合成的成员接受的形参类型是非volatile常量引用。

**链接指示：extern "C" **

对于其他语言编写的函数来说，编译器检查其调用的方式与处理普通C++函数的方式相同，但是生成的代码有所区别。C++用链接指示指出任意非C++函数所用的语言。

> 要把C++代码和其他语言编写的代码放在一起使用，要求有权访问该语言的编译器，并且这个编译器与当前的C++编译器是兼容的。

链接指示有两种形式：单个的、复合的。链接指示不能出现在类定义或函数定义的内部。同样的链接指示必须在函数的每个声明中都出现。

```cpp
// 单语句
extern "C" size_t strlen(const char*);
// 复合语句
extern "C" {
    int strcmp(const char*, const char*);
    char* strcat(char*, const char*);
}

extern "C" {
#include <string.h>
}
```

链接指示可以嵌套，因此头文件包含自带的链接指示的函数不受影响。

指向其他语言编写的函数的指针必须与函数本身使用相同的链接指示。指向C函数的指针与指向C++函数的指针是不一样的类型。

链接指示不仅对函数有效，而且对作为返回类型或形参类型的函数指针也有效。

```cpp
// f1是C函数，形参时指向C 函数的指针
extern "C" void f1(void(*)(int));
// FC 是指向C函数的指针
extern "C" typedef void FC(int);
```

通过链接指示对函数进行定义，可以令C++函数在其他语言编写的程序中可用

```cpp
// calc 函数可以被C 程序调用，编译器为该函数生成适合于指定语言的代码
extern "C" double calc(double dparm) {}
```

```cpp
// 链接到C 的预处理器支持
#ifdef __cplusplus
extern "C"
#endif
int strcmp(const char*, const char*);
```

链接指示与重载函数的相互作用依赖于目标语言。C语言不支持函数重载。如果一组重载函数中有一个是C函数，则其余必定都是C++函数。
