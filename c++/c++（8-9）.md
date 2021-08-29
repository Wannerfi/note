

[TOC]



# Primer c++第五版笔记2(到第9章完)

~~这是笔记，所以别问为什么写的跟书上一摸一样了~~



## 8 IO库

大部分IO库设施

- istream（输入流）类型，提供输入操作
- ostream（输出流）类型，提供输出操作
- cin，一个istream对象，从标准输入读取数据
- cout，一个ostream对象，向标准输出写入数据
- cerr，一个ostream对象，通常用于输出程序错误消息，写入到标准错误
- \>>运算符，用来从一个istream对象读取输入数据。
- \<<运算符，用来向一个ostream对象写入输出数据
- getline函数，从一个给定的istream读取一行数据，存入一个给定的string对象中

### 8.1 IO类

**表8.1 IO库类型和头文件**

| 头文件   | 类型                                                         |
| -------- | ------------------------------------------------------------ |
| iostream | istream，wistream 从流中读取数据<br />ostream, wostream 向流写入数据<br />iostream, wiostream 读写流 |
| fstream  | ifstream，wifstream 从文件读取数据<br />ofstream，wofstream 向文件写入数据<br />fstream，wfstream 读写文件 |
| sstream  | istringstream，wistringstream  从string读取数据<br />ostringstream，wostringstream 向string写入数据<br />stringstream，wstringstream 读写string |

*为了支持宽字符，标准库定义了一组类型和对象来操纵wchar_t类型的数据。宽字符版本的类型和函数的名字以一个w开始。例wcin、wcout和wcerr分别对应cin、cout、cerr的宽字符版本。宽字符版本的类型和对象与其对应的普通char版本的类型定义在同一个头文件中*

**IO类型间的关系**

标准库是我们能忽略这些不同类型的流之间的差异，这是通过**继承机制**实现的。利用模板可以使用具有继承关系的类，而不必了解继承机制如何工作的细节。

类型ifstream和istringstream都继承自istream

#### 8.1.1 IO对象无拷贝或赋值

如7.1.3节所见，不能拷贝或对IO对象赋值，因此也不能将形参或返回类型设置为流类型。进行IO操作的函数通常以引用方式传递和返回流。读写一个IO对象会改变其状态，因此传递和返回的引用不能是const的。

#### 8.1.2 条件状态

**表8.2 条件状态**

|                   |                                                              |
| ----------------- | ------------------------------------------------------------ |
| strm::iostate     | strm 是一种IO类型，在表8.1中已列出。iostate是一种机器相关的类型，提供了表达条件状态的完整功能 |
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

一个流一旦发生错误，其上后续的IO操作都会失败。只有当一个流处于无错状态时，才可以从它读取数据，向它写入数据。

```c++
//代码通常应该在使用一个流之前检查它是否处于良好状态
while(cin >> word)
```

**查询流的状态**

IO库定义了一个与机器无关的iostate类型。这个类型应作为一个位集合来使用。IO库定义了4个iostate类型的constaexpr值，表示特定的位模式。

badbit表示系统级错误，如不可恢复的读写错误。通常情况下，一旦badbit被置位，流就无法再使用了。在发生可恢复错误后，failbit被置位，如读取类型错误。如果到达文件结束位置，eofbit和failbit都会被置位。goodbit的值为0，表示流未发生错误。如果badbit、failbit和eofbit任一个被置位，则检测流状态的条件会失败。

查询这些标志位状态的函数。操作good砸死所有错误均未置位的情况下返回true，而bad、fail、和eof则在对应错误位被置位时返回true。此外，在badbit被置位时，fail也会返回true。这意味着，使用good或fail是确定流的总体状态的正确方法。实际上，我们将流当作条件使用的代码就等价于!fail()。而eof和bad操作只能表示特定的错误。

**管理条件状态**

```c++
//记住cin的当前状态
auto old_state = cin.rdstate();
cin.clear(); //使cin有效
process_input(cin) //使用cin
cin.setstate(old_state); //将cin置为原有状态

//复位failbit和badbit，保持其他标志位不变
cin.clear(cin.rdstate() & ~cin.failbit & ~cin.badbit);
```

#### 8.1.3 管理输出缓冲

导致缓冲刷新的原因

- 程序正常结束，作为main函数的return操作的一部分，缓冲刷新被执行
- 缓冲区满时，需要刷新缓冲，而后新的数据才能继续写入缓冲区
- 使用操纵符如 endl 来显式刷新缓冲区
- 在每个输出操作后，用操纵符 unitbuf 设置流的内部状态，来清空缓冲区。默认情况下，cerr是设置unitbuf的，因此写到cerr的内容都是立即刷新的
- 一个输出流可能被关联到另一个流。在这种情况下，当读写被关联的流时，关联到的流的缓冲区会被刷新。例如默认情况下，cin和cerr都关联到cout。因此读cin或写cerr都会导致cout的缓冲区被刷新。

```c++
//刷新输出缓冲区
cout << "hi!" << endl;//换行
cout << "hi!" << flush;//不附加
cout << "hi!" << ends;//空字符
```

**unitbuf** 操纵符

unitbuf 操纵符告诉流在接下来的每次写操作后都进行一次flush操作。nounitbuf 操纵符则重置流，使其恢复使用正常的系统管理的缓冲区刷新机制。

```c++
cout << unitbuf;
cout << nounitbuf;
```

>**警告** 如果程序崩溃，输出缓冲区不会被刷新

**关联输入和输出流**

标准库将cout和cin关联在一起

```c++
cin >> ival;//导致cout的缓冲区被刷新
```

> 交互式系统通常应该关联输入流和输出流。这意味着所有输出，包括用户提醒信息，都会在读操作之前被打印出来

```c++
/** 
       *  @brief  Ties this stream to an output stream.
       *  @param  __tiestr  The output stream.
       *  @return  The previously tied output stream, or NULL if the stream
       *           was not tied.
*/
cin.tie(&cout); //标准库将cin和cout关联在一起
//old_tie 指向当前关联的cin的流（如果有的话）
ostream *old_tie = cin.tie(nullptr);//cin 不再与其他流关联
//将cin与cerr关联(不推荐)
cin.tie(&cerr);//读取cin会刷新cerr而不是cout
cin.tie(old_tie); //重建cin与cout之间的正常关联
```

每个流可以同时最多关联到一个流，但多个流可以同时关联到同一个ostream

### 8.2 文件输入输出

**表8.3 fstream 特有的操作**

|                         |                                                              |
| ----------------------- | ------------------------------------------------------------ |
| fstream fstrm;          | 创建一个未绑定的文件流。fstream是头文件fstream中定义的一个类型 |
| fstream fstrm(s);       | 创建一个 fstream，并打开名为 s 的文件。文件 s 可以是 string 类型，或者是一个指向C风格字符串的指针。这些构造函数都是explicit 的。默认的文件模式 mode 依赖于 fstream 的类型 |
| fstream fstrm(s, mode); | 与前一个构造函数类似，但按指定mode打开文件                   |
| fstrm.open(s)           | 打开名为 s 的文件，并将文件与 fstrm 绑定。s 可以是一个 string 或一个指向 C 风格字符串的指针。默认的文件mode 依赖于 fstream 的类型，返回void |
| fstrm.close()           | 关闭与fstrm 绑定的文件。返回void                             |
| fstrm.is_open()         | 返回一个bool 值，指出与fstrm 关联的文件是否成功打开且尚未关闭 |

#### 8.2.1 使用文件流对象

```c++
ifstream in(ifile); // 构造一个ifstream并打开给定文件
ofstream out; // 输出文件流未关联到任何文件
```

**成员函数open和close**

一旦一个文件流已经打开，它就保持与对应文件的关联。实际上，对一个已经打开的文件流调用open会失败，并会导致failbit被置位。

**自动构造和析构**

> 当一个fstream 对象被销毁时，close会自动被调用

#### 8.2.2 文件模式

**表8.4 文件模式**

|        |                              |
| ------ | ---------------------------- |
| in     | 以读方式打开                 |
| out    | 以写方式打开                 |
| app    | 每次写操作前均定位到文件末尾 |
| ate    | 打开文件后立即定位到文件末尾 |
| trunc  | 截断文件                     |
| binary | 以二进制方式进行IO           |

指定文件模式有如下限制：

- 只可以对 ofstream 或fstream 对象设定 out模式
- 只可以对 ifstream 或 fstream 对象设定 in 模式
- 只有当 out 也被设定时才可设定 trunc 模式
- 只要 trunc 没被设定，就可以设定 app 模式。在 app 模式下，即使没有显式指定 out 模式，文件也总是以输出方式被打开
- 默认情况下，即使我们没有指定 trunc，以 out 模式打开的文件也会被截断。为了保留以 out 模式打开的文件的内容，我们必须同时设定 app 模式，这样只会将数据追加写到文件末尾；或者同时指定 in 模式，即打开文件同时进行读写操作
- ate 和 binary 模式可用于任何类型的文件流对象，且可以与其他任何文件模式组合使用

每个文件流类型都定义了一个默认的文件模式，当我们未指定文件模式时，就使用此默认模式。与ifstream关联的文件默认以 in 模式打开；与 ofstream 关联的文件默认以 out 模式打开；与 fstream 关联的文件默认以 in 和 out 模式打开

**以out模式打开文件会丢弃已有数据**

默认情况下，当我们打开一个ofstream 时，文件的内容会被丢弃。阻止一个 ofstream 清空给定文件内容的方法是同时指定 app 模式：

```c++
//在这几条语句，file1 都会被截断
ofstream out("file1"); //隐含以输出模式打开文件并截断文件
ofstream out2("file1", ofstream::out); //隐含地截断文件
ofstream out3("file1", ofstream::out | ofstream::trunc);
//为了保存文件内容，必须显式指定 app 模式
ofstream app("file2", ofstream::app); // 隐含为输出模式
ofstream app2("file2", ofstream::out | ofstream::app);
```

*保留被 ofstream 打开的文件中已有数据的唯一方式是显式指定 app 或 in 模式*

**每次调用 open 时都会确定文件模式**

```c++
ofstream out;
out.open("scratchpad"); //模式隐含设置为输出和截断
out.close(); // 关闭out，以便我们将其用于其他文件
out.open("precious", ofstream::app); // 模式为输出和追加
out.close();
```

### 8.3 string流

sstreawm 头文件定义了三个类型来支持内存IO。`istringstream` 从string读取数据，`ostringstream` 向string写入数据，头文件`stringstream` 向string读写数据。

**表8.5 stringstream 特有的操作**

|                  |                                                              |
| ---------------- | ------------------------------------------------------------ |
| sstream strm;    | strm 是一个未绑定的stringstream对象。sstream 是头文件sstream中定义的一个类型 |
| sstream strm(s); | strm 是一个 sstream 对象，保存strings的一个拷贝。此构造函数是explicit的 |
| strm.str()       | 返回strm所保存的string 的拷贝                                |
| strm.str(s)      | 将string s 拷贝到strm中。返回void                            |

#### 8.3.1 使用istringstream

```c++
/*
 * input
 * moas 356546513 654165
 * dsauh 56465 
 * iew 54541 56456 32157
 */

struct PersonInfo {
    string name;
    vector<string> phones;
};

int main() {
    string line, word; 
    vector<PersonInfo> people; //保存输入的所有记录
    while(getline(cin, line)) {
        PersonInfo info;
        istringstream record(line);
        record >> info.name;
        while(record >> word)
            info.phones.push_back(word);
        people.push_back(info);
    }
    return 0;
}
```

#### 8.3.2 使用ostringstream

```c++
    for(const auto& entry: people){
        ostringstream formatted, badNums;
        for(const auto& nums: entry.phones) {
            if(!valid(nums))
                badNums << " " << nums;
            else 
                formatted << " " << format(nums);
        }
        if(badNums.str().empty())
            os << entry.name << " " << formatted.str() << endl;
        else 
            cerr << "input error: " << entry.name
                << " invalid number(s) " << badNums.str() << endl;
    }
```



## 9 顺序容器

**表9.1 顺序容器类型**

|              |                                                              |
| ------------ | ------------------------------------------------------------ |
| vector       | 可变大小数组。支持快速随机访问。在尾部之外的位置插入或删除元素可能很慢 |
| deque        | 双端队列。支持快速随机访问。在头尾位置插入/删除速度都快      |
| list         | 双向链表。只支持双向顺序访问。在list中任何位置进行插入/删除操作速度都很快 |
| forward_list | 单向链表。只支持单向顺序访问。在链表任何位置进行插入/删除操作速度都很快 |
| array        | 固定大小数组。支持快速随机访问。不能添加或删除元素           |
| string       | 与vector相似的容器，但专门用于保存字符。随机访问快。在尾部插入/删除速度快 |

deque支持快速的随机访问。在deque的中间位置添加或删除元素的代价（可能）很高，但是在deque的两端添加或删除元素都是很快的。

*forward_list的设计目的是达到与最好的手写的单向链表数据结构相当的性能。因此，forward_list没有size操作。对其他容易而言，size保证是一个快速的常量时间的操作*

> 新标准库的容器比旧版本快得多。现代c++程序应该使用标准库容器，而不是更原始的数据结构，如内置数组

### 9.2 容器库概览

**表9.2 容器操作**

| 类型别名        |                                                        |
| --------------- | ------------------------------------------------------ |
| iterator        | 此容器类型的迭代器类型                                 |
| const_iterator  | 可以读取元素，但不能修改元素的迭代器类型               |
| size_type       | 无符号整数类型，足够保存此种容器类型最大可能容器的大小 |
| difference_type | 带符号整数类型，足够保存两个迭代器之间的距离           |
| value_type      | 元素类型                                               |
| reference       | 元素的左值类型：与value_type&含义相同                  |
| const_reference | 元素的const左值类型（即 const velue_type&）            |

| 构造函数         |                                                             |
| ---------------- | ----------------------------------------------------------- |
| C c;             | 默认构造函数，构造空容器（array）                           |
| C c1(c2);        | 构造c2的拷贝c1                                              |
| C c(b, e);       | 构造c，将迭代器b和e指定的范围内的元素拷贝到c（array不支持） |
| C c{a, b, c...}; | 列表初始化c                                                 |

| 赋值与swap        |                                               |
| ----------------- | --------------------------------------------- |
| c1 = c2           | 将c1中的元素替换为c2中元素                    |
| c1 = {a, b, c...} | 将c1中的元素替换为列表中元素（不适用于array） |
| a.swap(b)         | 交换a和b的元素                                |
| swap(a, b)        | 同上                                          |

| 大小         |                                          |
| ------------ | ---------------------------------------- |
| s.size()     | c中元素的数目（不支持forward_list）      |
| c.max_size() | c中可保存的最大元素数目                  |
| c.empty()    | 若c中保存了元素，返回false，否则返回true |

| 添加/删除元素（不适用于array） | 注：在不同容器中，这些操作的接口都不同 |
| ------------------------------ | -------------------------------------- |
| c.insert(args)                 | 将args中的元素拷贝进c                  |
| c.emplace(inits)               | 使用inits构造c中的一个元素             |
| c.erase(args)                  | 删除args指定的元素                     |
| c.clear                        | 删除c中的所有元素，返回void            |
| 关系运算符                     |                                        |
| ==, !=                         | 所有容器都支持相等（不等）运算符       |
| <. <=, >, >=                   | 关系运算符（无序关联容器不支持）       |

| 获取迭代器                               |                                           |
| ---------------------------------------- | ----------------------------------------- |
| c.begin(), c.end()                       | 返回指向c的首元素和尾元素之后位置的迭代器 |
| c.cbegin(), c.cend()                     | 返回const_iterator                        |
| 反向容器的额外成员（不支持forward_list） |                                           |
| reverse_iterator                         | 按逆序寻址元素的迭代器                    |
| const_reverse_iterator                   | 不能修改元素的逆序迭代器                  |
| c.rbegin(), crend()                      | 返回指向c的尾元素和首元素之前位置的迭代器 |
| c.crbegin(), c.crend()                   | 返回const_reverse_iterator                |

#### 9.2.1 迭代器

forward_list迭代器不支持递减运算符

**迭代器范围**

一个迭代器范围由一对迭代器表示，两个迭代器分别指向同一个容器中的元素或者是尾元素之后的位置。元素范围为`左闭右开`。end可以与begin指向相同的位置，但不能指向begin之前的位置

#### 9.2.2 容器类型成员

通过类型别名，可以在不了解容器中元素类型的情况下使用它。如果需要元素类型，可以使用容易的value_type。如果需要元素类型的一个引用，可以使用reference或const_reference。这些元素相关的类型别名在泛型编程中非常有用。

使用这些类型必须显示使用其类名

```c++
list<string>::iterator iter;
vector<int>::difference_type count;
```

#### 9.2.3 begin和end成员

r 开头的代表反向迭代器，c开头的代表const

不以c开头的函数都是被重载过的。实际上有两个begin成员，一个是const成员，返回容器的const_iterator类型。另一个是非常量成员，返回容器的iterator类型。

#### 9.2.4 容器的定义和初始化

每个容器类型都定义了一个默认构造函数。

**表9.3 容器定义和初始化**

|                                             |                                                              |
| ------------------------------------------- | ------------------------------------------------------------ |
| C c;                                        | 默认构造函数。如果C是一个array，则c中元素按默认方式初化；否则c为空 |
| C c1(c2)<br />C c1 = c2                     | c1初始化为c2的拷贝。c1 和 c2 必须是相同类型（相同的容器类型，保存相同的元素类型；对于array，两个具有相同大小） |
| C c{a, b, c, ...}<br />C c = {a, b, c, ...} | c初始化为初始化列表中的拷贝。列表中元素的类型必须与C的元素类型相容。对于array类型，列表中元素数目必须等于或小于array的大小。遗漏的元素都进行值初始化 |
| C c(b, e)                                   | c初始化为迭代器b 和 e 指定范围中的元素的拷贝。范围中元素的类型必须与C 的元素类型相容（array 不适用） |
|                                             | 只有顺序容器（不包括array）的构造函数才能接受大小的参数      |
| C seq(n)                                    | seq包含n个元素，这些元素进行值初始化；此构造函数是explicit。（string 不适用） |
| C seq(n, t)                                 | seq 包含 n 个初始化为值 t 的元素                             |

**将一个容器初始化为另一个容器的拷贝**

将一个新容器创建为另一个容器的拷贝的方法有两种：拷贝整个容器，拷贝有一个迭代器对指定的元素范围（array除外）

当传递迭代器参数来拷贝一个范围时，不要求容器类型是相同的，元素类型也可以不同，只能将要拷贝的元素转换。

```c++
list<string> authors ={"dua", "ius", "rhw"};
vector<const char*> articles = {"a", "an", "the"};

list<string> list2(authors) // 正确，类型匹配
deque<string> authList(authors); // 错误，容器类型不匹配
vector<string> words(articles); // 错误，容器类型必须匹配
// 正确，可以将const char* 元素转换为string
forward_list<string> words(articles.begin(), articles.end());
```

**可以列表初始化**

**与顺序容器大小相关的构造函数**

```c++
vector<int> ivec(10, -1); // 10个int元素，每个都初始化为-1
list<string> svec(10, "hi!"); // 10个strings，每个都初始化为"hi!"
forward_list<int> ivec(10); // 10个元素，每个都初始化为0
deque<string> svec(10); // 10个元素，每个都是空string
```

> 只有顺序容器的构造函数蔡接受大小参数，关联容器并不支持

**标准库array具有固定大小**

```c++
//定义一个array时，要制定元素类型和容器大小
array<int, 42> //类型为保存42个int的数组
//使用array类型时，必须同时指定元素类型和大小
array<int, 10>::size_type i;

//初始化
array<int, 3> ia1; // 3个默认初始化的int
array<int, 3> ia2 = {0, 1, 2}; // 列表初始化
array<int, 3> ia3 = {42}; // ia3[0]为42，剩余元素为0

//array支持拷贝和赋值
array<int, 3> digits = {0, 1, 2};
array<int, 3> copy = digits;
```

#### 9.2.5 赋值和swap

由于右边运算对象的大小可能与左边运算对象的大小不同，因此array类型不支持assign，也不允许用花括号包围的值列表进行赋值。

**表9.4 容器赋值运算**

|                               |                                                              |
| ----------------------------- | ------------------------------------------------------------ |
| c1 = c2                       | 将c1中的元素替换为c2中元素的拷贝。c1和c2必须具有相同的类型   |
| c = {a, b, c...}              | 将c1中元素替换为初始化列表中元素的拷贝（array不适用）        |
| swap(c1, c2)<br />c1.swap(c2) | 交换c1和c2中的元素。c1和c2必须具有相同的类型。swap通常比从c2向c1拷贝元素要快得多 |
|                               | assign操作不适用于关联容器和array                            |
| seq.assign(b, e)              | 将seq中的元素替换为迭代器b和e所表示的范围中的元素。迭代器b和e不能指向seq中的元素 |
| seq.assign(i1)                | 将seq中的元素替换为初始化列表i1中的元素                      |
| seq.assign(n, t)              | 将seq中的元素替换为n个值为t 的元素                           |

> 赋值相关运算会导致指向左边容器内部的迭代器、引用和指针失效。而swap操作将容器内容交换不会导致指向容器的迭代器、引用和指针失效（容器类型为array和stirng的情况除外）

**使用assign（仅顺序容器）**

顺序容器（array除外）定义了一个名为assign的成员，允许我们从一个不同但相容的类型赋值，或者从容器的一个子序列赋值。

```c++
//assign的参数决定了容器中将有多少个元素以及它们的值是什么
list<string> names;
vector<const char*> oldstyle;
names = oldstyle; // 错误，容器类型不匹配
//正确：可以将const char* 转换为string
names.assign(oldstyle.cbegin(), oldstyle.cend());
```

> 由于旧元素被替换，因此传递给assign的迭代器不能指向调用assign的容器

```c++
//assign的第二个版本。
//等价于slist1.clear()
//后跟slist1.insert(slist1.begin(), 10, "Hiya!")
list<string> slist1(1); //1个元素，空string
slist1.assign(10, "Hiya!"); //10个元素，都是"HiYa!"
```

**使用swap**

除array外，交换两个容器内容的操作保证会很快——元素本身并未交换，swap只是交换了两个容器的内部数据结构

> 除array外，swap不对任何元素进行拷贝、删除或插入操作，因此可以保证在常数时间内完成

元素不会移动的事实意味着，除string外，指向容器的迭代器、引用和指针在swap操作之后都不会失效。它们仍指向swap操作之前所指向的那些元素。但是在swap之后，这些元素已经属于不同的容器了。例如，假定iter在swap之前指向svec1[3] 的string，那么在swap之后它指向svec2[3] 的元素。与其他容器不同，对一个string调用swap会导致迭代器、引用和指针失效。

与其他容器不同，swap两个array会真正交换它们的元素。因此，交换两个array所需的时间与array中元素的数目成正比。因此，对于array，在swap操作之后，指针、引用和迭代器所绑定的元素保持不变，但元素值已经与另一个array中对应元素的值进行了交换。

新标准库中提供了成员函数版本和非成员版本的swap。而早期标准库版本只提供成员函数版本的swap，非成员版本的swap在泛型编程中是非常重要的。统一使用非成员版本的swap是一个好习惯。

#### 9.2.6 容器大小操作

max_size返回一个大于或等于该类型容器所能容纳的最大元素数的值。

forward_list支持max_size和empty，但不支持size

#### 9.2.7 关系运算符

比较两个容器实际上是进行元素的主队比较。这些运算符的工作方式与string的关系运算符类似。

> 只有当其元素类型也定义了相应的比较运算符时，我们才可以使用关系运算符来比较容器

### 9.3 顺序容器操作

**表9.5 向顺序容器添加元素的操作**

|                                              | 这些操作会改变容器的大小；array不支持这些操作                |
| -------------------------------------------- | ------------------------------------------------------------ |
|                                              | forward_list有自己专有版本的insert 和 emplace                |
|                                              | forward_list不支持push_back和emplace_back                    |
|                                              | vector 和 string 不支持 push_front 和 emplace_front          |
| c.push_back(t)<br />c.emplace_back(args)     | 在c 的尾部创建一个值为t 或由args 创建的元素。返回void        |
| c.push_front(t)<br />c.emplace_front(args)   | 在c 的头部创建一个值为t 或由args 创建的元素。返回void        |
| c.insert(p, t)<br />c.emplace_front(p, args) | 在迭代器p指向的元素之前创建一个值为t 或由args 创建的元素。返回指向新添加的元素的迭代器 |
| c.insert(p, n, t)                            | 在迭代器p指向的元素之前插入n 个值为t 的元素。返回指向新添加的元素的迭代器；若n 为0，则返回p |
| c.insert(p, b, e)                            | 将迭代器b 和 e 指定的范围内的元素插入到迭代器p 指向的元素之前。b 和e 不能指向c 中的元素。返回指向新添加的第一个元素的迭代器；若范围为空，则返回p |
| c.insert(p, i1)                              | i1 是一个花括号包围的元素值列表。将这些给定值插入到迭代器p 指向的元素之前。返回指向新添加的第一个元素的迭代器；若列表为空，则返回p |

> 向一个vector、string或deque 插入元素会使所有指向容器的迭代器、引用和指针失效

除array 和 froward_list之外，每个顺序容器（包括stirng 类型）都支持push_back

> 当用一个对象来初始化容器时，或将一个对象插入到容器中时，实际上放入到容器中的使对象值的一个拷贝，而不是对象本身。

```c++
//insert函数将元素插入到迭代器所指定的位置之前
slist.insert(iter, "Hello!");
//虽然某些容器不支持push_front操作，但是对于insert操作并无类似的限制
vector<string> svec;
list<string> slist;
slist.insert(slist.begin(), "Hello!"); // slist.push_front("Hello!");
svec.insert(svec.begin(), "Hello!");

svec.insert(svec.end(), 10, "Anna"); //将10个元素插入到svec的末尾，并初始化为string"Anna"

vector<string> v = {"a", "b", "c", "d"};
//将v的最后两个元素添加到slist的开始位置
slist.insert(slist.begin(), v.end() - 2, v.end());
slist.insert(slist.end(), {"e", "f", "g"});
//运行时错误；迭代器表示要拷贝的范围，不能指向与目的位置相同的容器
slist.insert(slist.begin(), slist.begin(), slist.end());
```

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

其中对emplace_back的调用和第二个push_back调用都会创建新Sales_data对象。在调用 emplace_back 时，会在容器管理的内存空间中直接创建对象。而调用 push_back 则会创建一个局部临时对象，将其压入容器中。

```c++
//emplace函数的参数根据元素类型而变化，参数必须与元素类型的构造函数相匹配
//iter 指向c 中一个元素，其中保存了Sales_data元素
c.emplace_back(); //使用Sales_data的默认构造函数
c.emplace(iter, "6545"); //使用Sales_data(stirng)
c.emplace_front("4646", 6, 6.6); //使用Sales_data(string, int, double);
```

#### 9.3.2 访问元素

```c++
//包括array在内的每个顺序容器都有一个front成员函数，除forward_list之外的所有顺序容器都有一个back成员
if(!c.empty()) {
    auto val = *c.begin(), val2 = c.front();//获取首元素值
    //获取尾元素值
    auto last = c.end();auto val3 = *(--last);//forward_list迭代器不能递减
    auto val4 = c.back();//forward_list不支持
}
```

**表9.6 在顺序容器中访问元素的操作**

|           | at 和下标操作只适用于string、vector、deque和array            |
| --------- | ------------------------------------------------------------ |
|           | back不适用于forward_list                                     |
| c.back()  | 返回c 中尾元素的引用。若c 为空，函数行为未定义               |
| c.front() | 返回c 中首元素的引用。若c 为空，函数行为未定义               |
| c[n]      | 返回c 中下标为n 的元素的引用，n是一个无符号整数。若n >= c.size()，则函数行为未定义 |
| c.at(n)   | 返回下标为n 的元素的引用。如果下标越界，则抛出out_of_range异常 |

#### 9.3.3 删除元素

**表9.7 顺序容器的删除操作**

|                | 这些操作会改变容器的大小，所以不适用于array                  |
| -------------- | ------------------------------------------------------------ |
|                | forward_list有特殊版本的erase                                |
|                | forward_list 不支持pop_back；vector 和string 不支持pop_front |
| c.pop_back()   | 删除c 中尾元素。若c 为空，则函数行为未定义。函数返回void     |
| c.pop_front()  | 删除c 中首元素。若c 为空，则函数行为未定义。函数返回void     |
| c.erease(p)    | 删除迭代器p 所指定的元素，返回一个指向被删元素之后元素的迭代器 |
| c.erease(b, e) | 删除迭代器b 和e 所指定范围内的元素。返回一个指向最后一个被删除元素之后元素的迭代器，若e 本身就是尾后迭代器，则函数也返回尾后迭代器 |
| c.clear()      | 删除c 中的所有元素。返回void                                 |

> 删除deque中除首尾位置之外的任何元素都会使所有迭代器、引用和指针失效。指向vector 或string 中删除点之后位置的迭代器、引用和指针都会失效

```c++
vector<int> v{1, 1, 1, 2, 2, 3, 4, 5};
for(auto t = v.begin();t != v.end();) {
	if((*t) == 1) {
    	t = v.erase(t);
	} else {
    	++t;
	}
}

//删除多个元素
auto t = v.begin();
while((*t) != 2) ++t;
//删除[v.begin(), t), 返回指向最后一个被删元素之后位置的迭代器
auto p = v.erase(v.begin(), t);
```

#### 9.3.4 特殊的forward_list操作

before_begin，返回首前迭代器

**表9.8 在forward_list中插入或删除元素的操作**

|                                                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| lst.before_begin()<br />lst.cbefore_begin()                  | 返回指向链表首元素之前不存在的元素的迭代器。次迭代器不能解引用。cbefore_begin()返回一个const_iterator |
| lst.insert_after(p, t)<br />lst.insert_after(p, n, t)<br />lst.insert_after(p, b, e)<br />lst.insert_after(p, i1) | 在迭代器p之后的位置插入元素。t是一个对象，n是数量，b和e是表示范围的一对迭代器（b和e不能指向lst内），i1是一个花括号列表。返回一个指向最后一个插入元素的迭代器。如果范围为空，则返回p。若p为尾后迭代器，则函数行为未定义 |
| emplace_after(p, args)                                       | 使用args在p指定的位置之后创建一个元素。返回一个指向这个新元素的迭代器。若p为尾后迭代器，则函数行为未定义 |
| lst.erase_after(p)<br />lst.erase_after(b, e)                | 删除p指向的位置之后的元素，或删除从b之后直到（但不包含）e之间的元素。返回一个指向被删元素之后元素的迭代器。若不存在这样的元素，则返回尾后迭代器，如果p指向lst的尾元素或者是一个尾后迭代器，则函数行为未定义。 |

```c++
    forward_list<int> flst = {0, 1, 2, 3, 4};
    auto prev = flst.before_begin();
    auto curr = flst.begin();
    while(curr != flst.end()) {
        if(*curr % 2)
            curr = flst.erase_after(prev);
        else {
            prev = curr;
            ++curr;
        }
    }
```

 #### 9.3.5 改变容器大小

可以用resize来增大或缩小容器。array不支持resize。如果当前大小大于所要求的大小，容器后部的元素会被删除；如果当前大小小于新大小，会将新元素添加到容器后部

```c++
list<int> ilist(10, 42); // 10个int，每个的值都是42
ilist.size(15); //将5个值为0的元素添加到ilist的末尾
ilist.resize(25, -1); //将10个值为-1的元素添加到ilist的末尾
ilist.resize(5); // 从ilist末尾删除20个元素
```

> 如果resize缩小容器，则指向被删除元素的迭代器、引用和指针都会失效；对vector、string或deque进行resieze可能导致迭代器、指针和引用失效

#### 9.3.6 容器操作可能使迭代器失效

向容器添加元素后

- 如果容器是vector或string，且存储空间被重新分配，则指向容器的迭代器、指针和引用都会失效。如果存储空间未重新分配，指向插入位置之前的元素的迭代器、指针和引用仍有效，但指向插入位置之后元素的迭代器、指针和引用将会失效
- 对于deque，插入到除首尾位置之外的任何位置都会导致迭代器、指针和引用失效。如果在首尾位置添加元素，迭代器会失效，但指向存在的元素的引用和指针不会失效。
- 对于list和froward_list，指向容器的迭代器（包括尾后迭代器和首前迭代器）、指针和引用仍有效

从一个容器中删除元素后，指向被删除元素的迭代器、指针和引用会失效

- 对于list和forward_list，指向容器其他位置的迭代器（包括尾后迭代器和首前迭代器）、引用和指针仍有效。
- 对于deque，如果在首尾之外的任何位置删除元素，那么指向被删除元素外其他元素的迭代器、引用或指针也会失效。如果是删除deque的尾元素，则尾后迭代器也会失效，但其他迭代器、引用和指针不受影响；如果是删除首元素，这些也不会受影响。
- 对于vector和string，指向被删元素之前的元素迭代器、引用和指针仍有效。之一：当我们删除元素时，尾后迭代器总是会失效。

**不要保存end返回的迭代器**

添加/删除vector或string的元素后，或在deque中首元素之外任何位置添加/删除元素后，原来end返回的迭代器总是会失效

### 9.4 vector对象是如何增长的

**表9.10 容器大小管理操作**

|                   | shrink_to_fit只适用于vector、string和deque |
| ----------------- | ------------------------------------------ |
|                   | capacity和reserve只适用于vector和string    |
| c.shrink_to_fit() | 将capacity()减少为与size()相同大小         |
| c.capacity()      | 不重新分配内存空间的话，c可以保存多少元素  |
| c.reserve(n)      | 分配至少能容纳n个元素的内存空间            |

> reserve并不改变容器中元素的数量，它仅影响vector预先分配多大的内存空间

在调用reserver之后，capacity将会大于或等于传递给reserve的参数。调用reserve永远也不会减少容器占用的内存空间。类似的，resize成员函数只改变容器中元素的数目，而不是容器的容量。

在新标准库中，可以调用shrink_to_fit来要求deque、vector或string退回不需要的内存空间。但是它只是一个请求，实际上调用shrink_to_fit也并不保证一定退回内存空间

只有在执行insert操作时size与capacity相等，或者调用resize或reserve时给定的大小超过当前capacity，vector才可能重新分配内存空间。

### 9.5 额外的string操作

#### 9.5.1 构造string的其他方法

**表9.11 构造string 的其他方法**

| n、len2和pos2都是无符号值 |                                                              |
| ------------------------- | ------------------------------------------------------------ |
| string s(cp, n)           | s是cp指向的数组中前n个字符的拷贝。此数组至少应该包含n个字符  |
| string s(s2, pos2)        | s是string s2从下标pos2开始的字符的拷贝。若pos2>s2.size()，构造函数的行为未定义 |
| string s(s2, pos2, len2)  | s是string s2从下标pos2开始len2个字符的拷贝。若pos2>s2.size()，构造函数的行为未定义。不管len2的值是多少，构造函数至多拷贝s2.size()-pos2个字符 |

```c++
    const char* cp = "Hello World!!!";
    char noNull[] = {'H', 'i'}; // 不以'\0'结束
    string s1(cp); //Hello World!!!
    string s2(noNull, 2); //Hi
    string s3(noNull); //未定义行为 Hi
    string s4(cp + 6, 5); //World
    string s5(s1, 6, 5); //World
    string s6(s1, 6); //World!!!
    string s7(s1, 6, 20);//World!!!
    string s8(s1, 16); //out_of_range异常
```

**substr操作**

```c++
    string s("hello world");
    string s2 = s.substr(0, 5);//hello
    string s3 = s.substr(6);//world
    string s4 = s.substr(6, 11);//world
    string s5 = s.substr(12); //out_of_range异常
```

#### 9.5.2 改变string的其他方法

定义了额外的insert和erase版本

```c++
s.insert(s.size(), 5, '!'); //在s末尾插入5个'!'
s.erase(s.size() - 5, 5); //从 s 删除最后5个字符
```

提供接受C风格字符数组的insert和assign版本

```c++
const char* cp = "Stately, plump Buck";
s.assign(cp, 7); //赋予从cp指向的地址开始的7个字符
s.insert(s.size(), cp + 7); //从cp的第7个字符开始拷贝，插入到s[s.size()]位置
s.insert(0, s2); //在s[0]之前插入s2
s.insert(0, s2, 0, s2.size()) //在s[0]之前插入s2[0]到
```

定义了额外成员append和replace

```c++
s.append("a"); //s.push_back("a");
s.replace(0, 1, "b"); // 删除从下标0开始，删除1个字符，并插入"b"
```

#### 9.5.3 string搜索操作

搜索操作返回一个string::size_type的值，表示匹配发生位置的下标。如果搜索失败，返回一个名为string::npos的static成员，初始化为-1，类型为unsigned

```c++
//大小写敏感
string name("AnnaBelle");
auto pos1 = name.find("Anna"); // pos1 = 0

//查找与给定字符串中任何一个字符匹配的位置
string numbers("0123456789"), name("r2d2");
auto pos = name.find_first_of(numbers);
```

**表9.14 string搜索操作**

|                           | 返回指定字符串出现的下标，未找到则返回npos    |
| ------------------------- | --------------------------------------------- |
| s.find(args)              | 查找s中args第一次出现的位置                   |
| s.rfind(args)             | 查找s中args最后一次出现的位置                 |
| s.find_first_of(args)     | 在s中查找args中任何一个字符第一次出现的位置   |
| s.find_last_of(args)      | 在s中查找args中任何一个字符最后一次出现的位置 |
| s.find_first_not_of(args) | 在s中查找第一个不在args中的字符               |
| s.find_last_not_of(args)  | 在s中查找最后一个不在args中的字符             |

|            | args形式                                                     |
| ---------- | ------------------------------------------------------------ |
| c, pos     | 从s中位置pos开始查找字符c。pos默认为0                        |
| s2, pos    | 从s中位置pos开始查找字符串s2。pos默认为0                     |
| cp, pos    | 从s中位置pos开始查找指针cp指向的以空字符结尾的C风格字符串，pos默认为0 |
| cp, pos, n | 从s中位置pos开始查找指针cp指向的数组的前n个字符，pos和n无默认值 |

#### 9.5.4 compare函数

**表9.15 s.compare的几种参数形式**

|                        |                                                              |
| ---------------------- | ------------------------------------------------------------ |
| s2                     | 比较s和s2                                                    |
| pos1, n1, s2           | 将s中从pos1开始的n1个字符与s2进行比较                        |
| pos1, n1, s2, pos2, n2 | 将s中从pos1开始的n1个字符与s2中从pos2开始的n2个字符进行比较  |
| cp                     | 比较s与cp指向的以空字符结尾的字符数组                        |
| pos1, n1, cp           | 将s中从pos1开始的n1个字符与cp指向的以空字符结尾的字符数组进行比较 |
| pos1, n1, cp, n2       | 将s中从pos1开始的n1个字符与指针cp指向的地址开始的n2个字符进行比较 |

#### 9.5.5 数值转换

```c++
int i = 42;
string s = to_string(i);
double d = stod(s);//转换为浮点数

//stod读取字符串中的字符并处理，直到遇到不可能是数值一部分的数字的字符
//转换s中数字开始的第一个子串
string s2 = "pi = 3.14";
d = stod(s2.substr(s2.find_first_of("+-.0123456789")));
//string参数中第一个非空白符必须是符号（+或-）或数字。它可以以0x或0X开头来表示十六进制数。对那些将字符串转换为浮点数的函数，string参数也可以以小数点（.）开头，并可以包含e或E来表示指数部分。对那些将字符串转换为整型值的函数，根据基数不同，string参数可以包含字母字符，对应大于数字9的数
```

> 如果string不能转换为一个数值，抛出invalid_argument异常。如果转换得到的数值无法用任何类型来表示，抛出out_of_range异常

**表9.15 string和数值之间的转换**

|                                                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| to_string(val)                                               | 重载数组，返回数值val的string表示。val可以是任何算术类型。   |
| stoi(s, p, b)<br />stol(s, p, b)<br />stoul(s, p, b)<br />stoll(s, p, b)<br />stoull(s, p, b) | 返回s的起始子串（表示证书内容的）数值，返回值类型分别是int, long, unsigned long, long long, unsigned long long 。b表示转换所用的基数，默认为10.p是size_t指针。用来保存s中第一个非数值字符的下标，p默认值为0，即函数不保存下标 |
| stof(s, p)<br />stod(s, p)<br />stold(s, p)                  | 返回s的起始子串（表示浮点数内容）的数值，返回值类型分别是float，double或long double。参数p的作用与整数转换函数中一样 |

### 9.6 容器适配器

三个顺序容器适配器：stack, queue和priority_queue。适配器是标准库中的一个通用概念。容器、迭代器和函数都有适配器。本质上，一个适配器是一种机制，能使某种事物的行为看起来像另外一种事物一样

**表9.17 所有容器适配器都支持的操作和类型**

|                           |                                                              |
| ------------------------- | ------------------------------------------------------------ |
| size_type                 | 一种类型，足以保存当前类型的最大对象的大小                   |
| value_type                | 元素类型                                                     |
| container_type            | 实现适配器的底层容器类型                                     |
| A a;                      | 创建一个名为a的空适配器                                      |
| A a(c);                   | 创建一个名为a的适配器，带有容器c的一个拷贝                   |
| 关系运算符                | 每个适配器都支持所有关系运算符：==, !=, <, <=, >, >=这些运算符返回底层容器的比较结果 |
| a.empty()                 | 若a包含任何元素，返回false，否则返回true                     |
| a.size()                  | 返回a中元素的数目                                            |
| swap(a, b)<br />a.swap(b) | 交换a和b的内容，a和b必须有相同类型，包括底层容器类型也必须相同 |

**定义一个适配器**

每个适配器都定义两个构造函数：默认构造函数创建一个空对象，接受一个容器的构造函数拷贝该容器来初始化适配器。

```c++
//deq是一个deque<int>，可以用deq来初始化一个新的stack
stack<int> stk(deq);
```

默认情况下，stack和queue是基于deque实现的，priority_queue是在vector之上实现的

```c++
//在创建适配器时将一个命名的顺序容器作为第二类型参数，来重载默认容器类型
//在vector上实现空栈
stack<string, vector<string> > str_stk;
//str_stk2在vector上实现，初始化时保存svec的拷贝
stack<string, vector<string> > str_stk2(svec);
```

所有适配器都要求容器具有添加和删除元素的能力。

stack只要求push_back、pop_back和back操作，因此可以使用除array和forward_list之外的任何容器类型来构造。

queue适配器要求back、push_back、front和push_front，因此可以构造于list或deque之上，但不能基于vector构造。

priority_queue除了front、push_back和pop_back操作之外还要求随机访问能力，因此它可以构造于vector或deque之上，但不能基于list构造

**栈适配器**

**表9.18 表9.17未列出的栈操作**

|                                   | 栈默认时基于deque实现，也可以在list或vector之上实现          |
| --------------------------------- | ------------------------------------------------------------ |
| s.pop()                           | 删除栈顶元素，但不返回该元素值                               |
| s.push(item)<br />s.emplace(args) | 创建一个新元素压入栈顶，该元素通过拷贝或移动item，或者由args构造 |
| s.top()                           | 返回栈顶元素，但不将元素出栈                                 |

**队列适配器**

**表9.19 表9.17未列出的queue和priority_queue操作**

|                                   | queue默认基于deque实现，priority_queue默认基于vector实现     |
| --------------------------------- | ------------------------------------------------------------ |
|                                   | queue也可以用list或vector实现，priority_queue也可以用deque实现 |
| q.pop()                           | 删除queue的首元素或priority_queue的最高优先级的元素，但不返回此元素 |
| q.front()<br />q.back()           | 返回首元素或尾元素，但不删除此元素，只适用于queue            |
| q.top()                           | 返回最高优先级元素，但不删除该元素，只适用于priority_queue   |
| q.push(item)<br />q.emplace(args) | 在queue末尾或priority_queue中且当地位置创建一个元素，其值为item，或者由args构造 |

