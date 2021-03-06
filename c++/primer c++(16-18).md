# Primer c++第五版笔记6(到第18章完)

~~这是笔记，所以别问为什么写的跟书上一摸一样了~~

[TOC]

## 16 模板与泛型编程

### 16.1 定义模板

#### 16.1.1 函数模板

```cpp
template <typename T>
int compare(T a, T b) {
    if(a < b) return -1;
    if(b < a) return 1;
    return 0;
}

int main() {
    cout << compare(1, -1) << endl;
    return 0;
}
```

**非类型模板参数**

通过特定的类型名而非关键字 class 或 typename 来指定非类型参数

```cpp
// 处理字符串字面常量
template<unsigned N, unsigned M>
int compare(const char (&p1)[N], const char (&p2)[M]) {
    return strcmp(p1, p2);
}
int main() {
    cout << compare("hello", "world") << endl;
    return 0;
}
```

绑定到非类型参数的实参必须是一个常量表达式。绑定到指针或引用非类型参数的实参必须具有静态的生存期。不能使用动态对象作为指针或引用非类型模板参数的实参。

**inline和constexpr的函数模板**

```cpp
template<typename T> inline T min(const T&, const T&);
```

**编写类型无关的代码**

编写泛型代码的重要原则

* 模板中的函数参数是const 的引用
* 函数体中的条件判断仅使用< 比较运算

函数参数为const的引用能保证函数可以用于不能拷贝的类型

**模板编译**

当编译器遇到模板定义时，并不生成代码。只有实例化出模板的一个特定版本时，编译器才会生成代码。

函数模板和类模板成员函数的定义通常放在头文件中

#### 16.1.2 类模板

与函数模板的不同之处是，编译器不能为类模板推断模板参数类型

```cpp
template <typename T>
class Blob {};
// 实例化类模板，编译器会实例化出等价的类
Blob<int> ia;
//template <>
//class Blob<int>{};
```

> 一个类模板的每个实例都形成一个独立的类

**在模板作用域中引用模板类型**

类模板用来实例化类型，一个实例化的类型总是包含模板参数的。

```cpp
std::shared_ptr<std::vector<T>> data;
```

**类模板的成员函数**

既可以在类模板内部，也可以在外部定义成员函数，且在内部的成员函数被隐式声明为内联函数。

类模板的成员函数本身是一个普通函数。但是类模板的每个实例都有自己版本的成员函数，因此定义在类模板之外的成员函数必须以关键字template开始，后接类模板参数列表。

```cpp
template <typename T>
class Blob {
    public:
        T& f(T& a);
    };
template <typename T>
T& Blob<T>::f(T& a) {return a;}
```

**类模板成员函数的实例化**

默认情况下，对于一个实例化了的类模板，其成员只有在使用时才会被实例化。这一特性使得即使某种类型不能完全符合模板操作的要求，仍然能用该类型实例化类

**在类模板的作用域中，可以直接使用模板名**

```cpp
template <typename T>
class BlobPtr {
public:
    // 不必使用 BlobPtr<T>&
    BlobPtr& operator++();
    BlobPtr& operator--();
};
```

在一个类模板的作用域内，可以直接使用模板名而不必指定模板实参

**类模板和友元**

如果类模板包含一个非模板友元，则友元可访问所有模板实例。如果友元自身时模板，类可以授权给所有友元模板实例，也可以之授权给特定实例

```cpp
// 一对一友好关系
// 为了引用模板的一个特定实例，必须先声明模板自身。
template <typename> class BlobPtr;
template <typename> class Blob;
template <typename T>
bool operator == (const Blob<T>&, const Blob<T>&);

template <typename T>
class Blob {
public:
    friend class BlobPtr<T>;
    friend bool operator == <T>(const Blob<T>&, const Blob<T>&);
};
```

先将Blob、BlobPtr 和operator==声明为模板。这些声明时operator==函数的参数声明以及Blob中的友元声明所需要的。

友元的声明用Blob的模板形参作为他们自己的模板实参。因此友好关系被限定在用相同类型实例化的Blob与BlobPtr相等运算符之间

```cpp
// 前置声明，将模板的一个特例声明为友元时用到
template<typename T> class Pal;
 
class C { //非模板类 
    friend class Pal<C>; // 用类C实例化的Pal是C的一个友元
    // Pal2 的所有实例都是C 的友元，这种情况无需前置声明
    template<typename T> friend class Pal2;
};

template <typename T> 
class C2 { // 模板类
    // C2的每个实例将相同实例化的Pal声明为友元
    friend class Pal<T>;
    
    // Pal2的所有实例都是C的每个实例的友元，不需要前置声明
    template <typename X> friend class Pal2;
    
    // Pal3是一个非模板类，它是C2 所有实例的友元
    friend class Pal3;
};
```

**令模板自己的类型参数成员友元**

```cpp
template <typename Type> 
class Bar {
    friend Type;
};
```

**模板类型别名**

可以定义typedef来引用实例化的类，但是typedef不能引用模板

新标准中可以定义类模板的类型别名

```cpp
template <typename T> using twin = pair<T, T>;
twin<string> a; // pair<string, string>
template <typename T> using partNo = pair<T, unsigned>;
partNo<string> books; // pair<string, unsigned>
```

**类模板的static 成员**

```cpp
template <typename T>
class Foo {
public:
    static std::size_t count() {return ctr;}
private:
    static std::size_t ctr;
};

template <typename T>
size_t Foo<T>::ctr = 0;
```

对于每个Foo 的实例都有其自己的static 成员实例。所有Foo\<X> 类型的对象共享相同的ctr对象和 count 函数。因此访问static成员时需要特定实例

static成员函数只有在使用时才会实例化

#### 16.1.3 模板参数

**模板声明**

一个特定文件所需的所有模板的声明通常一起放置在文件开始位置，出现于任何使用这些模板的代码之前

**使用类的类型成员**

默认情况下，C++假定通过作用域运算符访问的名字不是类型，所以要显式表示类型，可以通过关键字typename实现

```cpp
template <typename T>
typename T::value_type top(const T& c) {
    if(!c.empty())
        return c.back();
    else 
        return typename T::value_type();
}
```

显式表示类型时，必须使用关键字typename，不能使用class

**默认模板实参**

```cpp
// 使用默认模板实参less<T>和默认函数实参F()
template <typename T, typename F = less<T> >
int compare(const T& v1, const T& v2, F f = F()) {
    if(f(v1, v2)) return -1;
    if(f(v2, v1)) return 1;
    return 0;
}
```

与函数默认实参一样，对于一个模板参数，只有当它右侧的所有参数都有默认实参时，它才可以有默认实参

**模板默认实参与类模板**

模板名之后必须接上尖括号，尖括号指出类必须从一个模板实例化而来。如果类模板的所有模板参数都提供了默认实参，且希望使用这些默认实参，必须在模板名之后跟空尖括号对

```cpp
template<typename T = int>
class Numbers {
public:
    Numbers(T v = 0): val(v) {}
private:
    T val;
};
Numbers<long double> lots;
Numbers<> average; // 使用默认类型
```

#### 16.1.4 成员模板

模板类还是普通类可以包含本身是模板的成员函数。这种成员被称为成员模板。成员模板不能是虚函数

**普通类的成员模板**

```cpp
// 定义类似unique_ptr 使用的默认删除器类型
// 类将包含一个重载的函数调用运算符
// 它接受一个指针并对此指针执行delete
class DebugDelete{
public:
    DebugDelete(std::ostream& s = std::cerr): os(s) {}
    template<typename T>
    void operator()(T* p) const {
        os << "deleting unique_ptr" << std::endl;
        delete p;
    }
private:
    std::ostream &os;
}
int main() {
    double* p = new double;
    DebugDelete d;
    d(p); // 调用DebugDelete::operator()(double*)
    int* ip = new int;
    // 在临时对象上嗲用operator()(int*)
    DebugDelete()(ip);

    // 实例化DebugDelete::operator()<string>(string*>
    unique_ptr<int, DebugDelete> up(new int, DebugDelete());
    // 实例化DebugDelete::operator()<string>(string*)
    unique_ptr<string, DebugDelete> sp(new string, DebugDelete());
    return 0;
}
```

unique_ptr 的析构函数会调用DebugDelete 的调用运算符。因此，unique_ptr 的析构函数实例化时，DebugDelete的调用运算符都会实例化

```cpp
// DebugDelete 的成员模板实例化
void DebugDelete::operator()(int* p) const {delete p;}
void DebugDelete::operator()(string* p) const {delete p;}
```

**类模板的成员模板**

```cpp
template<typename T>
class Blob {
public:
    template<typename It>
    Blob(It b, It e);
};
template<typename T>
template<typename It>
Blob<T>::Blob(It b, It e) {}
```

成员模板是函数模板。当在类模板外定义一个成员模板时，必须同时为类模板和成员模板提供模板参数列表。类模板的参数列表在前，后跟成员自己的模板参数列表

**实例化与成员模板**

调用成员模板时，编译器根据该对象的类型来推断类模板参数的实参，根据传递给成员模板的函数实参来推断它的模板实参

```cpp
int main() {
    int ia[] = {0, 1, 2, 3};
    vector<long> vi = {0, 1, 2, 3};
    list<const char*> w = {"Hello", "World"};
    // 实例化Blob<int>类及其接受两个int*参数的构造函数
    Blob<int> a1(begin(ia), end(ia));
    Blob<int> a2(vi.begin(), vi.end());
    Blob<string> a3(w.begin(), w.end());
    return 0;
}
```

#### 16.1.5 控制实例化

模板只有使用时才会实例化，这意味相同的实例可能出现在多个对象文件中，多个文件中实例化相同模板会产生额外开销。新标准中可以通过**显式实例化**来避免这种开销

```cpp
extern template declaration;// 实例化声明
template declaration; // 实例化定义
```

将一个实例化声明为extern就表示承诺在程序其他位置有该实例化的一个非extern声明（定义）。对于一个给定的实例化版本，可能有多个extern声明，但必须只有一个定义。

```cpp
// Application.cc
// 模板类型还没实例化
extern template class Blob<string>;
extern template int compare(const int&, const int &);
Blob<string> sa1, sa2; // 实例化会出现在其他地方

Blob<int> a1 = {0, 1, 2}; // 实例化Blob<int> 及其接受initializer_list的构造函数
Blob<int> a2(a1); // 实例化拷贝构造函数
int i = compare(a1[0], a2[0]); // 实例化出现在其他地方
```

文件Application.o将包含Blob\<int>的实例及其接受initializer_list参数的构造函数和拷贝构造函数的实例。

```cpp
// templateBuild.cc
// 定义
template int compare(const int&, const int &);
template class Blob<string>; // 实例化类模板的所有成员
```

文件templateBuild.o 将包含 compare的int实例化版本的定义和Blob\<string>类的定义，编译此应用程序时，必须将templateBuild.o 和 Application.o 链接在一起。

**实例化定义会实例化所有成员**

类模板的实例化会实例化该模板的所有成员，包括内联的成员函数。因此用来显式实例化一个类模板的类型，所用类型必须能用于模板的所有成员函数

#### 16.1.6 效率与灵活性

**在运行时绑定删除器**

shared_ptr的删除器可以在reset指针时修改，所以删除器必须保存为指针或封装了指针的类。假定shared_ptr将它管理的指针保存在一个成员p中，删除器是通过一个名为del 的成员来访问的，则shared_ptr的析构函数必须包含类似下面的语句

```cpp
// del的值只有运行时才知道
del ? del(p) : delete p; // del(p)需要运行时跳转到del的地址
```

由于删除器是间接保存的，调用del(p) 需要一次运行时的跳转操作， 转到del中保存的地址来执行对应的代码

**在编译时绑定删除器**

unique_ptr的删除器类型必须在定义unique_ptr 时显式提供，即删除器的类型是类类型的一部分，因此删除其成员的类型是在编译时是知道的，删除其可以直接保存在unique_ptr对象中。

```cpp
// unique_ptr的析构函数
// del 在编译时绑定；直接调用实例化的删除器
del(p); // 无运行时额外开销
```

通过在编译时绑定删除器，unique_ptr避免了间接调用删除器的运行时开销。通过在运行时绑定删除器，shared_ptr 使用户重载删除器更为方便

### 16.2 模板实参推断

从函数实参来确定模板实参的过程被称为模板实参推断

#### 16.2.1 类型转换与模板类型参数

编译器通常不是对实参进行类型转换，而是生成一个新的模板实例。

顶层const无论在形参中还是实参中，都会被忽略。能应用于函数模板的类型转换包括

- const 转换：将非 const 对象的引用（或指针）传递给一个const 的引用（或指针）形参
- 数组或函数指针转换：如果函数形参不是引用类型，则可以对函数或函数类型的实参应用正常的指针转换。

其他类型转换，如算术转换、派生类向基类的转换以及用户定义的转换都不能应用于函数模板。

```cpp
template<typename T> T fobj(T, T); // 实参被拷贝
template<typename T> T fref(const T&, const T&); // 引用

int main() {
    string s1("Hello");
    const string s2("World");
    fobj(s1, s2); // 调用fobj(string, string) const被忽略
    fref(s1, s2); // 调用fref(const string&, const string&)，将s1转换为const是允许的
    
    int a[10], b[42];
    fobj(a, b); // 调用f(int*, int*)
//    fref(a, b); 数组类型不匹配
    return 0;
}
```

第一对调用中，顶层 const 被忽略。第三对调用中，两个数组都被转换成指针。第四对调用不合法，因为形参是一个引用，则数组不会转换为指针，a和b的类型不匹配。

> 将实参传递给带模板类型的函数形参时，能够自动应用的类型转换只有const 转换及数组或函数到指针的转换

**正常类型转换应用于普通函数实参**

```cpp
template<typename T>
ostream &print(ostream& os, const T& obj) {
    return os << obj;
}
int main() {
    print(cout, 42);
    ofstream f("output");
    print(f, 10); // 使用print(ostream&, int)
    return 0;
}
```

第二个调用中，第一个实参为ofstream，可以转换为ostream&，此参数类型不依赖模板参数，因此编译器会将f 隐式转换为ostream&

> 如果函数参数类型不是模板参数，则对实参进行正常的类型转换

#### 16.2.2 函数模板显式实参

**指定显式模板实参**

```cpp
// 没有任何函数实参的类型可用来推断T1 的类型
// 每次调用时需显式提供模板实参 T1
template<typename T1, typename T2, typename T3>
T1 sum(T2, T3);
// T1是显式指定的，T2和T3是从函数实参类型推断而来的
int i = 1;long long lng = 3;
auto val3 = sum<long long>(i, lng);
```

显式模板实参按由左到右的顺序与对应的模板参数匹配。只有尾部（最右）参数的显示模板实参才可以忽略，而且前提是它们可以从函数参数推断出来。

**正常类型转换应用于显式指定的实参**

对于用普通类型定义的函数参数，允许进行正常的类型转换，出于相同的原因，对于模板类型参数已经显式指定了的函数实参，也进行正常的类型转换。

```cpp
template<typename T>
bool compare(T, T);
long lng;
int main() {
    // compare(lng, 1024); 错误，模板参数不匹配
    compare<long>(lng, 1024); // 实例化 compare(long, long)
    compare<int>(lng, 1024); // 实例化 compare(int, int)
    return 0;
}
```

#### 16.2.3 尾置返回类型与类型转换

```cpp
template<typename It>
auto fcn(It beg, It end) -> decltype(*beg) {
    return *beg; // 返回序列中一个元素的引用
}
// 不知道返回结果的准确类型，但直到所需类型是所处理的序列的元素类型
vector<int> vi = {1, 2, 3};
vector<string> ca = {"Hello", "World"};
auto& i = fcn(vi.begin(), vi.end()); // 返回int&
auto& s = fcn(ca.begin(), ca.end()); // 返回string&
```

 知道函数应该返回\*beg，而且用decltype(\*beg)来获取表达式的类型。但是在编译器遇到函数的参数列表之前，beg都是不存在的，所以使用尾置返回类型。

**进行类型转换的标准库模板类**

标准库的**类型转换**模板的头文件在type_traits中，这个头文件中的类通常用于所谓的模板元程序设计。

可以使用remove_reference类获取元素类型。如果用引用类型实例化remove_reference，则type将表示被引用的类型。

```cpp
// 更一般的，给定一个迭代器beg
remove_reference<decltype(*beg)>::type
```

decltype(*beg) 返回元素类型的引用类型，remove_reference::type 去除引用，剩下元素类型本身。

```cpp
// 在函数中返回元素值的拷贝
// 为了使用模板参数的成员，必须使用typename
template<typename It>
auto fcn(It beg, It end) -> typename remove_reference<decltype(*beg)>::type {
    return *beg;
}
```

> type是一个类成员，依赖于模板参数，所以必须在返回类型的声明中使用typename来告知编译器，type表示一个类型

#### 16.2.4 函数指针和实参推断

当使用一个函数模板初始化一个函数指针或为一个函数指针赋值时，编译器使用指针的类型来推断模板实参。

```cpp
template<typename T>
int compare(const T&, const T&);
// pf1 指向实例 int compare(const int&, const int&)
int (*pf1)(const int&, const int&) = compare;

void func(int(*) (const string&, const string&));
void func(int(*) (const int&, const int&));

int main() {
//    func(compare); // 错误，不确定使用哪个compare实例
    func(compare<int>); // 显式指出实例化哪个compare版本
    return 0;
}
```

> 当参数是一个函数模板实例的地址时，程序上下文必须满足：对于每个模板参数，能唯一确定其类型或值

#### 16.2.5 模板实参推断和引用

```cpp
template<typename T> void f(T& p);
```

> 编译器会应用正常的引用绑定规则：const 是底层的，不是顶层的

**从左值引用函数参数推断类型**

当函数参数是模板类型参数的普通左值引用时（即T&），实参只能是一个左值。如果实参时const，则 T 将被推断为const类型，无法传递字面常量值。

如果函数参数的类型是const T&，则可以传递给它任何类型的实参。当函数参数本身为const 时，T 的类型推断结果不会是const类型。

**从右值引用函数参数推断类型**

```cpp
template<typename T> void f3(T&&);
f3(42); // 实参是一个int 类型的右值，模板参数T 是int
```

**引用折叠和右值引用参数**

c++语言在正常绑定规则之外定义了两个例外规则：

* 第一个例外规则影响右值引用参数的推断如何进行。当一个左值传递给函数的右值引用参数，且此右值引用指向模板类型参数(如T&&)时，编译器推断模板类型参数为实参的左值引用类型。因此，当调用f3(i) 时，编译器推断T 的类型为int&&，而非int。

  T被推断为 int& 好像意味着f3 的函数参数应该是一个类型 int& 的右值引用。通常不能（直接）定义一个引用的引用。但是通过类型别名或模板参数类型间接定义时可以的。

* 第二个例外绑定规则：如果间接创建一个引用的引用，则这些引用形成了“折叠”，除第一个例外，引用会折叠成一个普通的左值引用类型。在新标准中，只有右值引用的右值引用会折叠成右值引用。即对于给定类型X：

  * X& &, X& && 和 X&& &都折叠成类型X&
  * 类型X&& &&折叠成X&&

> 引用折叠只能应用于间接创建的引用的引用，如类型别名或模板参数

```cpp
f3(i); // 实参是一个左值；模板参数T是int&
f3(ci);// 实参是一个左值；模板参数T是const int&
```

> 这意味着，如果函数参数是指向模板参数类型的右值引用（T&&）,则可以传递给它任意类型的实参

**编写接受右值引用参数的模板函数**

```cpp
template<typename T>
void f3(T&& val) {
    T t = val; // 拷贝还是绑定一个引用
    t = fcn(t); // 赋值只改变t 还是t 和val 都会改变
    if(val == t) {} // 若T 是引用类型，则恒为true
}
```

如果传参为字面常量1，T为int，因此局部变量t的类型为int。对t 赋值时，参数val保持不变。

如果传参为左值i，则T为int&，因此局部变量t 的类型为int&。对t赋值时val 的值也改变了，if判断恒为true。

实际上，右值引用通常用于两种情况：模板转发实参，或模板被重载。要注意的是，使用右值引用的函数模板通常这样进行重载

```cpp
template<typename T> void f(T&&); // 绑定到非 const 右值
template<typename T> void f(cosnt T&); // 左值和const 右值
```

#### 16.2.6 理解std::move

```cpp
// move 的定义
template<typename T>
typename remove_reference<T>::type&& move(T&& t) {
    return static_cast<typename remove_reference<T>::type&&>(t);
}

string s1("hi!"), s2;
s2 = std::move(string("bye!")); // 从右值移动数据
s2 = std::move(s1); // 在赋值后s1的值是不确定的
```

**std::move 是如何工作的**

```cpp
//在第一个赋值中，该调用实例化为
string&& move(string&& t) {
    return static_cast<string&&>(t);
}
// 第二个赋值中，T的类型为string&, remove_reference<T>::type 为string，t的类型为string&
// 实例化为
string&& move(string& t) {
    return static_cast<string&&>(t);
}
```

**从一个左值static_cast 到一个右值引用是允许的**

#### 16.2.7 转发

```cpp
template<typename F, typename T1, typename T2>
void flip(F f, T1 f1, T2) {
    f(t2, t1);
}
```

这个函数在调用接受引用参数时就会出现问题

```cpp
void f(int v1, int& v2);

flip(f, j, 42); // j 不会被引用
// 此时会被实例化为
void flip(void(*fcn)(int, int&), int, int);
```

**定义能保持类型信息的函数参数**

为了能保持参数的所有性质：包括const，左值还是右值。可以将函数参数定义为指向模板类型参数的右值引用，就可以解决左值/右值属性；由于const时底层的，所以使用引用参数可以保持const属性；

```cpp
template<typename F, typename T1, typename T2>
void flip(F f, T1&& t1, T2&& t2) {
    f(t2, t1);
}
// 至此，该函数对于字面值（右值）的处理还是有问题
void g(int&& i, int& j);
flip(g, i, 42); // T2 类型为 int，不能从一个左值实例化int&&
```

**使用std::forward保持类型信息**

forward定义在头文件utility中，forward必须通过显示模板实参调用，返回该显示实参类型的右值引用。

通常情况下，使用forward 传递那些定义为模板类型参数的右值引用的函数参数。通过其返回类型上的引用折叠，forward 可以保持给定实参的左值/右值属性。

如果实参是右值，则T 是一个普通类型，forward\<T>返回T&&，如果T 为左值，返回类型是一个指向左值引用类型的右值引用，再将forward\<T>的返回类型进行引用折叠，将返回一个左值引用。

> 当用于一个指向模板参数类型的右值引用函数参数T&&时，forward会保持实参类型的所有细节

```cpp
template<typename F, typename T1, typename T2>
void flip(F f, T1&& t1, T2&& t2) {
    f(std::forward<T2>(t2), std::forward<T1>(t1));
}
```

### 16.3 重载与模板

函数模板可以被另一个模板或普通非模板函数重载。名字相同的函数必须具有不同数量或类型的参数。如果涉及函数模板，则函数匹配规则会有如下影响

* 对于一个调用，其候选函数包括所有模板实参推断成功的函数模板实例。
* 候选的函数模板总是可行的。
* 可行函数（模板与非模板）按类型转换（如果需要的话）来排序。可用于函数模板调用的类型转换是有限的。
* 函数有多个匹配选择时，匹配优先级为：
  - 同样好的函数中，选择非模板函数。
  - 同样好的函数中，只有函数模板可选，则选择更特例化的。
  - 否则，此调用有歧义。

> 正确定义一组重载的函数模板需要对类型间的关系及模板函数允许的有限的实参类型转换有深刻的理解

**编写重载模板**

```cpp
// 通用版本
template<typename T>
string debug_rep(const T& t) {
    ostringstream ret;
    ret << t;
    return ret.str(); // 返回ret 绑定的string副本
}

// 定义打印指针的版本
// 不能打印字符指针。因为IO库为char*值定义了<< 版本，此版本不打印地址值
template<typename T>
string debug_rep(T* p) {
    ostringstream ret;
    ret << "pointer: " << p;
    if(p) {
        ret << " " << debug_rep(*p); // 打印 p指向的值
    } else {
        ret << " null pointer";
    }
    return ret.str();
}
```

**多个可行模板**

```cpp
string s = "HelloWrold";
const string* sp = &s;
debug_rep(sp);

// 此时两个模板分别被实例化为
debug_rep(const string*&);
debug_rep(const string*);
```

一般的函数匹配规则无法区分这两个函数，将是有歧义的。但是根据重载函数模板的特殊规则，此调用被解析为debug_rep(const string*)，即更特化的版本。

设计这条规则的原因是，没有它，将无法对一个const的指针调用指针版本的debug_rep。因为模板debug_rep(const T&) 本质上可以用于任何类型。没有这条规则，传递const 的指针的调用永远是有歧义的。

**非模板和模板重载**

```cpp
string debug_rep(const string& s) {
    return '"' + s + '"';
}
string s("hi");
debug_rep(s); // 调用非模板函数 
```

**重载模板和类型转换**

```cpp
debug_rep("hi world!");
```

对于这个调用有三个版本可行

- debug_rep(const T&);
- debug_rep(T*);
- debug_rep(const string&)

对于非模板版本是可行的，但是需要进行一次用户定义的类型转换，因为它没有精确匹配那么好。因为T*版本更加特例化，所以编译器选择它。如果希望字符指针能按string处理，可以定义另外两个非模板重载版本

```cpp
string debug_rep(char* p) {
    return debug_rep(string(p));
}
string debug_rep(const char* p) {
    return debug_rep(string(p));
}
```

**缺少声明可能导致程序行为异常**

```cpp
template<typename T> string debug_rep(const T& t);
template<typename T> string debug_rep(T* p);
// 为了使debug_rep(char*)的定义正确工作，下面的声明必须在作用域中
string debug_rep(const string& );
string debug_rep(char* p) {
    // 如果接受一个const string& 的版本的声明不在作用域中
    // 返回语句将调用debug_rep(const T&)的T 实例化为string 的版本
    return debug_rep(string(p));
}
```

> 在定义任何函数之前，最好声明所有重载的函数版本。这样可以避免实例化别的版本

### 16.4 可变参数模板

可变参数模板就是接受可变数目参数的模板函数或模板类。可变数目的参数被称为参数包。存在两种参数包：模板参数包；函数参数包。

```cpp
// Args是模板参数包；rest是函数参数包
// Args表示零个或多个模板类型参数
// rest便是零个或多个函数参数
template<typename T, typename... Args>
void foo(const T& t, const Args&... rest);

int i = 0; double d = 3.14; string s = "hello";
foo(i, s, 42, d);
// 实例化出
void foo(const int&, const string&, const int&, const double&);
```

**sizeof... 运算符**

```cpp
// 使用sizeof... 运算符获取包中元素数目
template<typename... Args>
void g(Args... args) {
    cout << sizeof...(Args) << endl; // 类型参数的数目
    cout << sizeof...(args) << endl; // 函数参数的数目
}
```

#### 16.4.1 编写可变参数函数模板

可变参数函数通常是递归的。第一步调用处理包中的第一个参数，然后用剩余实参调用自身。print函数也是这样的模式，每次递归调用将第二个实参打印到第一个实参表示的流中。为了终止递归，还需要定义一个非可变参数的print函数：

```cpp
// 用来终止递归并打印最后一个元素的函数
// 此函数必须在可变参数版本的print定义之前声明
template<typename T>
ostream& print(ostream& os, const T& t) {
    return os << t; // 包中最后一个元素之后不打印分隔符
}
// 包中除了最后一个元素之外，其他元素都会调用这个版本的print
template<typename T, typename... Args>
ostream& print(ostream& os, const T& t, const Args&... rest) {
    os << t << ", ";
    return print(os, rest...);
}
```

可变参数版本的print的调用只传递了两个实参。结果是rest中的第一个实参被绑定到t，剩下实参形成下一个print调用的参数包。

```cpp
print(cout, i, s, 42); // 包中有两个参数
```

会递归调用。

| 调用                                          | t    | rest ... |
| --------------------------------------------- | ---- | -------- |
| print(cout, i, s, 42);                        | i    | s, 42    |
| print(cout, s, 42);                           | s    | 42       |
| print(cout, 42); // 调用非可变参数版本的print |      |          |

前两个调用只能与可变参数版本匹配。最后一次递归调用更特例化的非可变参数版本。

> 当定义可变参数版本的print 时，非可变参数版本的声明必须在作用域中。否则将无限递归

#### 16.4.2 包扩展

扩展包时，需要提供用于每个扩展元素的模式。扩展包就是将它分解为构成的元素，对每个元素应用模式，获得扩展后的列表。通过在模式右边放省略号(...)来出发扩展操作。

```cpp
template<typename T, typename... Args>
ostream& print(ostream& os, const T& t, const Args&... rest){ // 扩展Args
    os << t << ", ";
    rerturn print(os, rest...); // 扩展rest
}
```

第一个扩展操作扩展模板参数包，为print生成函数参数列表。第二个扩展操作为print调用生成实参列表。

对Args的扩展中，编译器将模式 const Args& 应用到模板参数包Args 中的每个元素。因此，此模式的扩展结果是逗号分隔的零个或多个类型的列表，每个类型都形如const type&。

第二个扩展中，模式是函数参数包的名字（rest）。此模式扩展出一个由包中元素组成的、逗号分隔的列表。因此该调用等同于 print(os, s, 42);

**理解包扩展**

```cpp
template<typename... Args>
ostream &errorMsg(ostream& os, const Args&... rest) {
    // print(os, debug_rep(a1), debug_rep(a2), ..., debug_rep(an));
    return print(os, debug_rep(rest)...);
}
```

该printprint调用使用了模式debug_reg(rest)。此模式表示希望对函数参数包rest中的每个元素调用debug_rep。扩展结果将使用逗号分隔的debug_rep调用列表。

> 扩展中的模式会独立的应用在包中的每个元素

#### 16.4.3 转发参数包

设计一个emplace_back成员，希望使用string的移动构造函数，还需要保持传递给emplace_back的实参的所有类型信息。

```cpp
class StrVec {
public:
    template<typename... Args> void emplace_back(Args&& ...);
}
template<typename... Args>
inline void StrVec::emplace_back(Args&&... args) {
    chk_n_alloc(); // 如果需要的话重新分配StrVec内存空间
    alloc.construct(first_free++, std::forword<Args>(args)...);
}
```

std::forward\<Args>(args)... 既扩展了模板参数包Args，也扩展了函数参数包args。此模式生成如下形式的元素 std::forward\<T_i>(t_i) 

### 16.5 模板特例化

**定义函数模板特例化**

特例化函数模板时，必须为原模板中的每个模板参数都提供实参。为了指出正在实例化一个模板，应使用关键字template\<>。空尖括号指出将为原模板的所有模板参数提供实参。

```cpp
template<typename T>
int compare(const T&, const T&);
// compare的特殊版本，处理字符数组的指针
template<>
int compare(const char* const& p1, const char* const& p2) {
    return strcmp(p1, p2);
}
```

当定义一个特例化版本时，函数参数类型必须与一个先前声明的模板中对应的类型匹配。

**函数重载与模板特例化**

当定义函数模板的特例化版本时，本质上接管了编辑器的工作。一个特例化版本本质上是一个实例，而非函数名的重载版本。因此特例化不影响函数匹配。

> 普通作用于规则应用于特例化
>
> 模板及其特例化版本应该声明在同一个头文件中。所有同名模板的声明应该放在前面，然后是这些模板的特例化版本

**类模板特例化**

作为例子，将为标准库hash模板定义一个特例化版本。特例化hash类必须定义：

- 重载的调用运算符，接受容器关键字类型，返回size_t
- 两个类型成员，result_type 和 argument_type，分别调用运算符的返回类型和参数类型
- 默认构造函数和拷贝赋值运算符（隐式定义）

```cpp
class Sales_data {
public:
    string bookNo; unsigned units_sold; double revenue;
};
namespace std {
template<> struct hash<Sales_data> {
    // 使用默认拷贝控制成员和构造函数
    typedef size_t result_type;
    typedef Sales_data argument_type; // 默认情况下，此类型需要==
    size_t operator()(const Sales_data& s) const;
};
size_t hash<Sales_data>::operator()(const Sales_data &s) const {
    return  hash<string>()(s.bookNo) ^
            hash<unsigned>()(s.units_sold) ^
            hash<double>()(s.revenue);
}
}
```

template\<>指出正在全特化模板。

**类模板部分特例化**

类模板的特例化不必为所有模板参数提供实参。一个类模板的部分特例化本身是一个模板。

> 只能部分特化类模板，而不能部分特化函数模板

```cpp
// 原始通用的版本
template<class T> struct remove_reference {
    typedef T type;
};
// 部分特化，用于左值引用和右值引用
template<class T> struct remove_reference<T&> {
    typedef T type;
};
template<class T> struct remove_reference<T&&> {
    typedef T type;
};
```

在类名之后，为要特例化的模板参数指定实参，这些实参列于模板名之后的尖括号中，参数按位置与原始模板对应。

```cpp
int i;
// int，使用原始模板
remove_reference<decltype(42)>::type a;
// int&，使用第一个部分特例化版本
remove_reference<decltype(i)>::type b;
// int&&，使用第二个部分特例化版本
remove_reference<decltype(std::move(i))>::type c;
```

**特例化成员而不是类**

可以只特例化成员函数而不是特例化整个模板。

```cpp
template<typename T>
struct Foo {
    Foo(const T& t = T()):mem(t) {}
    void Bar();
    T mem;
};
template<> void Foo<int>::Bar(); // 特例化成员
```

## 17 标准库特殊设施

### 17.1 tuple类型

tunple 是类似pair 的模板，但一个tuple 可以有任意数量的成员。tunple类型及其伴随类型和函数都定义在tuple 头文件中

**表17.1 tuple支持的 操作**

|                                             |                                                              |
| ------------------------------------------- | ------------------------------------------------------------ |
| tuple\<T1, T2, ..., Tn> t;                  | 声明tuple类型，所有成员都进行值初始化                        |
| tuple\<T1, T2, ..., Tn> t(v1, v2, ..., vn); | 初始化tuple，此构造函数是explicit                            |
| make_tuple(v1,  v2, ..., vn)                | 返回初始化的tuple                                            |
| t1 == t2                                    | 对所有成员逐一比对，当且仅当元素数量相等且所有成员对应相等时为true |
| t1 != t2                                    | 和上类似                                                     |
| t1 relop t2                                 | 两个tuple必须具有相同数量的成员。使用 < 运算符对成员逐一比较 |
| get\<i>(t)                                  | 返回t 的第i个数据成员的引用。tuple的所有成员都是public       |
| tuple_size\<tupleType>::value               | 类模板，获取tuple类型中成员的数量                            |
| tuple_element\<i, tupleType>::type          | 类模板，public成员type表示给定tuple类型中指定成员的类型      |

```cpp
int main() {
    // 默认初始化
    tuple<size_t, size_t, size_t> threeD;
    auto item = make_tuple(1, "hello", 3.1);
    auto i = get<0>(item); // 1

    typedef decltype(item) trans;
    size_t sz = tuple_size<trans>::value;
    tuple_element<1, trans>::type s = get<1>(item); // "hello"
    std::cout << i << " " << s << " " << sz<< endl; // 1 hello 3
    return 0;
}
```

只有两个tuple具有相同数量的成员时，才可以逐一比较它们，而且每对成员使用==运算符和 < 运算符必须是合法的。

### 17.2 bitset类型

biset 类用于位运算，定义位于头文件bitset中。编号为0的为低位。

```cpp
// 定义
biset<32> bitvec(1U); // 32位，低位为1，其他位为0
```

**表17.2 初始化bitset的方法**

|                                      |                                                              |
| ------------------------------------ | ------------------------------------------------------------ |
| bitset\<n> b;                        | b有n位；每位为0。构造函数是constexpr                         |
| bitset\<n> b(n);                     | 初始化                                                       |
| bitset\<n> b(s, pos, m, zero, one);  | b是string s从位置pos开始m个字符的拷贝。s只能包含字符0或1；包含其他字符会抛出invalid_argument异常。字符在b中分别保存为0和1。 |
| bitset\<n> b(cp, pos, m, zero, one); | 同上。cp表字符数组，若未提供m，则cp必须指向C风格字符串       |

用整型值来初始化bitset时，此值被转换为unsigned long long类型并被当做位模式来处理。biset默认值为0，高位超出给定值则被丢弃。

从string或字符数组指针来初始化bitset时，字符串中下标最小的字符对应高位

```cpp
bitset<32> bitvec4("1100"); // 二进制 1100
```

**表17.3 bitset操作**

|                        |                           |
| ---------------------- | ------------------------- |
| b.any()                | b中是否存在置位的二进制位 |
| b.all()                | b中所有位都置位了吗       |
| b.none()               | b中不存在置位的二进制位吗 |
| b.count()              | b中置位的位数             |
| b.size()               | 返回b中的位数             |
|                        |                           |
| b.test(pos)            | 查看pos位置是否置位       |
| b.set(pos, v)          | 置位                      |
| b.set()                | 同上                      |
| b.reset(pos)           | 复位                      |
| b.reset()              | 同上                      |
| b.flip(pos)            | 改变位状态                |
| b.flip()               | 同上                      |
| b[pos]                 | 获取位状态                |
|                        |                           |
| b.to_ulong()           |                           |
| b.toullong()           |                           |
|                        |                           |
| b.to_string(zero, one) |                           |
| os << b                |                           |
| is >> b                |                           |

输入运算符从输入流读取字符保存到临时的string对象中，之后用临时的string对象来初始化bitset。

### 17.3 正则表达式

RE库在头文件regex中

### 17.4 随机数

头文件random提供了随机数引擎类，用来生成随机unsigned整数序列；提供了随机数分布类，用来使用引擎返回服从服从特定概率分布的随机数。

> c++程序不应该使用库函数rand，而应使用default_random_engine类和恰当的分布类对象

#### 17.4.1 随机数引擎和分布

```cpp
    default_random_engine  e;// 生成随机无符号数
    for(size_t i = 0; i < 10;++i) {
        cout << e() << endl;
    }
```

**表17.15 随机数引擎操作**

|                     |                              |
| ------------------- | ---------------------------- |
| Engine e;           | 默认构造函数；使用默认的种子 |
| Engine e(s);        | 使用s作为种子                |
| e.seed(s);          | 使用种子s重置引擎状态        |
| e.min(); e.max()    | 此引擎可生成的最小值和最大值 |
| Engine::result_type | 此引擎生成的unsigned整型类型 |
| e.discard(u)        | 将引擎推进u步                |

**分布类型和引擎**

```cpp
    uniform_int_distribution<unsigned> u(0, 9);
    default_random_engine e; // 生成无符号随机整数
    for(size_t i = 0;i < 10;++i) {
        // 将u作随机数源
        // 每个调用返回在指定范围内并服从均匀分布的值
        cout << u(e) << endl;
    }
```

分布类型也是函数对象类。分布类型定义了一个调用运算符，使用它的引擎参数生成随机数，并将其映射到指定的分布。

随机数发生器指分布对象和引擎对象的组合。

种子相同时，生成的随机数序列也是相同的。

unitform_real_distribution类型的对象用来处理从随机整数到随机浮点数的映射。

**使用分布的默认结果类型**

```cpp
uniform_real_distribution<> u(0, 1); // 默认生成double值
```

**生成非均匀分布的随机数**

normal_distribution生成正态分布的值的序列

**bernoulli_distribution 类**

伯努利分布

### 17.5 IO库再探

#### 17.5.1 格式化输入和输出

标准库定义了一组操纵符来修改流的格式状态。endl操纵符将输出一个换行符并刷新缓冲区。

**表17.17 定义在iomanip中的操纵符**

|                         |                                          |
| ----------------------- | ---------------------------------------- |
| boolalpha / noboolalpha | 将true和false输出为字符串 / 恢复         |
| showbase / noshowbase   | 将整型值输出为表示进制的前缀 / 恢复      |
| showpoint / noshowpoint | 对付点至总是显示小数点 / 恢复            |
| showpos / noshowpos     | 对非负数显示+ / 恢复                     |
| uppercase / nouppercase | 十六进制值打印OX，科学计数法打印E / 恢复 |
| dec                     | 整型值显示为十进制                       |
| hex                     | 整型值显示为十六进制                     |
| oct                     | 整型值显示为八进制                       |
| left                    | 在值的右侧添加填充字符                   |
| right                   | 在值的左侧添加填充字符                   |
| internal                | 在符号和值之间添加填充字符               |
| fixed                   | 浮点值显示为定点十进制                   |
| scientific              | 浮点值显示为科学计数法                   |
| hexfloat                | 浮点值显示为十六进制                     |
| defaultfloat            | 充值浮点值格式为十进制                   |
| unitbuf / nounitbuf     | 每次输出操作后都刷新缓冲区 / 恢复        |
| skipws / noskipws       | 输入运算符跳过空白符 / 恢复              |
| flush                   | 刷新ostream 缓冲区                       |
| ends                    | 插入空字符，然后刷新ostream缓冲区        |
| endl                    | 插入换行，然后刷新ostream缓冲区          |
| setfill(ch)             | 用ch填充空白                             |
| setprecision(n)         | 将浮点精度设置为n                        |
| setw(w)                 | 读和写值的宽度为w个字符                  |
| setbase(b)              | 将整数输出为 b 进制                      |

setprecision 操纵符接受一个参数来设置精度

noskipws操纵符会令舒服运算符读取空白符，而不是跳过。skipws操纵符可恢复。

#### 17.5.2 未格式化的输入/输出操作

未格式化IO可将流当做一个无解释的字节序列来处理。

**表17.19 单字节底层IO操作**

|                |                                              |
| -------------- | -------------------------------------------- |
| is.get(ch)     | 从istream is读取下一个字节存入字符ch。返回is |
| os.put(ch)     | 将字符ch输出到ostream os。返回os             |
| is.get()       | 将is的下个字节作为int返回                    |
| is.putback(ch) | 将字符ch放回is。返回is                       |
| is.unget()     | 将is向后移动一个字节。返回is                 |
| is.peek()      | 将下个字节作为int返回，但不从流中删除它      |

**从输入操作返回的int值**

peek函数和无参get版本都以int类型从输入流返回一个字符。返回int的原因是：可以返回文件尾标记。char范围中没有额外的值表示文件尾。

返回int的函数将要返回的字符先转换成 unsigned char，然后将结果提升为int。因此即使字符集中有字符映射到负值，这些操作返回的int也是正值。而标准库使用负值表示文件尾。头文件cstdio 定义了EOF，可用来检测get的返回值是否为文件尾。

```cpp
int ch; // 使用int而不是char
while((ch = cin.get()) != EOF)
    cout.put(ch);
```



**表17.20 多字节底层IO操作**

|                               |                                                              |
| ----------------------------- | ------------------------------------------------------------ |
| is.get(sink, size, delim)     | 从is中读取最多size个字节，并保存在字符数组中，字符数组的起始地址为sink。读取直到遇到字符delim或读取了size个字节或遇到文件尾时停止。如果遇到delim，则其在输入流中，不存入sink |
| is.getline(sink, size, delim) | 同上。但会读取并丢弃delim                                    |
| is.read(simk, size)           | 读取最多size个字节，存入字符数组sink中。返回is               |
| is.gcount()                   | 返回上一个未格式化读取操作从is读取的字节数                   |
| os.write(source, size)        | 将字符数组source中的size个字节写入os。返回os                 |
| is.ignore(size, delim)        | 读取并忽略最多size个字符，包括delim                          |

> 一个常见的错误是将get或peek的返回值赋予char而不是int，而且编辑器不能发现这个错误。比如下面的循环永远不会停止
>
> char ch; // 此处使用char就是错误
>
> // cin.get 返回值被转换为char，然后与int比较
>
> while((ch = cin.get()) != EOF)
>
> ​	cout.put(ch);
>
> get返回EOF时，会被转换成unsigned char，此值与EOF的int值不再相等

#### 17.5.3 流随机访问

标准库提供了一堆函数，来定位（seek）到流中给定的位置，以及告诉（tell）当前位置。

> 随机IO本质上是依赖系统的

**表17.21 seek和tell函数**

g版本表示在读取数据，p版本表示在写入数据

|                                     |                                   |
| ----------------------------------- | --------------------------------- |
| tellg() / tellp()                   | 返回输入（出）流中标记的当前位置  |
| seekg(pos) / seekp(pos)             | 修改输入（出）流中的标记地址      |
| seekp(off, from) / seekg(off, from) | 定位标记到from之前或之后off个字符 |

from可能值

- beg，偏移量相对于流开始位置
- cur，偏移量相对于流当前位置
- end，偏移量相对于流结尾位置

**只有一个标记**

在一个流中只维护单一的标记，并不存在独立的读标记和写标记。

**访问标记**

函数tell返回pos_type值，表示流的当前位置

## 18 用于大型程序的工具

### 18.1 异常处理

#### 18.1.1 抛出异常

在c++中，通过抛出（throwing）一条表达式引发（raised）一个异常。被抛出的表达式的类型以及当前的调用链共同决定了哪段处理代码（handler）将被用来处理该异常。

**栈展开**

当throw出现在try语句块内时，检查与该try块关联的catch子句，若没找到则在外层继续寻找。此过程为栈展开。栈展开过程沿着嵌套函数的调用链不断查找，直到找到与异常匹配的catch子句为止。一直没找到则退出主函数后查找过程终止。

若找到匹配的catch子句，则进入子句并执行其中代码后，在该子句开始继续执行。当找不到匹配的catch时，程序将调用标准库函数terminate 终止程序执行。

在栈展开过程中，块退出后它的局部对象也将随之销毁。

如果块分配了资源，并且在负责释放这些资源的代码前面发生了异常，则释放资源的代码将不会被执行。析构函数在栈展开的过程中执行，因此析构函数不应该抛出不能被它自身处理的异常。即如果析构函数需要执行某个可能抛出异常的操作，该操作应该被放置在一个try语句块中并且在析构函数内部得到处理。

**异常对象**

编辑器使用异常抛出表达式来对异常对象进行拷贝初始化。因此throw语句中的表达式必须拥有完全类型。若该表达式是类类型的话，相应类应该含有一个可访问的析构函数和拷贝或移动构造函数。若该表达式是数组类型或函数类型，则表达式将被转换成与之对应的指针类型。

异常对象位于由编译器管理的空间中。当异常处理完毕后，异常对象被销毁。

当抛出表达式时，该表达式的静态编译时的类型决定了异常对象的类型。即若throw表达式解引用基类指针，而指针实际指向的是派生类对象，则只有基类部分被抛出

#### 18.1.2 捕获异常

catch子句中的异常声明类似只包含一个形参的函数形参列表。声明类型决定了处理代码所能捕获的异常类型，该类型必须是完全类型，不能是右值引用。

进入catch语句后，异常对象初始化异常声明中的参数。

异常声明的静态类型决定catch语句所能执行的操作。所以如果catch接受的异常与某个继承体系有关，则最好将该catch的参数定义成引用类型。

异常和catch异常声明的匹配规则要求时精确匹配的，绝大多数类型转换都不被允许。

重新抛出操作将异常传递给另外一个catch语句。这里为空throw语句

```cpp
throw;
```

空的throw语句只能出现在catch语句或该语句直接或间接调用的函数之内。重新抛出语句并不指定新的表达式，而是将当前的异常对象沿着调用链向上传递。

```cpp
catch(my_error& eObj) { // 引用类型
    eObj.status - errCodes::severeErr; // 修改了异常对象
    throw; // 异常对象的status是servereErr
}catch(other_error& eObj) { // 非引用
    eObj.status = errorCodes::badErr; // 只修改局部副本
    throw; // 异常对象的status成员没有改变
}
```

使用省略号作为异常声明，可以捕获所有异常。catch(...)通常与重新抛出语句一起使用。

```cpp
void mainp() {
    try {
        // 操作引发并抛出一个异常
    }
    catch(...) {
        // 处理异常的某些特殊操作
        throw;
    }
}
```

> 如果catch(...) 与其他catch语句一起出现，则catch(...)必须在最后的位置。出现在捕获所有异常语句后面的catch语句将永远不会被匹配。

#### 18.1.3 函数try语句块与构造函数

构造函数在进入函数体之前先执行初始值列表。因为在初始值列表抛出异常时构造函数体内的try语句块还未生效，所以构造函数体内的catch语句无法处理构造函数初始值列表抛出的异常。

为了处理该异常，可以将构造函数写成函数try语句块的形式。关键字try出现在表示构造函数初始值列表的冒号以及表示构造函数体的花括号之前。与这个try关联的catch既能处理构造函数体抛出的异常，也能处理成员初始化列表抛出的异常。

函数try语句块只能处理构造函数开始执行后发生的异常。如果在参数初始化的过程中发生异常，应该在调用者所在的上下文中处理。

```cpp
template<typename T>
Blob<T>::Blob(std::initializer_list<T> il) try :
data(std::make_shared<std::vector<T>>(il)) {
    
} catch(const std::bad_alloc& e) {
    handle_out_of_memory(e);
}
```

> 处理构造函数初始值异常的唯一方法时将构造函数写成函数try 语句。

#### 18.1.4 noexcept异常说明

noexcept说明指定某个函数不会抛出异常。noexcept如果出现，则在该函数的所有声明和定义语句都要出现。noexcept在typedef或类型别名中不能出现。成员函数中，该说明符需要在const及引用限定符之后，在final、override或虚函数的=0之前。

```cpp
void recoup(int) noexcept; // 不会抛出异常
void alloc(int);// 可能抛出异常
```

**违反异常说明**

编译器并不会在编译时检查noexcept说明。如果一个函数同时存在noexcept和throw或调用了抛出异常的其它函数，还是会抛出异常。一旦noexcept函数抛出异常，程序就会调用terminate以确保遵守不在运行时抛出异常的承诺，该过程对是否执行栈展开没约定，因此noexcept可用在两种情况下：

- 确认函数不会抛出异常
- 根本不知道该如何处理异常

> 向后兼容：这两条声明语句等价
>
> void recoup(int) noexcept;
>
> void recoup(int) throw();

**异常说明的实参**

```cpp
void recoup(int) noexcept(true); // recoup不会抛出异常
void alloc(int) noexcept(false); // alloc可能抛出异常
```

**noexcept运算符**

noexcept说明符的实参常与noexcept运算符混合使用。noexcept运算符是一元运算符，返回bool类型的右值常量表达式，用于表示给定的表达式是否会抛出异常。noexcept也不会求运算对象的值。

```cpp
void f() noexcept(noexcept(g())); // f和g的异常说明一致
```

> noexcept在参数列表后面时为异常说明符；有bool实参时为运算符

**异常说明与指针、虚函数和拷贝控制**

函数指针及该指针所指的函数必须具有一致的异常说明。如果虚函数承诺了它不会抛出异常，则后续派生的虚函数也必须做出相同的承诺。如果没有承诺，派生类的对应函数也可以承诺。

#### 18.1.5 异常类层次

- exception
- - bad_cast
  - bad_alloc
  - runtime_error
  - - overflow_error
    - underflow_error
    - range_error
  - logic_error
  - - domain_error
    - invalid_argument
    - out_of_range
    - length_error

exception仅定义了拷贝构造函数、拷贝赋值运算符、虚析构函数和what的虚成员。what函数返回const char*，该指针指向一个以unll结尾的字符数组，并确保不会抛出任何异常。

类exception、bad_cast、bad_alloc定义了默认构造函数。类runtime_error和logic_error没有默认构造函数，但有接受C风格字符串或std::string类型实参的构造函数。这些类中，what负责返回用于初始化异常对象的信息。

**书店应用程序的异常类**

实际的应用程序通常会自定义exception的派生类以扩展其继承体系。

```cpp
// 自定义异常类
class out_of_stock: public std::runtime_error {
public:
    explicit out_of_stock(const std::string& s):
        std::runtime_error(s) {};
};
class isbn_mismath: public std::logic_error {
public:
    explicit isbn_mismath(const std::string& s):
        std::logic_error(s) {};
    isbn_mismath(const std::string& s, 
                 const std::string& lhs, 
                 const std::string& rhs):
                 std::logic_error(s), left(lhs), right(rhs) {};
    const std::string left, right;
};
// 调用自定义异常
int main() {
    throw isbn_mismath("wrong isbns", "1", "2");
    return 0;
}
```

### 18.2 命名空间

多个库将名字放置在全局命名空间中将引发命名空间污染。

命名空间不能定义在函数或类的内部。

命名空间可以是不连续的。

命名空间之外定义的成员必须使用含有前缀的名字。确定名字（如函数）位于命名空间的作用域内，就可以直接使用该命名空间的其他成员。

```cpp
cplusplus_primer::Sales_data
cplusplus_primer::operator+(const Sales_data& lhs,
                            const Sales_data& rhs) {};
```

尽管命名空间的成员可以定义在命名空间外部，但是这样的定义必须出现在所属命名空间的外层空间中。

```cpp
::member_name // 全局命名空间中的一个成员
```

内联命名空间中的名字可以被外层命名空间直接使用。关键字inline必须出现在命名空间第一次定义的地方，后续再打开命名空间的时候可忽略。内联命名空间常用在代码版本更迭中，将新旧代码版本都放在同一命名空间下，再用额外的命名空间区分旧代码版本，新代码版本采用内联命名空间。

```cpp
inline namespace FifthEd {}
namespace FifthEd{} // 隐式内联
```

**未命名的命名空间**

```cpp
namespace {}
```

未命名的命名空间中定义的变量具有静态生命周期。该命名空间可以在某个给定的文件内不连续，但不能跨越多个文件，如果两个文件都有未命名的命名空间，则这两个空间互相无关。如果一个头文件定义了未命名的命名空间，则该命名空间中定义名字将在每个包含了该头文件的文件中对应不同实体。

> 未命名的命名空间仅在特定的文件内部有效，其作用范围不会横跨多个不同的文件

定义在未命名的命名空间中的名字可以直接使用。

> 未命名的命名空间取代文件中的静态声明

**using声明**

在类的作用域中，using声明只能指向基类成员

```cpp
// 命名空间别名
namespace Qlib = cplusplus_primer::QueryLib;
```

**using指示**

```cpp
using namespace std;
```

using指示将指定命名空间的作用域提升到当前作用域中。

**实参相关的查找与类类型形参**

对于命名空间中名字的隐藏规则来说有个重要例外：当给函数传递一个类类型的对象时，除了在常规的作用域查找外还会查找实参类所属的命名空间。这一例外对于传递类的引用或指针的调用同样有效。

```cpp
std::string s;
std::cin >> s;
// 等同于，operator>>函数定义在标准库string中
operator>>(std::cin, s);
```

当编译器发现operator>> 调用时，首先在当前作用于中查找合适函数，接着查找输出语句的外层作用域。因为>>表达式的形参时类类型，所以编译器还会查找cin和s的类所属的命名空间。

**友元声明与实参相关的查找**

当类声明友元时，该友元声明并没有使友元本身可见。而另一个未声明的类或函数如果第一次出现在友元声明中，则认为它是最近的外层命名空间的成员。

```cpp
namespace A {
    class C {
        // 这些函数隐式成为命名空间A的成员
        friend void f2(); // 除非另有声明，否则不会被找到
        friend void f(const C&); // 根据实参相关的查找规则可以被找到
    }
}

int main() {
    A::C cobj;
    f(cobj); // 正确，通过A::C的友元声明找到A::f
    f2(); // 错误，A::f2没有被声明
}
```

### 18.3 多重继承与虚继承

基类的构造顺序与派生列表中基类的出现顺序保持一致。

**继承的构造函数与多重继承**

```cpp
struct B1 {
    B1() = default;
    B1(const string&);
};
struct B2 {
    B2() = default;
    B2(const string&);
};
// 如果继承了相同的构造函数，则必须定义自己的构造函数版本
struct D1: public B1, public B2 {
    using B1::B1;
    using B2::B2;
    D1(const string& s): B1(s), B2(s); 
};
```

在合成的拷贝控制成员中，每个基类分别使用自己对应的成员隐式地完成构造、赋值和销毁等工作。

派生类到几种基类的转换中都一样好，会产生二义性问题。

对象、指针和引用的静态类型决定了能够使用的成员。

在多重继承的情况下，如果名字在多个基类中都被找到，则对该名字的使用将具有二义性。

**虚继承**

如果某个类在派生过程中出现了多次，则派生类中将包含该类的多个子对象。

虚继承的目的是令某个类做出声明，承诺愿意共享它的基类。其中共享的基类子对象成为虚基类。在这种机制下，派生类都只包含唯一一个共享的虚基类子对象。

虚派生只影响从指定了虚基类的派生类中进一步派生出的类，不会影响派生类本身。

```cpp
class ZooAnimal{};
class Raccoon : virtual public ZooAnimal {};
class Bear : virtual public ZooAnimal {};
// Panda只有一份ZooAnimal部分
class Panda : public Raccoon, public Bear {};
```

如果成员被多个基类覆盖，则一般情况下派生类必须为该成员自定义一个新的版本。

**构造函数与虚继承**

```cpp
class ZooAnimal{};
class Raccoon : virtual public ZooAnimal {};
class Bear : virtual public ZooAnimal {};
// Panda只有一份ZooAnimal部分
class Panda : public Raccoon, public Bear {};
```

含有虚基类的对象的构造顺序如下：

- 先使用Panda的构造函数初始值列表中提供的初始值构造虚基类ZooAnimal
- 然后依次构造Bear，Raccoon
- 最后构造Panda部分

如果Panda没有显示初始化ZooAnimal基类，则ZooAnimal的默认构造函数将被调用。

虚基类总是先于非虚基类构造，与它们在继承体系中的次序和位置无关。

