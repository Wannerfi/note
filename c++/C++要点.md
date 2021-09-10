第一章到第十四章。。
### constexpr和常量表达式
常量表达式是指值不会改变并且在编译过程就能得到计算结果的表达式
[与const的区别](https://zhuanlan.zhihu.com/p/20206577)
**限定符constexpr仅对指针有效，对指针所指的对象无关**
```cpp
constexpr int *q = nullptr;//指向整数的指针常量
```
### 类型别名
```cpp
typedef char *pstring;
const pstring cstr = nullptr;//cstr是指向char的指针常量
const char *cstr = nullptr;//是对const pstring cstr的错误理解
```
前者的基本数据类型是指针，const修饰指针，为指向char的指针常量
后者的基本数据类型是const char，为指向常量的指针

### auto类型说明符
auto会忽略顶层const，保留底层const

### decltype类型指示符
```cpp
decltype(f()) a;//a的类型就是函数f的返回类型
```
decltype保留顶层const和引用
如果表达式的内容是解引用操作，decltype将得到引用类型
decltype中变量名加上一层以上的括号，编译器会把它当成表达式，变量是一种可以作为赋值语句左值的特殊表达式，所以decltype会得到引用类型
```cpp
int p;
decltype((p)) a = p;//int &a = p;
```

**左值和右值**

C++表达式不是左值就是右值。左值可以位于赋值语句的左侧，右值则不能

使用关键字decltype的时候，左值和右值也有所不同。
int *p;
decltype(*p) == int &
decltype(&p) == int **

### 标准库类型vector
```cpp
vector<string> v{10, "hi"};
```
确认无法执行列表初始化后，编译器会尝试用默认值初始化vector对象，默认值初始化建议采用圆括号
### 遍历迭代器的过程中对迭代器所属容器进行添加或删除会改变迭代时要更新迭代器

### 数组
```cpp
int arr[] = {0, 1, 2, 3};
auto a2 = arr;//int *，实际初始化过程为auto a2 = &arr[0];
decltype(arr) a3 = {4, 5, 6, 7};//int[]
```
```cpp
//指针也是迭代器
int arr[] = {0, 10, 20};
int *p = begin(arr);
++p;//p -> arr[1]
```
### 与旧代码的接口
```cpp
string s = "string";
auto p = s.c_str();
```
### 多维数组
使用范围for处理多维数组，除了最内层的循环外，其他循环的控制变量都应该是引用类型。使用引用是为了避免数组被自动转成指针
```cpp
int arr[3][4] = {};
for(auto &i : arr) {
	for(auto j : i) {
		cout << j << " ";
	}
}
```
### sizeof运算符
它是运算符
```cpp
sizeof expr
sizeof(type)
cout << sizeof 3 << endl;//4
```
### 尾置返回类型
```cpp
//原来的返回类型地方用auto
auto func(int i) -> int(*)[10];//返回指向10个int的指针

//使用decltype
int arr[5];
decltype(arr) *arrPtr(int i) {
    return &arr;
}
```

### 调试帮助
assert预处理宏。首先对expr求值，如果表达式为假，assert输出信息并终止程序的执行。如果表达式为真，assert什么也不做。
```cpp
assert(expr);
```

### 函数指针
```cpp
int Calculate(int, int);
int (*p)(int, int);//指向函数int (int, int)的指针
p = Calculate;//指向函数Calculate
p = &Calculate;//同上
p(3, 2);//Calculate(3, 2);

int Calculate(int a, int b) {
    return a * b;
}
//decltype返回函数类型，不会自动转换成指针类型
int solve(decltype(Calculate) f, int a, int b) {
    return f(a, b);
}

using F = int(int, int);//函数类型
using PF = int(*)(int, int);//指针类型
auto f(int a) -> int (*)(int, int) {
    return Calculate;
};
```
### 构造函数
如果没有显式定义构造函数，那么编译器会隐式定义一个默认构造函数。
### 拷贝、赋值和析构
如果不主动定义拷贝、赋值、析构操作，编译器会自动合成。有些类不能依赖合成的版本，比如管理动态内存的类
### 可变数据成员
声明符mutable声明的成员永远不会是const
```cpp
class Screen {
public:
	void some_member() const {
        //尽管some_member为const成员函数，access_ctr的值也能改变
        ++access_ctr;//记录成员函数被调用的次数
    };
private:
    mutable size_t access_ctr;
};
```
### 类类型
一旦一个类的名字出现后，它就被认为是声明过了。声明过的类，就可以使用它的引用或指针类型。

### 类的作用域
一般内层作用域可以重新定义外层作用域中的名字。然而在类中，如果成员使用了外层作用域中的某个名字，而该名字代表一种类型，则类不能在之后重新定义该名字
```cpp
typedef double Money;
class Account {
public:
    Money balance() {return bal;}//使用外作用域的Money
private:
    typedef double Money;//错误，不能重新定义Money,一些编译器并不为此负责，可顺利编译
    //忽略代码有错的事实
    Money bal;
}
```
### 委托构造函数
把它自己的一些（或者全部）职责委托给了其他构造函数
```cpp
class Sales_data {
public:
    Sales_data(std::string s, unsigned cnt, double price): 
    bookNo(s), units_sold(cnt), revenue(cnt * price) {}
    //其余构造函数全部委托给另一个构造函数
    Sales_data(): Sales_data("", 0, 0) {}
    Sales_data(std::string s): Sales_data(s, 0, 0) {}
    Sales_data(std::istream &is): Sales_data() {read(is, *this)}
    //如果受委托的函数体包含代码的话，先执行受委托的函数体的代码
    //然后控制权才会交还给委托者的函数体
}
```
### 隐式的类类型转换
如果构造函数只接受一个实参，则它实际上定义了转换为此类类型的隐式转换机制，这种构造函数有时称作转换构造函数。这种转换只执行一次，所以不总是有效。
```cpp
class Sales_data {
public:
    //...
    Sales_data(): Sales_data("", 0, 0){}
    Sales_data(std::istream &is): Sales_data() {read(is, *this)}
    Sales_data &combine(const Sales_data &);
}

//错误，需要先将"9-99-99"转换成string，再从string转换成Sale_data
item.combine("9-99-99");

//显示转换
class Sales_data {
public:
    Sales_data() = default;
    Sales_data(std::string s, unsigned cnt, double price):
            bookNo(s), units_sold(cnt), revenue(cnt * price) {};
    //显式转换
    explicit Sales_data(const std::string &s): bookNo(s) { };
    explicit Sales_data(std::istream &);
}

```
关键字explicit只对一个实参的构造函数有效。需要多个实参的构造函数不能用于执行隐式转换，所以无须将这些构造函数制定为explicit
explicit关键字声明构造函数时，它将只能以直接初始化的形式使用，而且不会再自动转换过程中使用该构造函数

### 聚合类
聚合类使得用户可以直接访问其成员，并且具有特殊的初始化语法形式。当一个类满足如下条件时，我们说它是聚合的：
所有成员都是public
没有定义任何构造函数
没有类内初始值
没有基类，也没有virtual函数
```cpp
struct Data {
    int ival;
    string s;
};
//提供一个花括号括起来的成员初始化列表，并用它初始化聚合类的数据成员
Data vall = {0, "Anna"};
```

### 字面值常量类
类符合下述要求，则它是一个字面常量类:
数据成员都必须是字面值类型；
至少含有一个constexpr构造函数；
类型成员的初始值必须是一条常量表达式或constexpr构造函数。
类必须使用析构函数的默认定义，该成员负责销毁类的对象；
```cpp
class Debug {
public:
    constexpr Debug(bool b = true): hw(b), io(b), other(b) {}
    constexpr Debug(bool h, bool i, bool o): hw(h), io(i), other(o) {}
    constexpr bool any() {return hw || io || other;}
    void set_io(bool b) {io = b;}
    void set_hw(bool b) {hw = b;}
    void set_other(bool b) {hw = b;}
private:
    bool hw;
    bool io;
    bool other;
};
```
### IO库
大部分IO库设施

- istream（输入流）类型，提供输入操作
- ostream（输出流）类型，提供输出操作
- cin，一个istream对象，从标准输入读取数据
- cout，一个ostream对象，向标准输出写入数据
- cerr，一个ostream对象，通常用于输出程序错误消息，写入到标准错误
- \>>运算符，用来从一个istream对象读取输入数据。
- \<<运算符，用来向一个ostream对象写入输出数据
- getline函数，从一个给定的istream读取一行数据，存入一个给定的string对象中

| 头文件   | 类型                                                         |
| -------- | ------------------------------------------------------------ |
| iostream | istream，wistream 从流中读取数据<br />ostream, wostream 向流写入数据<br />iostream, wiostream 读写流 |
| fstream  | ifstream，wifstream 从文件读取数据<br />ofstream，wofstream 向文件写入数据<br />fstream，wfstream 读写文件 |
| sstream  | istringstream，wistringstream  从string读取数据<br />ostringstream，wostringstream 向string写入数据<br />stringstream，wstringstream 读写string |

类型ifstream和istringstream都继承自istream

读写一个IO对象会改变其状态。

条件状态
|                   |                                                              |
| ----------------- | ------------------------------------------------------------ |
| strm::badbit      | strm::badbit 用来指出流已崩溃                                |
| strm::failbit     | strm::failbit 用来指出一个IO操作失败了                       |
| strm::eofbit      | strm::eofbit 用来指出流到达了文件结束                        |
| strm::goodbit     | strm::goodbit 用来指出流未处于错误状态。此值保证为0          |
| s.eof()           | 若流 s 的 eofbit 置位，则返回 true                           |
| s.fail()          | 若流 s 的 failbit 或 badbit 置位，则返回 true                |
| s.bad()           | 若流 s 的 badbit 置位，则返回 true                           |
| s.good()          | 若流 s 处于有效状态，则返回 true                             |
| s.clear()         | 将流 s 中所有条件状态位复位，将流的状态设置为有效。返回void  |
| s.clear(flags)    | 根据给定的flags标志位，将流 s 中对应条件状态位复位。flags的类型为strm::iostate。返回void |
| s.setstate(flags) | 根据给定的flags标志位，将流 s 中对应条件状态位置位。flags的类型为strm::iostate。返回void |
| s.rdstate()       | 返回流 s 的当前条件状态，返回值类型为strm::iostate           |

导致缓冲刷新的原因
- 程序正常结束，作为main函数的return操作的一部分，缓冲刷新被执行
- 使用操纵符如 endl 来显式刷新缓冲区
- 写到cerr的内容都是立即刷新的
- 一个输出流可能被关联到另一个流。在这种情况下，当读写被关联的流时，关联到的流的缓冲区会被刷新。默认情况下，cin和cerr都关联到cout。因此读cin或写cerr都会导致cout的缓冲区被刷新。

### 顺序容器
|              |                                                              |
| ------------ | ------------------------------------------------------------ |
| vector       | 可变大小数组。支持快速随机访问。在尾部之外的位置插入或删除元素可能很慢 |
| deque        | 双端队列。支持快速随机访问。在头尾位置插入/删除速度都快      |
| list         | 双向链表。只支持双向顺序访问。在list中任何位置进行插入/删除操作速度都很快 |
| forward_list | 单向链表。只支持单向顺序访问。在链表任何位置进行插入/删除操作速度都很快 |
| array        | 固定大小数组。支持快速随机访问。不能添加或删除元素           |
| string       | 与vector相似的容器，但专门用于保存字符。随机访问快。在尾部插入/删除速度快 |

**使用swap**
除array外，交换两个容器内容的操作保证会很快——元素本身并未交换，swap只是交换了两个容器的内部数据结构，所以很快


|                                              | 这些操作会改变容器的大小；array不支持这些操作                |
| -------------------------------------------- | ------------------------------------------------------------ |
| c.push_back(t)<br />c.emplace_back(args)     | 在c 的尾部创建一个值为t 或由args 创建的元素。返回void        |
| c.push_front(t)<br />c.emplace_front(args)   | 在c 的头部创建一个值为t 或由args 创建的元素。返回void        |
| c.insert(p, t)<br />c.emplace_front(p, args) | 在迭代器p指向的元素之前创建一个值为t 或由args 创建的元素。返回指向新添加的元素的迭代器 |
| c.insert(p, n, t)                            | 在迭代器p指向的元素之前插入n 个值为t 的元素。返回指向新添加的元素的迭代器；若n 为0，则返回p |
| c.insert(p, b, e)                            | 将迭代器b 和 e 指定的范围内的元素插入到迭代器p 指向的元素之前。b 和e 不能指向c 中的元素。返回指向新添加的第一个元素的迭代器；若范围为空，则返回p |
| c.insert(p, i1)                              | i1 是一个花括号包围的元素值列表。将这些给定值插入到迭代器p 指向的元素之前。返回指向新添加的第一个元素的迭代器；若列表为空，则返回p |

**使用insert 的返回值**
通过使用insert 的返回值，可以在容器中的一个特定位置反复插入元素
```c++
list<string> lst;
auto iter = lst.begin();
while(cin >> word)
    iter = lst.insert(iter, word);

vector<int> v{1, 1, 1, 2, 2, 3, 4, 5};
for(auto it = v.begin();it != v.end();++it) {
	if((*it) == 2) {
    	it = v.insert(it, 10);
        ++it;
	}
}
```

**使用emplace操作**
新标准引入三个新成员：emplace_front, emplace, emplace_back，这些操作构造而不是拷贝元素。
当调用push或insert 成员函数时，我们将元素类型的对象传递给他们，这些对象被拷贝到容器中。而当我们调用一个emplace成员函数时，则是将参数传递给元素类型的构造函数。emplace成员使用这些参数在容器管理的内存空间中直接构造元素。
```c++
//c保存Sales_data元素
//在c 的末尾构造一个Sales_data对象，使用三个参数的Sales_data构造函数
c.emplace_back("545645", 54, 5.36);
//错误，没有接受三个参数的push_back版本
c.push_back("54", 547, 2.3);
//正确，创建一个临时的Sales_data对象传递给push_back
c.push_back(Sales_data("54", 5, 3.3));
```

**删除元素**
```c++
vector<int> v{1, 1, 1, 2, 2, 3, 4, 5};
for(auto t = v.begin();t != v.end();) {
	if((*t) == 1) {
    	t = v.erase(t);
	} else {
    	++t;
	}
}
```

**string的其他操作**
str.insert()
str.replace()
str.find()
...

**数值转换**
stod() -> string to double
atoi() -> char* to int
...

**容器适配器**
三个顺序容器适配器：stack, queue和priority_queue

### 泛型算法
> 算法永远不会执行容器的操作
泛型算法本身不会执行容器的操作，只会运行于迭代器之上，执行迭代器的操作。该特性带来了一个必要的编程假定：算法永远不会改变底层容器的大小。
标准库定义了一类特殊的迭代器，插入器（inserter）。当给这类迭代器赋值时，迭代器可以完成向容器添加元素的效果，但算法自身永远不会做这样的操作

```CPP
//计算vec中元素的和，和的初值是0
int sum = accumulate(vec.cbegin(), vec.cend(), 0);

//前两个表示第一个序列中元素范围，第三个表示第二个序列的首元素
//roster2重点元素数目至少于roster1一样多
equal(roster1.cbegin(), roster1.cend(), roster2.cbegin());
//假定比较运算符可执行

/接受一对迭代器表示范围，接受一个值作为第三个参数
fill(vec.begin(), vec.end(), 0);

//接受一个单迭代器、一个计数值、一个值，将给定值赋予迭代器指向的元素开始的指定个元素
fill_n(vec.begin(), vec.size(), 0); //重置为0
fill_n(dest, n, val);//算法不检查写时，dest容量大小

copy(begin(a), end(a), b);//把a的内容拷贝给b

//多个算法都提供所谓的“拷贝”版本。这些算法计算新元素的值，但不会将它们放置在输入序列的末尾，而是创建一个新序列保存这些结果。
//前两个是迭代器，表示输入序列，后两个一个是要搜索的值，另一个是新值
replace(ilst.begin(), ilst.end(), 0, 42);
//保持原序列不变。接受额外第三个迭代器参数，指出调整后序列的保存位置
//ilst未改变，ivec包含ilst的拷贝，并调整了新值
replace_copy(ilst.begin(), ilst.end()
            back_inserter(ivec), 0, 42);

//按字典序排序words
sort(words.begin(), words.end());
//使用unique
auto end_unique = unique(words.begin(), words.end());
//删除重复单词
words.erase(end_unique, words.end());
```

**定制操作**
谓词是一个可调用的表达式，其返回结果是一个能作为条件的值
```cpp
//stable_sort，除了对按照谓词规则排序外，保持元素的相对位置不变
stable_sort(words.begin(), words.end(), isShorter);
```
lambda捕获和返回
```cpp
//值捕获：被捕获的变量的值是在lambda创建时拷贝，而不是调用时拷贝
size_t v1 = 42;
auto f =[v1] {return v1;};
v1 = 0;
auto j = f();//j为42，保存了创建时v1的拷贝


//采用引用方式捕获一个变量，必须保证被引用的对象在lambda执行的时候是存在的
size_t v1 = 42;
auto f =[v1] {return v1;};
v1 = 0;
auto j = f();//j为0

//默认情况下，对于值拷贝的变量，lambda不会改变其值。如果希望能改变，必须在参数列表首加上mutable关键字。
size_t v1 = 42;
auto f = [v1]() mutable {return ++v1;};
v1 = 0;
auto j = f();//j为43
```
**bind函数**
```cpp
auto newCallable = bind(callable, arg_list);
```
### 插入迭代器
插入器是一种迭代器适配器，它接受一个容器，生成一个迭代器，能实现给定容器添加元素。
```cpp
*it = val;
//等同于
it = c.insert(it, val);//it指向新加入的元素
++it;//递增it使它指向原来的元素
```
### iostream迭代器
通过使用流迭代器，可以用泛型算法从流对象读取数据以及向其写入数据。

### 关联容器
| 按关键字有序保存元素 |                                   |
| -------------------- | --------------------------------- |
| map                  | 关联数组：保存关键字-值对         |
| set                  | 关键字即值，即只保存关键字的容器  |
| multimap             | 关键字可重复出现的map             |
| multiset             | 关键字可重复出现的set             |
| 无序集合             |                                   |
| unordered_map        | 用哈希函数组织的map               |
| unordered_set        | 用哈希函数组织的set               |
| unordered_multimap   | 哈希组织的map：关键字可以重复出现 |
| unordered_multiset   | 哈希组织的set：关键字可以重复出现 |

如果一个类型定义了“行为正常”的<运算符，则它可以用作关键字类型。
**pair类型的操作**
| pair<T1, T2> p;                                        | p是一个pair，两个类型分别为T1和T2的成员都进行了值初始化      |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| pair<T1, T2> p(v1, v2)<br />pair<T1, T2> p = (v1, v2); | p是一个成员类型为T1和T2的pair；first和second成员分别用v1和v2进行初始化 |
| make_pair(v1, v2);                                     | 返回一个用v1和v2初始化的pair。pair的类型从v1和v2的类型推断出来 |
| p.first                                                | 返回p的名为first的（公有）数据成员                           |
| p.second                                               | 返回p的名为second的（公有）数据成员                          |
| p1 relop p2                                            | 关系运算符(<. >, <=, >=)按字典序定义                         |
| p1 == p2<br />p1 != p2                                 | 当first和second成员分别相等时，两个pair相等，相等性判断利用元素的==运算符实现 |
**关联容器额外的类型别名**
| key_type    | 此容器类型的关键字类型                                       |
| ----------- | ------------------------------------------------------------ |
| mapped_type | 每个关键字关联的类型；只适用于map                            |
| value_type  | 对于set，同key_type；对于map，为pair<const key_type, mapped_type> |

**map和unordered_map的下标操作**
| c[k]    | 返回关键字为k的元素；如果k不在c中，添加一个关键字为k的元素，对其进行值初始化 |
| ------- | ------------------------------------------------------------ |
| c.at(k) | 访问关键字为k的元素，带参数检查；如果k不在c中，抛出一个out_of_range异常 |

**无序容器**
默认情况下，无序容器使用关键字类型的==元素符来比较元素，还使用一个hash<key_type>类型的对象来生成每个元素的哈希值。对于自定义类型的关键字，无法定义其无序容器，除非提供其hash模板版本。

### 动态内存
**智能指针**
智能指针与常规指针的区别是它负责自动释放所指的对象。shared_ptr允许多个指针指向同一个对象；unique_ptr则“独占”所指向的对象。标准库还定义了名为weak_ptr的伴随类，它是一种弱引用，指向shared_ptr所管理的对象。这三种类型都定义在memory头文件中。

shared_ptr独有的操作
| make_shared<T>(args) | 返回一个shared_ptr，指向一个动态分配的类型为T的对象，使用args初始化此对象 |
| -------------------- | ------------------------------------------------------------ |
| shared_ptr<T> p(q)   | p 是shared_ptr q的拷贝；此操作会递增q中的计数器。q中的指针必须能转换为T* |
| p = q                | p 和q 都是shared_ptr，所保存的指针必须能互相转换。此操作会递减p的引用技术，递增q的引用计数；若p的引用计数变为0，则将其管理的原内存释放 |
| p.unique()           | 若p.use_count()为1，返回true，否则返回false                  |
| p.use_count()        | 返回与p 共享对象的智能指针数量；可能很慢，主要用于调试       |

**直接管理内存**
```cpp
int *pi1 = new int; //默认初始化，*pi1的值未定义
int *pi2 = new int();//值初始化为0

//默认情况下，如果new不能分配所要求的内存空间，会抛出类型为bad_alloc的异常。可以通过改变使用new的方式来阻止异常抛出。
int *p2 = new (nothrow) int;//如果分配失败，new返回一个空指针
```

定义和改变shared_ptr的其他方法
| shared_ptr<T> p(q)                           | p管理内置指针q所指向的对象；q必须指向new分配的内存，且能够转换为T*类型 |
| -------------------------------------------- | ------------------------------------------------------------ |
| shared_ptr<T> p(u)                           | p从unique_ptr u那里接管了对象的所有权；将u置空               |
| shared_ptr<T> p(q, d)                        | p接管了内置指针q所指向的对象的所有权。q必须能转换为T*类型。p将使用可调用对象d来代替delete |
| shared_ptr<T> p(p2, d)                       | p时shared_ptr p2的拷贝，唯一的区别是p将用可调用对象d来代替delete |
| p.reset()<br />p.reset(q)<br />p.reset(q, d) | 若p是唯一指向其对象的shared_ptr，reset会释放此对象。若传递了可选的参数内置指针q，会令p指向q，否则会将p置空。若还传递了参数d，将会调用d而不是delete来释放q |

**unique_ptr**
| unique_ptr<T> u1<br />unique_ptr<T, D> u2 | 空unique_ptr，可以指向类型为T的对象。u1会使用delete来释放它的指针；u2会使用一个类型为D的可调用对象来释放它的指针 |
| ----------------------------------------- | ------------------------------------------------------------ |
| unique_ptr<T, D> u(d)                     | 空unique_ptr，指向类型为T的对象，用类型为D的对象d代替delete  |
| u = nullptr                               | 释放u指向的对象，将u置为空                                   |
| u.release()                               | u放弃对指针的控制权，返回指针，并将u置空                     |
| u.reset()                                 | 释放u指向的对象                                              |
| u.reset(q)<br />u.reset(nullptr)          | 如果提供了内置指针q，令u指向这个对象，否则将u置空            |

**weak_ptr**
weak_ptr是一种不控制所指向对象生存期的智能指针，它指向由一个shared_ptr所管理的对象。
| weak_ptr<T> w     | 空weak_ptr可以指向类型为T的对象                              |
| ----------------- | ------------------------------------------------------------ |
| weak_ptr<T> w(sp) | 与shared_ptr sp指向相同对象的weak_ptr。T必须能转换为sp指向的类型 |
| w = p             | p可以是一个shared_ptr或一个weak_ptr。赋值后w与p共享对象      |
| w.reset()         | 将w置空                                                      |
| w.use_count()     | 与w共享对象的shared_ptr的数量                                |
| w.expired()       | 若w.use_count()为0，返回true，否则返回false                  |
| w.lock()          | 如果expired为true，返回空shared_ptr，否则返回指向w对象的shared_ptr |

**动态数组**
动态数组并不是数组类型
```cpp
typedef int arrT[42];//arrT表示42个int的数组类型
int* p = new arrT;//分配一个42个int的数组，同int* p = new int[42];

//动态分配一个空数组是合法的
char arr[0];//错误，不能定义长度为0的数组
char* cp = new char[0];//正确，但cp不能解引用

delete [] pa;//pa必须指向一个动态分配的数组或为空
```

**allocator类**
allocator类将内存分配和对象构造分离。
| allocator<T> a       | 定义了一个名为a的allocator对象，可以为类型为T的对象分配内存  |
| -------------------- | ------------------------------------------------------------ |
| a.allocate(n)        | 分配一段原始的，未构造的内存，保存n个类型为T的对象           |
| a.deallocate(p, n)   | 释放从T*指针p中地址开始的内存，这块内存保存了n个类型为T的对象；p必须是先前由allocate返回的指针，且n必须是p创建时所要求的大小。在调用deallocate之前，用户必须对每个在这块内存中创建的对象调用destroy |
| c.construct(p, args) | p必须是一个类型为T*的指针，指向一块原始内存；arg被传递给类型为T的构造函数，用来在p指向的内存中构造一个对象 |
| a.destroy( p)         | p为T*类型的指针，此算法对p指向的对象执行析构函数             |

allocator算法
| uninitialized_copy(b, e, b2)   | 从迭代器b和e指出的输入范围中拷贝元素到迭代器b2指定的未构造的原始内存中。b2指向的内存必须足够大，能容纳输入序列中元素的拷贝 |
| ------------------------------ | ------------------------------------------------------------ |
| uninitialized_copy_n(b, n, b2) | 从迭代器b指向的元素开始，拷贝n个元素到b2开始的内存中         |
| uninitialized_fill(b, e, t)    | 从迭代器b和e指定的原始内存范围中创建对象，对象的值均为t的拷贝 |
| uninitialized_fill_n(b, n, t)  | 从迭代器b指向的内存地址开始创建n个对象。b必须指向足够大的未构造的原始内存，能够容纳给定数量的对象 |

### 拷贝控制
拷贝构造函数、拷贝赋值运算符、移动构造函数、移动赋值运算符和析构函数。如果一个类没有定义所有这些拷贝控制成员，编译器会自动定义缺失的操作。
**拷贝构造**
构造时触发
```cpp
class Foo {
public:
    Foo();//默认构造函数
    Foo(const Foo&);//拷贝构造函数
}
```
初始化标准库容器或调用其insert或push成员时，容器会对其元素进行拷贝初始化。而emplace成员创建的元素都进行直接初始化。
**拷贝赋值**
赋值时触发
对于某些类，合成拷贝赋值运算符用来禁止该类型对象的赋值。
```cpp
class Fpp {
public:
    Foo& operator=(const Foo&);
}
```
**三/五法则，需要析构函数的类也需要拷贝和赋值操作**
三个基本操作可以控制类的拷贝操作：拷贝构造函数、拷贝赋值运算符、析构函数。新标准下，类还可以定义移动构造函数和移动赋值运算符。
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
**阻止拷贝**
```cpp
struct NoCopy {
    NoCopy() = default;
    NoCopy(const NoCopy&) = delete; // 阻止拷贝
    NoCopy &operator=(const NoCopy&) = delete;//阻止赋值
    ~NoCopy() = default;
};
```
本质上，当不可能拷贝、赋值或销毁类的成员时，类的合成拷贝控制成员就被定义为删除的.
在新标准发布之前，类是通过将其拷贝构造函数和拷贝赋值运算符声明为private来阻止拷贝。
**拷贝控制和资源管理**
```cpp
class HasPtr {
public:
    HasPtr(const string &s = string()):
        ps(new string(s)), i(0) {};
    HasPtr(const HasPtr &p):
        ps(new string(*p.ps)), i(p.i) {};
    //如果将一个对象赋予它自身，赋值运算符必须能正确工作
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
**交换操作**
```cpp
//自定义swap，swap函数应该调用swap，而不是std::swap。如果存在类型特定的swap版本，swap调用会与之匹配。如果不存在类型特定的版本，则会使用std中的版本（假定作用域中有using声明）
class HasPtr {
    firend void swap(HasPtr&, HasPtr&);
}
inline
void swap(HasPtr &lhs, HasPtr &rhs) {
    using std::swap;
    swap(lhs.ps, rhs.ps);
    swap(lhs.i, rhs.i);
}

//拷贝并交换。这个技术自动处理了自赋值情况且天然就是安全的。
    HasPtr& operator=(HasPtr rhs) {
        swap(*this, rhs); //rhs现在指向本对象曾经使用的内存
        return *this; //rhs被销毁，delete了rhs中的指针
    }
```
**右值引用**
右值引用，就是必须绑定到右值的引用。通过&&来获取右值引用。右值引用的一个重要性质——只能绑定到一个将要销毁的对象。
变量是左值
```cpp
int &&rr1 = 42;
int &&rr2 = rr1;//错误，rr1是左值
```
**标准库move函数**
新标准库函数move，可以获取绑定到左值上的右值引用。
```cpp
int &&rr3 = std::move(rr1);//ok
```
调用move就意味着承诺：除了对rr1赋值或销毁它外，将不再使用它。在调用move后，不能对移后源对象的值做任何假设。
**移动操作**
移动构造函数的第一个参数是该类类型的右值引用。除了完成资源移动，移动构造函数还必须确保移后源对象处于销毁是无害的状态。特别是，一旦资源完成移动，源对象必须不再指向被移动的资源。
```cpp
StrVec::StrVec(StrVec &&s) noexcept : //移动操作不应抛出任何异常
    elements(s.elements), first_free(s.first_free), cap(s.cap) {
    //对s运行析构函数是安全的
    s.elements = s.first_free = s.cap = nullptr;
}

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
移后源对象必须可析构
只有当一个类没有定义任何自己版本的拷贝控制成员 ，且类的每个非static数据成员都可以移动时，编译器才会为它合成移动构造函数和移动赋值运算符。
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
> 关于三/五法则。所有五个拷贝控制成员应该看作一个整体：一般来说，如果一个类定义了任何一个拷贝操作，它就应该定义所有五个操作。

**移动迭代器**
通过调用标准库的make_move_iterator函数将普通迭代器转换为移动迭代器
**右值引用和成员函数**
强制左侧运算符对象(即this指向的对象)是一个左值。指出this的左值/右值属性的方式和定义const成员函数相同，在参数列表后放置一个引用限定符
```cpp
class Foo {
public:
    Foo &operator=(const Foo&) &;//只能向可修改的左值赋值
};
```
**重载运算符**
```cpp
//重载运算符本质上是对函数的调用
data1 + data2;//普通表达式
operator+(data1, data2);//等价的函数调用
data1.operator+=(data2);//同上
```
当读取操作发生错误时，输入运算符应该负责从错误中恢复
```cpp
//如果类同时定义了算术运算符和相关的复合赋值运算符，通常情况下应该使用复合赋值来实现算数运算符
//要考虑两个引用为同一个对象的情况
Sales_data
operator+(const Sales_data& lhs, const Sales_data& rhs) {
    Sales_data sum = lhs;
    sum += rhs;
    return sum;
}
```
区分前置和后置运算符。后置版本接受一个额外的（不被使用）int类型的形参。使用后置运算符时，编译器为这个形参提供一个值为0的实参。从语法上来说后置函数可以使用这个额外的形参，但实际上不会这么做。该形参的唯一作用就是区分前置版本和后置版本的函数。
```cpp
    StrBlobPtr& operator++() {
        check(curr, "increment past end of StrBlobPtr");//检查是否越界
        ++curr;
        return *this;
    }
    StrBlobPtr& operator++(int) {
        StrBlobPtr ret = *this;
        ++*this;
        return ret;
    }

//显式调用后置运算符
StrBlobPtr p(al);
p.operator++(0); //后置版本
p.operator++(); //前置版本
```

**成员访问运算符**
```cpp
    string& operator*() const {
        auto p = check(curr, "dereference past end");
        return (*p)[curr]; // *p 为对象所指的vector
    }
    string* operator->() const {
        return & this->operator*();
    }
```
operator*可以完成任何指定的操作，比如返回一个固定值。但是箭头运算符永远不能丢掉成员访问这个最基本的含义。
point->mem的执行过程如下
- 如果point是指针，则应用内置的箭头运算符，表达式等价于(*point).mem
- 如果point是定义了operator->的类的一个对象，则使用point.operator->()的结果来获取mem。其中，如果该结果是一个指针，则执行第一步；如果该结果本身含有重载的operator->()，则重复调用当前步骤。最终，当这一过程结束时程序或者返回了所需的内容，或者返回一些表示程序错误的信息。

**函数调用运算符**
如果类定义了调用运算符，则该类的对象称作函数对象。函数对象常常作为泛型算法的实参。
lambda是函数对象。编写一个lambda后，编译器将表达式翻译成一个未命名类的未命名对象。在lambda表达式产生的类中含有一个重载的函数调用运算符。
```cpp
[sz](const string &a) {return a.size() > sz;}
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
**在算法中使用标准库函数对象**
我只是想说标准库也有定义函数对象
```cpp
// 降序排列
sort(svec.begin(), svec.end(), greater<string>());

vector<string*> nameTable;
//标准库规定其函数对象对于指针同样适用
sort(nameTable.begin(), nameTable.end(), less<string*>());
```
**标准库function类型**
```cpp
function<int(int, int)> f1 = add;
function<int(int, int)> f2 = divide();
function<int(int, int)> f3 = [](int i, int j) {return i * j;};
```
**类型转换运算符**
一个类型转换函数必须是类的成员函数；不能声明返回类型，形参列表也必须为空。类型转换函数通常不改变对象的内容，为const
```cpp
//构造函数将算术类型的值转换成SmallInt对象，而类型转换运算符将SmallInt转换成int
//事实上这个样例在参与算术运算时会产生二义性
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
值得注意的是，当表达式在以下位置时，显式的类型转换将被隐式执行
- if、while及do语句的条件部分
- for 语句头的条件表达式
- 逻辑运算符的运算对象
- 条件运算符的条件表达式（？：）

一些运算符的重载需要为非成员函数，例如输入输出运算符。因为成员函数的左侧运算对象必须为类对象。+，-等运算符为非成员函数时，还能执行类型转换规则。
**避免有二义性的类型转换**
- 不要令两个类执行相同的类型转换，即A接受B类对象的构造函数，B类定义转换目标为A的转换运算符
- 避免转换目标是内置算术类型的类型转换，除了显式转换为bool

调用重载函数时，不会考虑标准类型转换级别，会认为这些类型转换都一样好。只有当重载函数能通过同一个类型转换函数得到匹配时，才会考虑其中出现的标准类型转换。