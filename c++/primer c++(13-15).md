# Primer c++第五版笔记4(到第15章完)

~~这是笔记，所以别问为什么写的跟书上一摸一样了~~

[TOC]

## 13 拷贝控制

定义一个类时，可以显式或隐式地指定此类型的对象拷贝、移动、赋值和销毁时做什么。`拷贝构造函数、拷贝复制运算符、移动构造函数、移动赋值运算符和析构函数`。

如果一个类没有定义所有这些拷贝控制成员，编译器会自动定义缺失的操作。

### 13.1 拷贝、赋值与销毁

#### 13.1.1 拷贝构造函数

如果一个构造函数的第一个参数时自身类型的引用，且任何额外参数都有默认值，则此狗奥函数时拷贝构造函数

```cpp
class Foo {
public:
    Foo();//默认构造函数
    Foo(const Foo&);//拷贝构造函数
}
```

拷贝构造函数的第一个参数必须是一个引用类型。拷贝构造函数在几种情况下都会被隐式地使用。因此，拷贝构造函数通常不应该是explicit的。

**合成拷贝构造函数**

与合成默认构造函数不同，即使定义了其他构造函数，编译器也会为我们合成一个拷贝构造函数。即使定义了其他构造函数，编译器也会自动合成一个拷贝构造函数。

对于某些类来说，合成拷贝构造函数用来阻止我们拷贝该类类型的对象。

**拷贝初始化**

```cpp
string dots(10, ',');//直接初始化
string s(dots);//直接初始化
string s2 = dots;//拷贝初始化
string null_book = "irhwfu";//拷贝初始化
string nines = string(100, '9');//拷贝初始化
```

使用拷贝初始化时，要求编译器将右侧运算对象拷贝到正在创建的对象中，如果需要的话还要进行类型转换。拷贝初始化通常使用拷贝构造函数来完成。如果一个类中由移动构造函数，则拷贝初始化有时会使用移动构造函数而非拷贝构造函数来完成。

拷贝初始化发生情况

- 使用=定义变量

- 将一个对象作为实参传递给一个非引用类型的实参
- 从一个返回类型为非引用类型的函数返回一个对象
- 用花括号列表初始化一个数组中的元素或一个聚合类中的成员

初始化标准库容器或调用其insert或push成员时，容器会对其元素进行拷贝初始化。而emplace成员创建的元素都进行直接初始化。

**参数和返回值**

拷贝构造函数被用来初始化非引用类类型参数，这一特性解释了为什么拷贝构造函数自己的参数必须是引用类型。如果其参数不是引用类型，则调用永远也不会成功。为了调用拷贝构造函数，必须拷贝它的实参，但是为了拷贝实参，又需要调用拷贝构造函数，无限循环。

**拷贝初始化的限制**

如果使用初始化值要求通过一个explicit的构造函数来进行类型转换，那么使用拷贝初始化还是直接初始化显得尤为重要了

```cpp
vector<int> v1(10);//true，直接初始化
vector<int> v2 = 10;//error，接受大小参数的构造函数时explicit的
```

**编译器可以绕过拷贝构造函数**

在拷贝初始化过程中，编译器可以（不是必须）跳过拷贝、移动构造函数，直接创建对象。但是在这个程序点上，拷贝/移动构造函数必须是存在且可访问的（例：不能是private）。

```cpp
string null_book = "0000";//拷贝初始化
string null_book("0000");//略过拷贝构造函数
```

#### 13.1.2 拷贝赋值运算符

重载运算符本质上是函数，其名字由operator关键字后接表示要定义的运算符的符号组成。某些运算符，必须定义为成员函数。如果一个运算符是一个成员函数，其左侧运算对象就绑定到隐式的this参数。

```cpp
class Fpp {
public:
    Foo& operator=(const Foo&);
}
```

赋值运算符通常应该返回一个指向其左侧运算对象的引用。

**合成拷贝赋值运算符**

对于某些类，合成拷贝赋值运算符用来禁止该类型对象的赋值。

```cpp
Sales_data&
Sales_data::operator=(const Sales_data& rhs) {
    bookNo = rhs.bookNo;
    units_sold = rhs.units_sold;
    revenue = rhs.revenue;
    return *this;
}
```

#### 13.1.3 析构函数

析构函数是类的一个成员函数，没有返回值，也不接收参数，所以不能被重载。隐式销毁一个内置指针类型的成员不会delete它所指向的对象。

**调用析构函数的情况**

- 离开作用域的变量
- 对于动态分配的对象，当对指向它的指针应用delete运算符时被销毁。
- 容器被销毁时，其元素被销毁
- 对于临时对象，当创建它的完整表达式结束时被销毁

*当指向一个对象的引用或指针离开作用域时，析构函数不会执行*

**合成析构函数**

当一个类未定义自己的析构函数时，编译器会为它定义一个`合成析构函数`。对于某些类，合成析构函数用来阻止该类型的对象被销毁。

```cpp
class Sales_data {
public:
    //成员会被自动销毁，除此之外不需要做其他事情
    ~Sales_data() {}
};
```

在（空）析构函数体执行完毕后，成员会被自动销毁。析构函数自身并不直接销毁成员。成员是在析构函数体之后隐含的析构阶段中被销毁的。在整个对象销毁过程中，析构函数体是作为成员销毁步骤之外的另一个部分进行的。

#### 13.1.4 三/五法则

有三个基本操作可以控制类的拷贝操作：拷贝构造函数、拷贝赋值运算符、析构函数。新标准下，类还可以定义移动构造函数和移动赋值运算符。

**需要析构函数的类也需要拷贝和赋值操作**

如果类需要析构函数，几乎可以肯定它也需要一个拷贝构造函数和一个拷贝赋值运算符。

```cpp
//类在构造函数中分配动态内，需要定义析构函数来释放构造函数分配的内存
class HasPtr {
public:
    HasPtr(const std::string& s = std::string()):
    	ps(new std::string(s)), i(0) {}
    ~HasPtr() { delete ps; }
    //error，HasPtr需要拷贝构造函数和拷贝赋值运算符
}
HasPtr f(HasPtr hp) {
    HadPtr ret = hp;
    return ret;
}
//当f返回时，hp和ret都被销毁，两个对象都会调用HadPtr的析构函数。
//析构函数会delete ret和hp中的指针成员
//当这两个对象包含相同的指针值，会导致此指针被delete两次，并且原指针变得无效
```

**如果类需要拷贝构造函数，几乎可以肯定它也需要拷贝赋值运算符。反之亦然**

#### 13.1.5 使用=default

使用=default来显式要求编译器生成合成的版本

```cpp
class Sales_data {
public:
    Sales_data() = default;
    Sales_data(const Sales_data&) = default;
    Sales_data &operator=(const Sales_data &);
    ~Sales_data() = default;
}
```

*只能对具有合成版本的成员函数使用=default*

#### 13.1.6 阻止拷贝

**定义删除的函数**

```cpp
struct NoCopy {
    NoCopy() = default;
    NoCopy(const NoCopy&) = delete; // 阻止拷贝
    NoCopy &operator=(const NoCopy&) = delete;//阻止赋值
    ~NoCopy() = default;
};
```

和=default不同的一点是，=delete必须出现在函数第一次声明的时候，另一点是可以对任何函数指定=delete来引导函数匹配过程。

**析构函数不能是删除的成员**

对于删除了析构函数的类型，不能定义这种类型的变量或成员，但是可以动态分配这种类型的对象，但是不能释放这些对象.

```cpp
struct NoDtor {
    NoDtor() = default;
    ~NoDtor() = delete;
}
NoDtor nd;//error，NoDtor的析构函数是删除的
NoDtor *p = new NoDtor();//true，但是不能delete p
```

**合成的拷贝控制成员可能是删除的**

如果一个类有数据成员不能默认构造、拷贝、复制或销毁，则对应的成员函数将被定义为删除的。

一个成员有删除的或不可访问的析构函数会导致合成的默认和拷贝构造函数被定义为删除的。

对于具有引用成员或无法默认构造的const成员的类，编译器不会为其合成默认构造函数。

*本质上，当不可能拷贝、赋值或销毁类的成员时，类的合成拷贝控制成员就被定义为删除的*

**private拷贝控制**

在新标准发布之前，类是通过将其拷贝构造函数和拷贝赋值运算符声明为private来阻止拷贝。但是友元和成员函数仍旧可以拷贝对象。为了阻止友元和成员函数进行拷贝，这些拷贝控制成员声明为private，但不定义。

```cpp
class PrivateCopy {
    //默认private
    PrivateCopy(const PrivateCopy&);
    PrivateCopy &operator=(const PrivateCopy&);
public:
    PrivateCopy() = default;
    ~PrivateCopy();
}
```

### 13.2 拷贝控制和资源管理

通常，管理类外资源的类必须定义拷贝控制成员。为了定义这些成员，必须确定此类型对象的拷贝语义。可以定义拷贝操作，使类的行为看起来像一个值或者像一个指针。string类的行为像一个值，sahred_ptr类像指针。

#### 13.2.1 行为像值的类

```cpp
class HasPtr {
public:
    HasPtr(const string &s = string()):
        ps(new string(s)), i(0) {};
    HasPtr(const HasPtr &p):
        ps(new string(*p.ps)), i(p.i) {};
    HasPtr& operator=(const HasPtr &rhs) {
        auto newp = new string(*rhs.ps);//拷贝底层string
        delete ps;
        ps = newp;
        i = rhs.i;
        return *this;
    };
    ~HasPtr() { delete ps;}
private:
    string *ps;
    int i;
};
```

> 赋值运算符
>
> - 如果将一个对象赋予它自身，赋值运算符必须能正确工作
> - 大多数赋值运算符符合了析构函数和拷贝构造函数的工作
>
> 一个好的模式使先将右侧运算对象拷贝到一个局部临时对象后再进行处理

#### 13.2.2 定义行为像指针的类

**使用引用计数直接管理资源**

```cpp
class HasPtr {
public:
    HasPtr(const string &s = string()):
        ps(new string(s)), i(0), use(new size_t(1)) {};
    HasPtr(const HasPtr &p):
        ps(p.ps), i(p.i), use(p.use) {++*use};
    HasPtr& operator=(const HasPtr &rhs) {
        ++*rhs.use;
        if(--*use == 0) {
            delete ps;
            delete use;
        }
        ps = rhs.ps;
        i = rhs.i;
        use = rhs.use;
        return *this;
    };
    ~HasPtr() {
        if(--*use == 0) {
            delete ps;
            delete use;
        }
    }
private:
    string *ps;
    int i;
    size_t *use//引用计数
};
```

### 13.3 交换操作

```cpp
//自定义swap
class HasPtr {
    firend void swap(HasPtr&, HasPtr&);
}
inline
void swap(HasPtr &lhs, HasPtr &rhs) {
    using std::swap;
    swap(lhs.ps, rhs.ps);
    swap(lhs.i, rhs.i);
}
```

**swap函数应该调用swap，而不是std::swap**

在本例中，数据成员是内置类型，没有特定版本的swap，所以对std::swap的调用会使用std::swap

正确的swap函数

```cpp
void swap(Foo &lhs, Foo &rhs) {
    using std::swap;
    swap(lhs.h, rhs.h);
}
```

> 如果存在类型特定的swap版本，swap调用会与之匹配。如果不存在类型特定的版本，则会使用std中的版本（假定作用域中有using声明）。swap函数中的using声明没有隐藏特定版本的swap，这点在18.2.3节解释

**在赋值运算符中使用swap**

定义swap的类通常用swap来定义它们的赋值运算符。这些运算符使用了`拷贝并交换`的技术。

```cpp
    HasPtr& operator=(HasPtr rhs) {
        swap(*this, rhs); //rhs现在指向本对象曾经使用的内存
        return *this; //rhs被销毁，delete了rhs中的指针
    }
```

这个技术自动处理了自赋值情况且天然就是安全的。在改变左侧运算符对象之前拷贝右侧运算对象保证了自赋值的正确。唯一可能抛出异常的是拷贝构造函数中的new表达式。

### 13.4 拷贝控制示例

簿记操作示例。两个类命名为Message和Folder，分别表示消息和消息目录。。每个Message对象可以出现在多个Folder中。但是Message的内容只有一个副本。即如果Message的内容被改编，则从任何Folder来浏览此Message时，都会看到改变后的内容。

为了记录Message所在的Folder，每个Message都保存它所在Folder的指针的set，每个Folder都保存它包含的Message的指针的set

```cpp
//提供save和remove操作，来向给定Foler添加/删除一条Message
//拷贝Message时，副本和原对象为不同Message对象，但是出现在相同Folder中
//销毁Message要更新所在的Folder
class Message {
    friend class Folder;
public:
    //folders被隐式初始化为空集合
    explicit Message(const string &str = ""):
        contents(str) {};
    //拷贝控制成员，用来管理指向本Message的指针
    Message(const Message&);
    Message& operator=(const Message&);
    ~Message();
    //从给定Folder集合中添加/删除本Message
    void save(Folder&);
    void remove(Folder&);
    void swap(Message &lhs, Message &rhs);
private:
    string contents;//实际消息文本
    set<Folder*> folders;//包含本Message的Folder
    //拷贝构造函数、拷贝赋值运算符和析构函数所使用的工具函数
    //将本Message添加到指向参数的Folder中
    void add_to_Folders(const Message&);
    //从folders中的每个Folder中删除本Message
    void remove_from_Folders();
};
Message::Message(const Message &m) :
    contents(m.contents), folders(m.folders){
    add_to_Folders(m);
}
Message& Message::operator=(const Message &rhs) {
    remove_from_Folders();
    contents = rhs.contents;
    folders = rhs.folders;
    add_to_Folders(rhs);
    return *this;
}
Message::~Message() {
    remove_from_Folders();
}
void Message::save(Folder &f) {
    folders.insert(&f);
    f.addMsg(this);
}
void Message::remove(Folder &f) {
    folders.erase(&f);
    f.remMsg(this);
}

void Message::swap(Message &lhs, Message &rhs) {
    using std::swap;//不重要，但是是好习惯
    //删除原指针
    for(auto f: lhs.folders)
        f->remMsg(&lhs);
    for(auto f: rhs.folders)
        f->remMsg(&rhs);
    swap(lhs.folders, rhs.folders);
    swap(lhs.contents, rhs.contents);
    for(auto f: lhs.folders)
        f->addMsg(&lhs);
    for(auto f: rhs.folders)
        f->addMsg(&rhs);
}

void Message::add_to_Folders(const Message &m) {
    for(auto f : m.folders)
        f->addMsg(this);
}
void Message::remove_from_Folders() {
    for(auto f : folders)
        f->remMsg(this);
}
```

### 13.5 动态内存管理类

例：实现只适用string的标准库vector类的简化版本，StrVec类

StrVec类将使用allocator来获得原始内存，添加新元素用allocator的construct成员在原始内存中创建对象，需要删除一个元素时使用destroy成员来销毁元素。

每个StrVec有三个指针成员指向其元素所使用的内存

- elements，指向分配的内存中的首元素
- first_free，指向最后一个实际元素之后的位置
- cap，指向分配的内存末尾之后的位置

**移动构造函数和std::move**

标准库保证“移后源”string仍然保持一个有效的、可析构的状态。可以假定每个string都有指向char数组的指针，string的移动构造函数进行了指针的拷贝。

第二个机制是一个名为move的标准库函数，定义在头文件utility中。关于move目前有两个关键点。一、当reallocate在新内存中构造string时，必须调用move彪表示希望使用string 的移动构造函数，否则将会使用拷贝构造函数。其次通常不为move提供using声明，直接调用std::move

```cpp
class StrVec {
public:
    StrVec():
        elements(nullptr), first_free(nullptr), cap(nullptr) {};
    StrVec(const StrVec&);
    StrVec& operator=(const StrVec&);
    ~StrVec();
    void push_back(const string&);//拷贝元素
    size_t size() const { return first_free - elements; };
    size_t capacity() const { return cap - elements; };
    string* begin() const { return elements; };
    string* end() const { return first_free; };
private:
    allocator<string> alloc;//分配元素
    //工具函数，被拷贝构造函数、赋值运算符和析构函数使用
    std::pair<string*, string*> alloc_n_copy(const string*, const string*);
    void free();//销毁元素并释放内存
    void reallocate();//获取更多内存并拷贝已有元素
    string *elements;//指向数组首元素
    string *first_free;//指向数组第一个空闲元素
    string *cap;//指向数组尾后位置
    //保证StrVec至少有容纳一个新元素的空间
    //添加元素操作必须调用
    void chk_n_alloc() {
        if(size() == capacity()) reallocate();
    };
};
StrVec::StrVec(const StrVec &s) {
    auto newdata = alloc_n_copy(s.begin(), s.end());
    elements = newdata.first;
    first_free = cap = newdata.second;
}
StrVec& StrVec::operator=(const StrVec &rhs) {
    auto data = alloc_n_copy(rhs.begin(), rhs.end());
    free();
    elements = data.first;
    first_free = cap = data.second;
    return *this;
}
StrVec::~StrVec() {
    free();
}

void StrVec::push_back(const string &s) {
    chk_n_alloc();
    StrVec::alloc.construct(first_free++, s);//调用string的拷贝构造函数
}

pair<string*, string*>
StrVec::alloc_n_copy(const string *b, const string *e) {
    auto data = StrVec::alloc.allocate(e - b);
    return {data, uninitialized_copy(b, e, data)};
};
void StrVec::free() {
    if(elements) {//elements != nullptr
        for(auto p = first_free;p != elements;)
            StrVec::alloc.destroy(--p);
        StrVec::alloc.deallocate(elements, cap - elements);
    }
}
void StrVec::reallocate() {
    auto newcapacity = size() ? 2 * size() : 1;
    auto newdata = StrVec::alloc.allocate(newcapacity);
    auto dest = newdata;//指向新数组中下一个空闲位置
    auto elem = elements; //指向旧数组中下一个元素
    for(size_t i = 0;i != size();++i)
        StrVec::alloc.construct(dest++, std::move(*elem++));
    free();
    elements = newdata;
    first_free = dest;
    cap = elements + newcapacity;
}
```

### 13.6 对象移动

新标准的一个最主要的特性是可以移动而非拷贝对象的能力。对于立即销毁的对象，移动对象会大幅度提升性能。

使用移动的另一个原因是类似IO类或者unique_ptr类。这些类都包含不能被共享的资源。因此这些类型的对象不能拷贝但可以移动。

#### 13.6.1 右值引用

`右值引用`，就是必须绑定到右值的引用。通过&&来获取右值引用。右值引用的一个重要性质——只能绑定到一个将要销毁的对象。

一般来说，左值表达式表示的是一个对象的身份，右值表达式表示的是对象的值。左值表达式不能绑定到要求转换的表达式、字面常量或是返回右值的表达式。而右值引用可以绑定到这类表达式上，但是将一个右值引用直接绑定到一个左值上。

*const的左值引用可以绑定到右值*

```cpp
int i = 43;
const int &r3 = i * 42;//正确，绑定到右值上
int &&rr2 = i* 42;//正确
```

**变量是左值**

变量可以看作只有一个运算对象而没有运算符的表达式，所以`不能将右值引用绑定到一个右值引用类型的变量上`。

```cpp
int &&rr1 = 42;
int &&rr2 = rr1;//错误，rr1是左值
```

**标准库move函数**

新标准库函数move，可以获取绑定到左值上的右值引用。定义在头文件utility中，机制在16.2.6节中介绍

```cpp
int &&rr3 = std::move(rr1);//ok
```

调用move就意味着承诺：除了对rr1赋值或销毁它外，将不再使用它。在调用move后，不能对移后源对象的值做任何假设。

*可以销毁一个移后源对象，也可以赋予它新值，但不能使用移后源对象的值*

直接调用std::move而不是move，原因在18.2.3节中解释。

#### 13.6.2 移动构造函数和移动赋值运算符

移动构造函数的第一个参数是该类类型的右值引用。除了完成资源移动，移动构造函数还必须确保移后源对象处于销毁是无害的状态。特别是，一旦资源完成移动，源对象必须不再指向被移动的资源。

```cpp
StrVec::StrVec(StrVec &&s) noexcept : //移动操作不应抛出任何异常
    elements(s.elements), first_free(s.first_free), cap(s.cap) {
    //对s运行析构函数是安全的
    s.elements = s.first_free = s.cap = nullptr;
}
```

noexcept通知标准库，此构造函数不抛出任何异常。移动构造函数不分配任何新内存，它接管给定的StrVec中的内存。

**移动操作、标准库容器和异常**

因为移动操作通常不分配任何资源，因此移动操作通常不会抛出任何异常。此事应该通知标准库，否则它会认为移动操作可能会抛出异常并且为了处理这种可能而做一些额外的工作。

一种通知的方法是在构造函数中指明noexcept，它是承诺函数不抛出异常的一种方法。

```cpp
class StrVec {
public:
    StrVec(StrVec&&) noexcept;//移动构造函数
}
StrVec::StrVec(StrVec &&s) noexcept : /*成员初始化器 */ {}
```

声明和定义中都需要指定noexcept，不抛出异常的移动构造函数和移动赋值运算符必须标记noexcept。

- 虽然移动操作通常不抛出异常，但抛出异常也是允许的
- 标准库能对异常发生时自身的行为提供保障

**移动赋值运算符**

```cpp
StrVec& StrVec::operator=(StrVec &&rhs) noexcept {
    //检测自赋值
    if(this != &rhs) {
        free();
        elements = rhs.elements;
        first_free = rhs.first_free;
        cap = rhs.cap;
        rhs.elements = rhs.first_free = rhs.cap = nullptr;
    }
    return *this;
}
```

**移后源对象必须可析构**

编写一个移动操作时，必须确保移后源对象进入一个可析构的状态，还必须保证对象仍然时有效的，即能安全地赋值或者安全地使用而不依赖其当前值。

**合成的移动操作**

同拷贝构造函数和拷贝赋值运算符，编译器也会合成移动构造函数和移动赋值运算符。但是合成移动操作的条件与合成拷贝操作的条件不大相同。

> 只有当一个类没有定义任何自己版本的拷贝控制成员 ，且类的每个非static数据成员都可以移动时，编译器才会为它合成移动构造函数和移动赋值运算符。

```cpp
struct X {//合成移动操作
    int i;
    string s;
}
```

存在不能合成移动构造函数的情况，例如存在无法合成的成员变量，析构函数被定义为删除或不可访问，类成员为const或引用。

> 定义了移动构造函数或移动赋值运算符的类必须也定义自己的拷贝操作。否则这些成员默认地被定义为删除的

**移动右值，拷贝左值，但如果没有移动构造函数，右值也被拷贝**

**拷贝并交换赋值运算符和移动操作**

```cpp
class HasPtr {
public:
    //移动构造函数
    HasPtr(HasPtr &&p) noexcept : ps(p.ps), i(p.i) {p.ps = 0;}
    //赋值运算符既是移动赋值运算符，也是拷贝赋值运算符
    HasPtr& operator=(HasPtr rhs) {
        swap(*this, rhs);
        return *this;
    }
};
```

对于赋值运算符，有一个非引用参数，参数要进行拷贝初始化，依赖实参的类型，左值被拷贝，右值被移动，因此该运算符既是移动赋值运算符，也是拷贝赋值运算符

> 关于三/五法则
>
> 所有五个拷贝控制成员应该看作一个整体：一般来说，如果一个类定义了任何一个拷贝操作，它就应该定义所有五个操作。

**Message类的移动操作**

```cpp
void Message::move_Folders(Message *m) {
    folders = std::move(m->folders);
    for(auto f: folders) {
        f->remMsg(m);//删除旧message
        f->addMsg(this);//添加新message
    }
    m->folders.clear();//确保销毁m是无害的
}
//要注意的是，向set插入一个元素可能会抛出bad_alloc异常(12.1.2节)
//因此移动构造函数和移动赋值运算符都不标记为noexcept
Message::Message(const Message &&m): contents(std::move(m.contents)) {
    move_Folders(&m);
}
Message& Message::operator=(Message &&rhs) {
    if(this != rhs) {
        remove_from_Folders();
        contents = std::move(rhs.contents);
        move_Folders(&rhs);
    }
    return *this;
}
```

**移动迭代器**

新标准库定义了`移动迭代器`适配器，解引用生成一个右值引用。通过调用标准库的make_move_iterator函数将普通迭代器转换为移动迭代器

```cpp
void StrVec::reallocate() {
    auto newcapacity = size() ? 2 * size() : 1;
    auto first alloc.allocate(newcapacity);
    auto last = uninitialized_copy(make_move_iterator(begin()),
                                   make_move_iterator(end()),
                                   first);
    free();
    elements = first;
    first_free = last;
    cap = elements + newcapacity
}
```

> 不要随便使用移动操作
>
> 由于移后源对象的状态不确定，所以调用move时，必须确认移后源对象没有其他用户

#### 13.6.3 右值引用和成员函数

```cpp
void push_back(const X&);//拷贝
void push_back(X&&); //移动
//区分移动和拷贝的重载函数通常有一个版本接受一个const T&，另一个版本接受T&&
```

**右值和左值引用成员函数**

```cpp
//右值的使用方式可能令人惊讶
    string s1 = "a value", s2 = "another";
    cout << (s1 + s2).find('a') << endl;//右值上调用find成员
    cout << (s1 + s2 = "wow!") << endl;//输出右值
    cout << s1 << " " << s2 << " " << s1 + s2 << endl;
```

有时候希望在自己的类中阻止这种用法，希望强制左侧运算符对象(即this指向的对象)是一个左值。指出this的左值/右值属性的方式和定义const成员函数相同，在参数列表后放置一个引用限定符

```cpp
class Foo {
public:
    Foo &operator=(const Foo&) &;//只能向可修改的左值赋值
};
Foo &Foo::operator=(const Foo &) &{
    //其他代码
    return *this;
}
```

引用限定符可以是&或&&，分别指出this可以指向一个左值或右值，之呢个能用于（非static）成员函数，且必须同时出现在函数的声明和定义中。

```cpp
Foo retVal();//retVal的调用是一个右值

Foo somMem() const &;//const必须在引用限定符之前
```

**重载和引用函数**

引用限定符也可以区分重载版本

```cpp
class Foo {
public:
    Foo sorted() &&;//可用于可改变的右值
    Foo sorted() const &; //可用于任何类型的Foo
private:
    vector<int> data;
};
Foo Foo::sorted() && {
    sort(data.begin(), data.end());
    return *this;
}
//本对象不能对其进行原址排序
Foo Foo::sorted() const & {
    Foo ret(*this);
    sort(ret.data.begin(), ret.data.end());
    return ret;
}
```

*如果一个成员函数有引用限定符，则具有相同参数列表的所有版本都必须有引用限定符*

## 14 重载运算和类型转换

### 14.1 基本概念

除了重载的函数调用运算符operator()之外，其他重载运算符不能含有默认实参。

> 当一个重载的运算符是成员函数时，this绑定到左侧运算符对象。成员运算符函数的(显式)参数数量比运算对象的数量少一个

可以重载大多数运算符，无权发明新的运算符号。+，-，*，&既是一元运算符也是二元运算符。

> 可以被重载的运算符
>
> | +    | -    | *    | /     | %      | ^        |
> | ---- | ---- | ---- | ----- | ------ | -------- |
> | &    | \|   | ~    | !     | ,      | =        |
> | <    | >    | <=   | >=    | ++     | --       |
> | <<   | >>   | ==   | !=    | &&     | \|\|     |
> | +=   | -=   | /=   | %=    | ^=     | &=       |
> | \|=  | *=   | <<=  | >>=   | []     | ()       |
> | ->   | ->*  | new  | new[] | delete | delete[] |

**直接调用一个重载的运算符函数**

```cpp
data1 + data2;//普通表达式
operator+(data1, data2);//等价的函数调用
data1.operator+=(data2);//同上
```

**某些运算符不应该被重载**

因为使用重载的运算符本质上是一次函数调用，所譬如逻辑运算符和逗号运算符的求值顺序规则无法保留下来，所以不建议重载。

**选择成为成员或者非成员**

如果希望含有混合类型的表达式中使用对称性运算符，那么必须定义成非成员函数。因为成员函数的左侧运算对象必须是运算符所属类的一个对象。

```cpp
string s;
s + "!";//如果为成员函数，为s.operator+("!");但不能使用"!"+s
operator+("hi", s);//非成员则可以通过类型转换实现
```

### 14.2 输入输出运算符

#### 14.2.1 重载输出运算符<<

ostream对象要用引用类型，参数一般为常量的引用，避免复制实参

```cpp
ostream &operator<<(ostream& os, const Sales_data& item) {
    os << item.isbn();
    return os;
}
```

> 通常输出运算符应该主要负责打印对象的内容而非控制格式，输出运算符不应该打印换行符

**输入输出运算符必须是非成员对象**

与iostream标准库兼容的输入输出运算符必须是普通的非成员函数，若为类的成员函数，它的左侧运算对象将是类的一个对象

```cpp
Sales_data data;
data << cout; //若operator<<为类成员函数
```

IO运算符通常需要读写类的非公有数据成员，所以IO运算符一般被声明为友元

#### 14.2.2 重载输入运算符>>

```cpp
istream &operator>>(istream& is, Sales_data& item) {
    double price;
    is >> item.boosNo >> item.units_sold >> price;
    if(is) //读入成功
        item.revenue = item.units_sold * price;
    else 
        item = Sales_data(); 
    return is;
}
```

> 输入运算符必须处理输入可能失败的情况，而输出运算符不需要

**输入时的错误**

可能发生的错误

- 当流含有错误类型的数据时读取操作可能失败
- 当读取操作达到文件末尾或遇到输入流的其他错误时也会失败

*当读取操作发生错误时，输入运算符应该负责从错误中恢复*

### 14.3 算术和关系运算符

通常情况下，算术和关系运算符定义成非成员函数以允许对左侧或右侧的运算对象进行转换。如果类定义了算术运算符，一般也会定义一个对应的复合赋值运算符。

```cpp
//要考虑两个引用为同一个对象的情况
Sales_data
operator+(const Sales_data& lhs, const Sales_data& rhs) {
    Sales_data sum = lhs;
    sum += rhs;
    return sum;
}
```

> 如果类同时定义了算术运算符和相关的复合赋值运算符，通常情况下应该使用复合赋值来实现算数运算符

#### 14.3.1 相等运算符

```cpp
bool operator==(const Sales_data& lhs, const Sales_data& rhs) {
    return lhs.isbn() == rhs.isbn() &&
            lhs.revenue == rhs.revenue &&
            lhs.units_sold == rhs.revenue;
}
bool operator!=(const Sales_data& lhs, const Sales_data& rhs) {
    return !(lhs == rhs);
}
```

设计准则

- 判断操作应该为非成员函数
- operator==运算符应该能判断一组给定对象中是否含有重复元素
- 相等运算符应该具有传递性
- 如果定义了operator==，也应该定义operator!=
- 相等运算符和不相等运算符可以委托工作

#### 14.3.2 关系运算符

因为关联容器和一些算法要用到小于运算符，所以定义operator<会比较有用

### 14.4 赋值运算符

除了拷贝赋值和移动赋值运算符，类还可以定义其他赋值运算符以使用别的类型作为右侧运算对象

```cpp
    Sales_data& operator=(std::initializer_list<std::string> il) {
        auto data = alloc_n_copy(il.begin(), il.end()); //自定义成员函数
        free();
        elements = data.first;
        first_free = cap = data.second;
        return *this;
    }
```

> 不论形参类型是什么，赋值运算符都必须定义为成员函数，返回左侧运算对象的引用

### 14.5 下标运算符

> 下标运算符必须是成员函数

下标运算符通常返回引用，这样下标可以出现在赋值运算符的任意一端。而且最好同时定义下标运算符的常量版本和非常量版本，当作用于一个常量对象时，返回常量引用以确保不会给返回的对象赋值。

```cpp
    std::string& operator[](std::size_t n) {
        return elements[n];
    }
    const std::string& operator[](std::size_t n) const {
        return elements[n];
    }
```

### 14.6 递增和递减运算符

> 虽然c++语言不要求递增和递减运算符必须为类的成员，但是它们改变的时操作对象的状态，所以建议为成员函数
>
> 定义递增和递减运算符的类应该同时定义前置版本和后置版本

**定义前置递增/递减运算符**

```cpp
    StrBlobPtr& operator++() {
        check(curr, "increment past end of StrBlobPtr");//检查是否越界
        ++curr;
        return *this;
    }
    StrBlobPtr& operator--() {
        --curr;
        check(curr, "decrement past begin of StrBlobPtr");
        return *this;
    }
```

**区分前置和后置运算符**

后置版本接受一个额外的（不被使用）int类型的形参。使用后置运算符时，编译器为这个形参提供一个值为0的实参。从语法上来说后置函数可以使用这个额外的形参，但实际上不会这么做。该形参的唯一作用就是区分前置版本和后置版本的函数。

```cpp
    StrBlobPtr& operator++(int) {
        StrBlobPtr ret = *this;
        ++*this;
        return ret;
    }
    StrBlobPtr& operator--(int) {
        StrBlobPtr ret = *this;
        --*this;
        return ret;
    }
```

**显式调用后置运算符**

```cpp
StrBlobPtr p(al);
p.operator++(0); //后置版本
p.operator++(); //前置版本
```

### 14.7 成员访问运算符

```cpp
    string& operator*() const {
        auto p = check(curr, "dereference past end");
        return (*p)[curr]; // *p 为对象所指的vector
    }
    string* operator->() const {
        return & this->operator*();
    }
```

**对箭头运算符返回值的限定**

operator*可以完成任何指定的操作，比如返回一个固定值。但是箭头运算符永远不能丢掉成员访问这个最基本的含义。

对于形如point->mem的表达式来说，point必须是指向类对象的指针或者是一个重载了operator->的类的对象。根据point类型的不同，point->mem分别等价于

```cpp
(*point).mem; //point是一个内置的指针类型
point.operator()->mem; // point是类的一个对象
```

point->mem的执行过程如下

1. 如果point是指针，则应用内置的箭头运算符，表达式等价于(*point).mem
2. 如果point是定义了operator->的类的一个对象，则使用point.operator->()的结果来获取mem。其中，如果该结果是一个指针，则执行第一步；如果该结果本身含有重载的operator->()，则重复调用当前步骤。最终，当这一过程结束时程序或者返回了所需的内容，或者返回一些表示程序错误的信息。

> 重载的箭头运算符必须返回类的指针或者定义了箭头运算符的某个类的对象

### 14.8 函数调用运算符

```cpp
struct absInt {
    int operator() (int val) const {
        return val < 0 ? -val : val;
    }
};
absInt absObj;
int p = absObj(43);
```

> 函数调用运算符必须是成员函数，而且可以重载

如果类定义了调用运算符，则该类的对象称作`函数对象`。函数对象常常作为泛型算法的实参。

#### 14.8.1 lambda是函数对象

编写一个lambda后，编译器将表达式翻译成一个未命名类的未命名对象。在lambda表达式产生的类中含有一个重载的函数调用运算符。

```cpp
stable_sort(words.begin(), words.end(), [](const string &a, const string &b){
    return a.size() < b.size();
});

//其行为类似如下类的对象
class ShorterString {
public:
    bool operator()(const string &s1, const string &s2) const {
        return s1.size() < s2.size();
    }
}
```

函数调用运算符的形参列表和函数体与lambda表达式完全一样。默认情况下lambda不能改变它捕获的变量，函数调用运算符是一个const成员函数。如果lambda被声明为可变的，则调用运算符就并不是const的。

**表示lambda及相应捕获行为的类**

lambda表达式通过引用捕获变量时，要确保执行时所引的对象确实存在。然而通过值捕获的变量被拷贝到lambda中。

```cpp
auto wc = find_if(words.begin(), words.end(), 
                  [sz](const string &a) {return a.size() > sz;});


//lambda表达式产生的类
class SizeComp {
    SizeComp(size_t n): sz(n) {}
    //lambda表达式
    bool operator()(const string &s) const {
        return s.size() >= sz;
    }
private:
    size_t sz;//值捕获对象
}
```

#### 14.8.2 标准库定义的函数对象

标准库定义了一组表示算数运算符、关系运算符和逻辑运算符的类，每个类分别定义了一个执行命名操作的调用运算符。在头文件functional中

**表14.2 标准库函数对象**

| 算术              | 关系                 | 逻辑               |
| ----------------- | -------------------- | ------------------ |
| plus\<Type>       | equal_to\<Type>      | logical_and\<Type> |
| minus\<Type>      | not_equal_to\<Type>  | logical_or\<Type>  |
| multiplies\<Type> | greater\<Type>       | logical_not\<Type> |
| divides\<Type>    | greater_equal\<Type> |                    |
| modulus\<Type>    | less\<Type>          |                    |
| negate\<Type>     | less_equal\<Type>    |                    |

**在算法中使用标准库函数对象**

```cpp
// 降序排列
sort(svec.begin(), svec.end(), greater<string>());

vector<string*> nameTable;
//标准库规定其函数对象对于指针同样适用
sort(nameTable.begin(), nameTable.end(), less<string*>());
```

#### 14.8.3 可调用对象与function

C++中可调用对象种类：函数、函数指针、lambda表达式、bind创建的对象以及重载了函数调用运算符的类。可调用对象也有类型。比如lambda有未命名类类型；函数和函数指针的类型为返回值类型和实参类型等。

两个不同类型的可调用对象可能共享同一种`调用形式`。调用形式指明了调用返回的类型以及传递给调用的实参类型。

**不同类型可能具有相同的调用形式**

```cpp
//调用形式 int(int, int)
int add(int i, int j) {return i + j;}

//虽然调用形式相同，但是lambda有它自己的类类型，跟函数类型不同
auto mod = [](int i, int j) {return i % j;}

struct divide {
    int operator()(int denominator, int divisor) {
        return denominator / divisor;
    }
}
```

**标准库function类型**

头文件functional，**表14.3 function的操作**

| function\<T> f;                                              | f是一个用来存储可调用对象的空function，这些可调用对象的调用形式应该与函数类型T相同（T为retType(args)） |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| function\<T> f(nullptr);                                     | 显式构造一个空function                                       |
| function\<T> f(obj);                                         | 在f中存储可调用对象obj的副本                                 |
| f                                                            | 将f作为条件：当f含有一个可调用对象时为真；否则为假           |
| f(args)                                                      | 调用f中的对下给你，参数是args                                |
| 定义为function\<T>的成员的类型                               |                                                              |
| result_type                                                  | 该function类型的可调用对象返回的类型                         |
| argument_type<br />first_argument_type<br />second_argument_type | 当T有一个或两个实参时定义的类型。如果T只有一个实参，则argument_type时该类型的同义词；如果T有两个实参，则first_argument_type和second_argument_type分别代表两个实参的类型 |

```cpp
function<int(int, int)> f1 = add;
function<int(int, int)> f2 = divide();
function<int(int, int)> f3 = [](int i, int j) {return i * j;};
```

**重载的函数与function**

```cpp
int add(int i, int j) {return i + j;};
Sales_data add(const Sales_data&, const Sales_data&);

map<string, function<int(int, int)>> binops;
binops.insert({"+", add}); // 产生二义性

//使用函数指针解决
int (*fp)(int, int) = add;
binops.insert({"+", fp});
```

> 新版本标准库中的function类与旧版本中的unary_function和binary_function没有关联，后两个类已经被更通用的bind函数替代了

### 14.9 重载、类型转换与运算符

转换构造函数和类型转换运算符共同定义了`类类型转换`，这样的转换有时也被称作`用户定义的类型转换`

#### 14.9.1 类型转换运算符

它负责将一个类类型的值转换成其他类型。

```cpp
operator type() const;
```

类型转换运算符可以面向任意类型（除void之外）进行定义，只要该类型能作为函数的返回类型。因此不允许转换成数组或者函数类型。

> 一个类型转换函数必须是类的成员函数；不能声明返回类型，形参列表也必须为空。类型转换函数通常不改变对象的内容，是const

**定义含有类型转换运算符的类**

```cpp
//构造函数将算术类型的值转换成SmallInt对象，而类型转换运算符将SmallInt对下给你转换成int
class SmallInt {
public:
    SmallInt(int i = 0): val(i) {
        if(i < 0 || i > 255)
            throw std::out_of_range("Bad SmallInt value");
    }
    operator int() const {return val;}
private:
    std::size_t val;
};
```

虽然编译器一次只能执行一个用户定义的类型转换，但是转成成内置类型后可以通过内置的类型转换继续转换成任何其他算术类型

```cpp
SmallInt si = 3.14;
si + 3.14; //将si转换成int，内置类型转换将所得的int继续转换成double
```

因为类型转换运算符是隐式执行的，所以无法给这些函数传递实参，也就不能在类型转换运算符的定义中使用形参。尽管类型转换函数不负责指定返回类型，但实际上每个类型转换函数都会返回一个对应类型的值。

**显式的类型转换运算符**

一般类很少提供类型转换运算符，除了定义向bool的类型转换。然而类型转换运算符可能产生意外结果。如果istream含有向bool的类型转换时，下面代码将编译通过

```cpp
int i = 42;
cin << i; //将输出运算符作用于输入流，本来应该报错。但是通过类型转换，cin被转换成bool类型，接着会被提升为int，作为内置的左移运算符的左侧运算符对象

//显式类型转换运算符
class SmallInt {
public:
    explicit operator int() const {return val;}
}
```

值得注意的是，当表达式在以下位置时，显式的类型转换将被隐式执行

- if、while及do语句的条件部分
- for 语句头的条件表达式
- 逻辑运算符的运算对象
- 条件运算符的条件表达式（？：）

**转换为bool**

标准库的早期版本，IO类型定义了向void*的转换规则以避免上面提到的问题。C11新标准下通过定义向bool的显式类型转换实现相同目的。

#### 14.9.2 避免有二义性的类型转换

- 不要令两个类执行相同的类型转换，即A接受B类对象的构造函数，B类定义转换目标为A的转换运算符
- 避免转换目标是内置算术类型的类型转换，除了显式转换为bool

```cpp
struct C {
    C(int);
}
struct E {
    E(double);
}
void manip2(const C&);
void manip2(const E&);
//二义性错误，调用重载函数时，不会考虑准表类型转换级别，会认为这些类型转换都一样好
//只有当重载函数能通过同一个类型转换函数得到匹配时，才会考虑其中出现的标准类型转换
manip2(10);
```

#### 14.9.3 函数匹配和重载运算符

当调用一个命名的函数时，具有该名字的成员函数和非成员函数不会彼此重载，这是因为调用它们的语法形式是不同的。而当使用重载的运算符时，这两者都在考虑的范围内。

```cpp
class SmallInt {
public:
    friend SmallInt operator+(const SmallInt&, const SmallInt&);
    SmallInt(int i = 0): val(i);
    operator int() const {return val;}
private:
    std::size_t val;
};
SmallInt s;
int i = s + 0;//产生二义性
```

0可以转换成SmallInt使用SmallInt的+，SmallInt也能转换成int执行内置的加法运算。

## 15 面向对象程序设计

动态绑定有时又被称为`运行时绑定`，在使用基类的引用（或指针）调用一个虚函数时发生

### 15.２ 定义基类和派生类

#### 15.2.1 定义基类

```cpp
class Quote {
public:
    Quote() = default;
    Quote(const std::string &book, double sales_peice):
        bookNo(book), price(sales_peice) { };
    std::string isbn() const {return bookNo;};
    virtual double net_peice(std::size_t n) const {
        return n * price;
    }
    virtual ~Quote() = default;
private:
    std::string bookNo;
protected:
    double price = 0.0;
};
```

> 基类通常都应该定义一个虚析构函数，即使该函数不执行任何实际操作

使用指针或引用调用虚函数时，该调用将被动态绑定。任何构造函数之外的非静态函数都可以是虚函数。

成员函数如果没被声明为虚函数，则其解析过程发生在编译时而非运行时。

#### 15.2.2 定义派生类

派生类必须通过使用`类派生列表`明确指出基类。

```cpp
class Bulk_quote: public Quote {
public:
    Bulk_quote() = default;
    Bulk_quote(const std::string&, double, std::size_t, double);
    double net_price(std::size_t) const override;
private:
    std::size_t min_qty = 0;
    double discount = 0.0;
};
```

**派生类中的虚函数**

在const关键字后面、或者在引用成员函数的引用限定符后面添加关键字override，来显式注明它使用某个成员函数覆盖继承的虚函数。

**派生类对象及派生类向基类的类型转换**

派生类对象包含基类的子对象，所以能把派生类对象当成基类对象来使用。C++标准没有明确规定派生类的对象在内存中如何分布，不一定为连续存储的。

**派生类构造函数**

通过构造函数初始化列表来将实参传递给基类构造函数

```cpp
Bulk_quote::Bulk_quote(const std::string &book, double p,
                       std::size_t qty, double disc):
                       Quote(book, p), min_qty(qty), discount(disc) {};
```

首先初始化基类部分，然后按照声明的顺序一次初始化派生类的成员

**继承与静态成员**

如果基类定义了一个静态成员，则在整个继承体系中只存在该成员的唯一定义。静态成员遵循通用的访问控制规则。

**派生类的声明不包含派生列表**

**被用作基类的类**

类只有被定义了才能被当作基类，所以类不能派生它本身。

**防止继承的发生**

```cpp
class NoDerived final {}; //NoDerived不能作为基类
class Last final: Base {}; // final不能作为基类
```

#### 15.2.3 类型转换与继承

可以将基类的指针或引用绑定到派生类上。这意味着，当使用基类的引用（或指针）时，实际上并不清楚该引用（或指针）所绑定对象的真实类型。

**静态类型和动态类型**

表达式的静态类型在编译时总是已知的，动态类型直到运行时才可知。只有基类的指针或引用的静态类型可能与其动态类型不一致

**不存在从基类向派生类的隐式类型转换，在对象之间不存在类型转换**

派生类对象都包含一个基类部分，基类的引用或指针可以绑定到该基类部分上。可以使用dynamic_cast请求类型转换，该转换的安全检查将在运行时执行。也可以在已知安全的情况下，使用static_cast强制转换。

派生类向基类的自动类型转换只对指针或引用类型有效。值得注意的是，当初始化或赋值一个类类型的对象时，实际上是在调用某个函数，如构造函数、赋值运算符。这些函数通常为类类型的const版本的引用。所以允许给基类的拷贝/移动操作传递一个派生类对象。这些操作不是虚函数，实际运行的版本为基类版本，同样也只能处理基类自己的成员，派生类部分被忽略掉。

### 15.3 虚函数

基类中的虚函数在派生类中隐含得也是一个虚函数

**final和override说明符**

如果使用override标记某个函数，但该函数并没有覆盖已存在的虚函数，此时编译器将报错。

```cpp
struct B {
    virtual void f1(int) const;
};
struct D1: B {
    void f1(int) const override;
};
```

指定函数为final，之后任何尝试覆盖该函数的操作都将引发错误

```cpp
struct B {
    virtual void f1(int) const;
};
struct D2: B {
    void f1(int) const final;
};
```

如果某次函数调用使用了默认实参，则该实参值由调用的静态类型决定。即如果荣国基类的引用或指针调用函数，则使用基类中定义的默认实参。

**回避虚函数的机制**

使用作用域运算符显式指定。该调用将在编译时完成解析。

```cpp
double undiscounted = baseP->Quote::net_price(42);
```

### 15.4 抽象基类

**纯虚函数**

纯虚函数的函数体必须定义在类的外部

```cpp
class Disc_quote: public Quote {
public:
    Disc_quote() = default;
    Disc_quote(const std::string& book, double price,
               std::size_t qty, double disc):
                    Quote(book, price),
                    quantity(qty), discount(disc) { };
    //纯虚函数
    virtual double net_price(std::size_t) const = 0;
protected:
    std::size_t quantity = 0;
    double discount = .0;
};
```

**含有纯虚函数的类是抽象基类**

抽象基类不能实例化

> 重构: 重构负责重新设计类的体系以便将操作或数据从一个类移动到另一个类中。
>

### 15.5 访问控制与继承

**受保护的成员**

- 对类的用户不可访问
- 对派生类的成员和友元是可访问的

**公有、私有和受保护继承**

派生访问说明符的目的是控制派生类用户（包括派生类的派生类）对于基类成员的访问权限。

**派生类向基类转换的可访问性**

- 只有当派生类公有地继承基类时，用户代码才能使用派生类向基类的转换，第三条规则除外
- 派生类的成员函数和友元都能使用派生类向基类的转换。
- 当派生类继承基类的方式为公有或受保护的，派生类的派生才能使用派生类向基类的转换

> 总之，类的对象只能访问类的公有（接口）成员。派生类及友元不能访问私有成员，可以访问公有成员和受保护的成员

**友元与继承**

友元关系不能传递，也不能继承。派生类的友元不能随意访问基类的成员。

**改变个别成员的可访问性**

使用using声明改变访问级别

```c++
class Base {
public:
    std::size_t size() const {
        return n;
    }
protected:
    std::size_t n;
};
class Derived: private Base {
public:
    using Base::size;
protected:
    using Base::n;
};
```

### 15.6 继承中的类作用域

当存在继承关系时，派生类的作用域嵌套在其基类的作用域之内

**在编译时进行名字查找**

能使用的成员由静态类型决定

**名字冲突与继承**

内层作用域（即派生类）的名字将隐藏定义在外层作用域（基类）的名字。

**可以通过作用域运算符来使用隐藏的成员**

> 名字查找与继承
>
> 假定调用p->mem() 或 obj.mem()，会执行以下四个步骤
>
> - 先确定p（或obj）的静态类型
> - 在p（或obj）的静态类型对应的类中查找mem。如果找不到，则顺着继承链往基类查找，若仍找不到，则报错。
> - 一旦找到mem，就进行常规的类型检查，确认本次调用是否合法
> - 若合法，则根据调用的是否为虚函数而产生不同代码
> - - mem为虚函数且通过引用或指针调用，则为动态类型
>   - 否则产生一个常规函数调用

**名字查找先于类型检查**

内层作用域的函数不会重载外层作用域的函数。即派生类中的函数不会重载其基类中的成员。即使派生类成员和基类成员的形参列表不一致，基类成员也会被隐藏掉。

```cpp
struct Base {
    int memfcn();
};
struct Derived: public Base {
    int memfcn(int);
};
Derived d;
d.memfcn(); // error
```

**虚函数与作用域**

```c++
class Base {
public:
    virtual int fcn();
};
class D1: public Base {
public:
    // 隐藏基类的fcn，这个fcn不是虚函数
    // D1 继承了 Base::fcn() 的定义
    int fcn(int); // 形参列表与Base中的fcn不一致
    virtual void f2(); // 新的虚函数
};
class D2: public D1 {
public:
    int fcn(int); // 是一个非虚函数，隐藏了D1::fcn(int)
    int fcn(); // 覆盖了Base的虚函数fcn
    void f2(); // 覆盖了D1的虚函数f2
};
```

**可通过基类调用隐藏的虚函数**

**覆盖重载的函数**

> 当派生类需要重载基类的某个版本，但又希望保留基类的其他版本（即不隐藏基类的同名函数），可以使用**using声明语句**指定名字而不指定形参列表，这样就能将该函数的所有重载实例添加到派生类作用域中

### 15.7 构造函数与拷贝控制

如果类没有定义拷贝控制操作，则编译器会自动合成一个版本，或者定义成delete

#### 15.7.1 虚析构函数

当delete以动态分配的对象的指针时将执行析构函数。如果该指针指向继承体系中的某个类型，则有可能出现指针的静态类型与被删除对象的动态类型不符的情况。

将析构函数定义成虚函数以确保执行正确的析构函数版本

```cpp
class Quote {
public:
    // 需要虚析构函数，来正确调用派生类的析构函数
    virtual ~Quote() = default;
};
```

和其他虚函数一样，析构函数的虚属性也会被继承

> delete一个指向派生类对象的基类指针将产生未定义的行为

> 如果一个类需要析构函数，那么也同样需要拷贝和赋值操作。基类的析构函数为特例，并不遵循此准则。因为无法推断基类是否还需要赋值运算符或拷贝构造函数（或者说当类成员存在指针时，就需要自定义拷贝和赋值操作）

**虚析构函数将阻止合成移动操作**

#### 15.7.2 合成拷贝控制与继承

合成的拷贝控制成员、构造函数、赋值运算符和析构函数还负责对直接基类部分执行对应的操作。

对于派生类的析构函数来说，除了销毁自身的成员外，还负责销毁其直接基类的成员。

**派生类中删除的拷贝控制与基类的关系**

导致派生类成员成为被删除的函数

- 基类中的默认构造函数、拷贝构造函数、拷贝赋值运算符或析构函数是被删除的或不可访问，导致基类部分无法进行构造、赋值或销毁操作。
- 基类中的移动操作是删除或不可访问，则派生类中的该函数也将是被删除的。若基类析构函数是删除的或不可访问的，则派生类的移动构造函数也是被删除的。

#### 15.7.3 派生类的拷贝控制成员

派生类的构造函数、拷贝和移动构造函数，赋值运算符都要负责基类部分，而析构函数只负责销毁派生类自己分配的资源

> 当派生类定义了拷贝或移动操作时，该操作负责拷贝或移动包括基类部分成员在内的整个对象

**定义派生类的拷贝或移动拷贝函数**

```cpp
class Base {};
class D: public Base {
public:
    // 默认情况下，基类的默认构造函数初始化对象的基类部分
    // 想要使用拷贝或移动构造函数，必须在构造函数初始值列表中
    // 显式调用该构造函数
    D(const D& d): Base(d) // 拷贝基类成员
    {}
    D(D&& d): Base(std::move(d)) // 移动基类成员
    {}
};
```

Base(d)一般会匹配Base 的拷贝构造函数。如果拷贝构造函数没有显示拷贝基类成员，基类部分将会使用默认构造函数进行初始化，而子类成员的值是从其他对象拷贝得来的。

**派生类赋值运算符**

派生类的赋值运算符必须显示为基类部分赋值

```cpp
//Base::operator=(const Base&); 不会被自动调用
D& D::operator=(const D& rhs) {
    Base::operator=(rhs); // 为基类部分赋值
    // ...
    return *this;
}
```

> 要注意的是，无论基类的构造函数或赋值运算符是自定义的版本还是合成的版本，派生类的对应操作都能使用它们。如执行Base::operator=的调用语句，与该运算符是否为Base自动合成无关

**派生类析构函数**

与拷贝构造函数、移动构造函数和赋值运算符不同，派生类析构函数只负责销毁派生类自己分配的资源

```cpp
class D: public Base {
public:
    // Base::~Base 被自动调用执行
    ~D() {}
}
```

**在构造函数和析构函数中调用虚函数**

如果构造函数或析构函数调用了某个虚函数，应该执行与构造函数或析构函数所属类型相对应的虚函数版本

#### 15.7.4 继承的构造函数

构造函数并非以常规的方式继承而来，为了方便，姑且称其为“继承”。类不能继承默认、拷贝和移动构造函数。如果派生类没有直接定义这些构造函数，则编译器将合成它们。

派生类继承基类构造函数的方式是提供注明了（直接）基类名的using声明语句。

```cpp
// 继承Disc_quote类的构造函数
class Bulk_quote: public Disc_quote {
public:
    using Disc_quote::Disc_quote; // 继承Disc_quote的构造函数
    double net_price(std::size t) const;
}
```

当using声明语句作用于构造函数时，using声明语句将令编译器产生代码。对于基类的每个构造函数，编译器在派生类中生成一个形参列表完全相同的构造函数。形如`derived(parms): base(args) {}`

**继承的构造函数的特点**

构造函数的using声明不会改变该构造函数的访问级别。不论using声明在哪，基类的私有构造函数在派生类中还是私有。而且，using声明语句不能指定explicit或constexpr。

当基类的构造函数含有默认实参时，这些实参并不会被继承。相反，派生类将获得多个继承的构造函数，这些构造函数包括分别省略掉一个含有默认实参的形参。

派生类中重写的构造函数将不会被继承，默认、拷贝和移动构造函数不会被继承。继承的构造函数不会被用作用户定义的构造函数来使用，因此，如果一个类只含有继承的构造函数，则将拥有合成的默认构造函数

### 15.8 容器与继承

容器中一般存放基类的指针。派生类的智能指针可以被自动转换成基类的智能指针

#### 15.8.1 编写Basket类

**模拟虚拷贝**

```cpp
class Quote {
public:
    // 返回当前对象动态分配的拷贝
    virtual Quote* clone() const & {return new Quote(*this);}
    virtual Quote* clone() && {return new Quote(std::move(*this));}
};
class Bulk_quote: public Quote {
    virtual Bulk_quote* clone() const & {return new Bulk_quote(*this);}
    virtual Bulk_quote* clone() && {return new Bulk_quote(std::move(*this));}
};
```

clone返回与clone所属类型一致的对象

