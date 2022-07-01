# Primer c++第五版笔记3(到第12章完)

~~这是笔记，所以别问为什么写的跟书上一摸一样了~~

[TOC]



## 10 泛型算法

### 10.1 概述

大多数算法都定义在头文件algorithm中。标准库还在头文件numeric中定义了一组数值泛型算法。

```c++
//如果找到val，返回结果指向它，否则返回vec.cend()
auto result = find(vec.cbegin(), vec.cend(), val);
int* result = find(begin(arr), end(arr), val);
```

**迭代器令算法不依赖于容器，但算法依赖于元素类型的操作**

大多数算法都使用了元素类型上的操作，比如比较大小。不过，大多数算法提供了一种方法，允许使用自定义的操作来代替默认的运算符

> **算法永远不会执行容器的操作**
>
> 泛型算法本身不会执行容器的操作，只会运行于迭代器之上，执行迭代器的操作。该特性带来了一个必要的编程假定：算法永远不会改变底层容器的大小。
>
> 标准库定义了一类特殊的迭代器，插入器（inserter）。当给这类迭代器赋值时，迭代器可以完成向容器添加元素的效果，但算法自身永远不会做这样的操作

### 10.2 初识泛型算法

除了少数例外，标准库算法都对一个范围内的元素进行操作。此元素范围称为“输入范围”。接受输入范围的算法总是使用前两个参数来表示此范围。

#### 10.2.1 只读算法

```c++
//计算vec中元素的和，和的初值是0
int sum = accumulate(vec.cbegin(), vec.cend(), 0);
//accumulate的第三个参数的类型决定了函数中使用哪个加法运算符以及返回值的类型
```

**算法和元素类型**

accumulate将第三个参数作为求和起点，这假定了：元素类型求和的操作必须是可行的

```c++
string sum = accumulate(v.cbegin(), v.cend(), string(""));

//error，const char* 上没有定义+运算符
string sum = accumulate(v.cbegin(), v.cend(), "");
```

**操作两个序列的算法**

```c++
//前两个表示第一个序列中元素范围，第三个表示第二个序列的首元素
//roster2重点元素数目至少于roster1一样多
equal(roster1.cbegin(), roster1.cend(), roster2.cbegin());
//假定比较运算符可执行
```

> 那些只接受一个单一迭代器来表示第二个序列的算法，都假定第二个序列至少与第一个序列一样长

#### 10.2.2 写容器元素的算法

使用这类算法应确保序列原大小至少不小于要求算法写入的元素数目

```c++
//接受一对迭代器表示范围，接受一个值作为第三个参数
fill(vec.begin(), vec.end(), 0);
```

**算法不检查写操作**

```c++
//接受一个单迭代器、一个计数值、一个值，将给定值赋予迭代器指向的元素开始的指定个元素
fill_n(vec.begin(), vec.size(), 0); //重置为0
fill_n(dest, n, val);//算法不检查写时，dest容量大小
```

**介绍back_inserter**

插入迭代器（insert_iterator）是一种向容器中添加元素的迭代器。使用`back_inserter`，用算法向容器写入数据，头文件为iterator

back_inserter接受一个指向容器的引用，返回一个与该容器绑定的插入迭代器。当通过此迭代器赋值时，赋值运算符会调用push_back将一个具有给定值的元素添加到容器中

```c++
vector<int> vec;
auto it = back_inserter(vec);
*it = 5;

//使用back_inserter创建一个迭代器，作为算法的目的位置来使用
fill_n(back_inserter(vec), 10, 0); //添加10个元素到vec
```

每步迭代中，fill_n向给定序列的一个元素赋值。由于传参是back_inserter返回的迭代器，因此每次赋值都会在vec上调用push_back。最终添加10个0到vec的末尾

**拷贝算法**

此算法将输入范围中的元素拷贝到目的序列中，传递给copy的目的序列至少要包含与输入序列一样多的元素。

```c++
//接受三个迭代器，前两个表示一个输入范围
//第三个表示目的序列的起始位置。
int a[] = {-11, 1, 2};
int b[sizeof(a) / sizeof(*a)];//a, b的大小一样
auto ret = copy(begin(a), end(a), b);//把a的内容拷贝给b
```

copy返回的是其目的位置迭代器（递增后）的值。即ret恰好指向拷贝到b的尾元素之后的位置。

多个算法都提供所谓的“拷贝”版本。这些算法计算新元素的值，但不会将它们放置在输入序列的末尾，二十创建一个新序列保存这些结果。

```c++
//前两个是迭代器，表示输入序列，后两个一个是要搜索的值，另一个是新值
replace(ilst.begin(), ilst.end(), 0, 42);
//保持原序列不变。接受额外第三个迭代器参数，指出调整后序列的保存位置
//ilst未改变，ivec包含ilst的拷贝，并调整了新值
replace_copy(ilst.begin(), ilst.end()
            back_inserter(ivec), 0, 42);
```

#### 10.2.3 重排容器元素算法

```c++
//按字典序排序words
sort(words.begin(), words.end());
//使用unique
auto end_unique = unique(words.begin(), words.end());
//删除重复单词
words.erase(end_unique, words.end());
```

**使用unique**

调用unique后，words的大小并未改变。但相邻的重复元素被下一个不重复元素覆盖了，使得不重复元素出现在序列的开始部分。unique返回的迭代器指向最后一个不重复元素之后的位置。此位置之后的元素仍然存在，但是不知道它们的值是什么。

### 10.3 定制操作

#### 10.3.1 向算法传递参数

sort的重载版本，接受第三个参数，此参数时一个`谓词`

**谓词**

谓词是一个可调用的表达式，其返回结果是一个能作为条件的值。标准库算法的谓词分为：一元谓词（只接受单一参数），二元谓词（有两个参数）。接受谓词参数的算法对输入序列中的元素调用谓词。

```c++
//stable_sort，除了对按照谓词规则排序外，保持元素的相对位置不变
stable_sort(words.begin(), words.end(), isShorter);
```

#### 10.3.2 lambda表达式

**介绍lambda**

可以向一个算法传递任何类别的可调用对象。对于一个对象或一个表达式，如果可以对其使用调用运算符，则称它为可调用的。目前使用过的仅有的两种可调用对象是函数和函数指针，还有其他两种可调用对象：重载了函数调用运算符的类，lambda表达式。lambda表达式具有如下形式

```c++
[capture list](parameter list) -> return type {function body}
```

其中capture list（捕获列表）是一个lambda所在函数中定义的局部变量的列表。与普通函数不同的是，lambda必须使用尾置返回来指定返回类型。参数列表和返回类型可以忽略。

```c++
//lambda的调用方式
auto f = []{return 34;};
cout << f() << endl;
```

与普通函数不同，lambda不能有默认参数

**使用捕获列表**

lambda以一对[]开始，可以在其中提供一个以逗号分割的名字列表，名字为它所在函数中定义的

```c++
[sz](const string &a) {return a.size() >= sz;};
```

**调用find_if**

标准库find_if算法来查找第一个具有特定大小的元素。该算法接受一个迭代器，表示一个范围，第三个参数是一个一元谓词。

```c++
auto wc = find_if(words.begin(), words.end(),
                 [sz](const string &a) {
                     return a.size() >= sz;
                 });
```

**for_each算法**

```c++
//for_each接受一个可调用对象，对输入序列中每个元素调用此对象
for_each(wc, words.end(), [](const string &s) {
    cout << s << " ";
});
```

> 捕获列表只用于局部非static变量，lambda可以直接使用static变量和在它所在函数之外声明的名字

#### 10.3.3 lambda捕获和返回

**值捕获**

被捕获的变量的值是在lambda创建时拷贝，而不是调用时拷贝

```c++
size_t v1 = 42;
auto f =[v1] {return v1;};
v1 = 0;
auto j = f();//j为42，保存了创建时v1的拷贝
```

**引用捕获**

```c++
size_t v1 = 42;
auto f =[v1] {return v1;};
v1 = 0;
auto j = f();//j为0
```

采用引用方式捕获一个变量，必须包装呢个被引用的对象在lambda执行的时候是存在的

如果函数返回一个lambda，则与函数不能返回一个局部变量的引用类似，此lambda也不能包含引用捕获

**隐式捕获**

```c++
//c显示值捕获，其他隐式引用捕获
[&, c] (const string& s) {os << s << c;};
//os显式引用捕获，其他隐式值捕获
[=, &os] (const string& s) {os << s << c;};
```

当混合使用隐式捕获和显式捕获时，捕获列表中的第一个元素必须是&或=。此符号指定了默认捕获方式

当混合使用隐式捕获和显式捕获时，显式捕获必须使用与隐式捕获不同的方式

**可变lambda**

默认情况下，对于值拷贝的变量，lambda不会改变其值。如果希望能改变，必须在参数列表首加上mutable关键字。

```c++
size_t v1 = 42;
auto f = [v1]() mutable {return ++v1;};
v1 = 0;
auto j = f();//j为43
```

```c++
size_t v1 = 42;
auto f = [&v1] {return ++v1;};
v1 = 0;
auto j = f();//j为1
```

**指定lambda返回类型**

默认情况下，如果一个lambda体包含return之外的任何语句，则编译器假定此lambda返回void

需要为一个lambda定义返回类型时，必须使用尾置返回类型。

```c++
transform(v.begin(), v.end(), 
          [](int i) -> int {
		  	  if(i < 0) return -i;
              else return i;
          });
```

#### 10.3.4 参数绑定

**标准库bind函数**

标准库函数bind，定义在头文件functional中。接受一个可调用对象，生成一个新的可调用对象来“适应”原对象的参数列表

bind的一般形式

```c++
auto newCallable = bind(callable, arg_list);
```

其中，newCallable本身是一个可调用对象，arg_list是一个逗号分隔的参数列表，对应给定的newCallable的参数。调用newCallable时，newCallable会调用callable，并传递给它arg_list中的参数。

arg_list中的参数可能包含形如_n的名字，其中n是一个整数，这些参数是“占位符”，表示newCallable的参数，他们占据了传递给newCallable的参数的“位置”。数值n表示生成的可调用对象中参数的位置：\_1为newCallable的第一个参数，\_2为第二个参数

**绑定check_size的sz参数**

```c++
find_if(words.begin(), words.end(), bind(check_size, std::placeholders::_1, sz));
//check_size为可调用参数，bind语句相当于check_size(const string &a, sz);
```

**使用placeholders名字**

名字_n都定义在一个名为placeholders的命名空间中，这个命名空间本身定义在std命名空间。

```c++
using std::placeholders::_1;
using namespace std::placeholders;//使用占位符命名空间里面的所有名字
```

place holders命名空间也定义在functional头文件中

**bind的参数**

```c++
//g是一个由两个参数的可调用参数
auto g = bind(f, a, b, _2, c, _1);
g(_1, _2) == f(a, b, _2, c, _1);
```

**用bind重排参数顺序**

```c++
//按单词长度由短至长排序
sort(words.begin(), words.end(), isShorter);
//按单词长度由长至短排序
sort(words.begin(), words.end(), bind(isShorter, _2, _1));
```

**绑定引用参数**

如果希望传递给bind一个对象而又不拷贝它，就必须使用标准库ref函数

```c++
for_each(words.begin(), words.end(), 
        [&os, c] (const string &s) {os << s << c;});
//等同于
ostream &print(ostream& os, const string &s, char c) {
    return os << s << c;
}
//等同于
for_each(words.begin(), words.end(), 
         bind(print, ref(os), _1, ' '));
//函数ref返回一个对象，包含给定的引用，此对象是可以拷贝的。标准库中还有一个cref函数，
//生成一个保存const引用的类。和bind一样，函数ref和cref也定义在头文件functional中
```

### 10.4 再探迭代器

* 插入迭代器(insert iterator)：这些迭代器被绑定到一个容器上，可用来向容器插入元素
* 流迭代器(stream iterator): 这些迭代器被绑定到输入或输出流上，可用来遍历所关联的IO流
* 反向迭代器(reverse iterator): 这些迭代器向后而不是向前移动。除了forward_list之外的标准库容器都有反向迭代器
* 移动迭代器(move iterator): 这些专用的迭代器不是拷贝其中的元素，而是移动它们

#### 10.4.1 插入迭代器

插入器是一种迭代器适配器，它接受一个容器，生成一个迭代器，能实现给定容器添加元素。当我们通过一个插入迭代器进行赋值时，该迭代器调用容器操作来向给定容器的指定位置插入一个元素。

**插入迭代器操作**

| it = t          | 在it指定的当前位置插入值t。假定c是it绑定的容器，依赖于插入迭代器的不同种类，此赋值会分别调用c.push_back(t), c.push_front(t)或c.insert(t, p),其中p为传递给inserter的迭代器位置 |
| --------------- | ------------------------------------------------------------ |
| *it, ++it, it++ | 这些操作虽然存在，但不会对it做任何事情。每个操作都返回it     |

插入迭代器有三种类型

* back_inserter 创建一个使用push_back的迭代器
* front_inserter创建一个使用push_front的迭代器
* insert创建一个使用insert的迭代器。此函数接受第二个参数，这个参数必须是一个指向给定容器的迭代器。元素被插入到给定迭代器所表示的元素之前。

当调用inserter(c, iter)时，会得到一个迭代器，接下来使用它时，会将元素插入到iter原来所指向的元素之前的位置。

如果it是由inserter生成的迭代器，则下面这样的赋值语句

```c++
*it = val;
//等同于
it = c.insert(it, val);//it指向新加入的元素
++it;//递增it使它指向原来的元素
```

当使用front_inserter时，元素总是插入到容器第一个元素之前，该迭代器会更新首元素的位置，而inserter则一直指向原来的位置。

```c++
list<int> lst = {1, 2, 3, 4}, lst2, lst3;
//拷贝完成后，lst2包含4 3 2 1
copy(lst.cbegin(), lst.cend(), front_inserter(lst2));
//拷贝完成后，lst3包含1 2 3 4
copy(lst.cbegin(), lst.cend(), insert(lst3, lst3.begin()));
```

#### 10.4.2 iostream迭代器

虽然iostream类型不是容器，但标准库定义了可以用于这些IO类型对象的迭代器。istream_iterator读取输入流，ostream_iterator向一个输出流写数据。这些迭代器将它们对应的流当作一个特定类型的元素序列来处理。通过使用流迭代器，可以用泛型算法从流对象读取数据以及向其写入数据。

**istream_iterator操作**

当创建一个流迭代器时，必须指定迭代器将要读写的对象类型。一个istream_iterator使用>>来读取流。因此，istream_iterator要读取的类型必须定义了输入运算符。当然，我们还可以默认初始化迭代器，这样就创建了一个可以当作尾后值使用的迭代器。

```c++
istream_iterator<int> int_it(cin); //从cin读取int
istream_iterator<int> int_eof; //尾后迭代器
ifstream in("afile");
istream_iterator<string> str_it(in); //从“afile”读取字符串
//用istream_iterator从标准输入读取数据，存入一个vector
istream_iterator<int> in_iter(cin), eof;
while(in_iter != eof)
    //后置递增运算读取流，返回迭代器的旧值
    //解引用迭代器，获得从流读取的前一个值
    vec.push_back(*in_iter++);
```

eof被定义为空的istream_iterator，从而可以当作尾后迭代器来使用。对于一个绑定到流的迭代器，一旦其关联的流遇到文件尾或遇到IO错误，迭代器的值就与尾后迭代器相等。

```c++
isistream_iterator<int> in_iter(cin), eof;
vector<int> vec(in_iter, eof); //从迭代器范围构造vec
```

本例中用一对表示元素范围的迭代器来构造vec。这两个迭代器是istream_iterator，这意味着元素范围是通过关联的流中读取数据获得的。这个构造函数从cin中读取数据，直到遇到文件尾或者遇到一个不是int的数据为止。

**istream_iterator的操作**

| istream_iterator<T> in(is); | in从输入流is读取类型为T的值                                  |
| --------------------------- | ------------------------------------------------------------ |
| istream_iterator<T> end;    | 读取类型为T的值的istream_iterator迭代器，表示尾后位置        |
| in1 == in2<br />in1 != in2  | in1和in2必须读取相同类型，如果它们都是尾后迭代器，或绑定到相同的输入，则两者相等。 |
| *in                         | 返回从流中读取的值                                           |
| in->men                     | 同(*in).men                                                  |
| ++in, in++                  | 使用元素类型所定义的>>运算符从输入流中读取下一个值。与以往一样，前置版本返回一个指向递增后迭代器的引用，后置版本返回旧值 |

**使用算法操作流迭代器**

```c++
istream_iterator<int> in(cin), eof;
cout << accumulate(in, eof, 0) << endl;
//此调用会计算出从标准输入读取的值的和
```

**istream_iterator允许使用懒惰求值**

当我们将一个istream_iteraotr绑定到一个流时，标准库并不保证迭代器立刻从流读取数据。具体实现可以推迟从流中读取数据，直到我们使用迭代器才真正读取。标准库中的实现所保证的是，当我们第一次解引用迭代器之前，从流中读取数据的操作已经完成了。

**ostream_iterator操作**

我们可以对任何具有输出运算符（<<运算符）的类型定义ostream_iterator。当创建一个ostream_iterator时，我们可以提供(可选)第二参数，一个C风格字符串（必须是，即一个字符串字面常量或一个指向以空字符结尾的字符数组的指针），在输出每个元素后都打印此字符串。必须将ostream_iterator绑定到一个指定的流，不允许空的或表示尾后位置的ostream_iterator。

**表10.4 ostream_iterator操作**

| iostream_iterator<T> out(os);   | out将类型为T的值写到输出流os中                               |
| ------------------------------- | ------------------------------------------------------------ |
| ostream_iterator<T> out(os, d); | out将类型为T的值写到输出流os中，每个值后面都输出一个d。d指向一个空字符结尾的字符数组 |
| out = val                       | 用<<运算符将val写入到out所绑定的ostream中。val的类型必须与out可写的类型兼容 |
| *out, ++out, out++              | 这些运算符是存在的，但不对out做任何事情。每个运算符都返回out |

```c++
//用ostream_iterator来输出值的序列
ostream_iterator<int> out_iter(cout, " ");
for(auto e : vec)
    *out_iter++ = e;//赋值语句实际上将元素写到cout
//将vec中的每个元素写到cout，每个元素后加空格符。每次向out_iter赋值时，写操作就会被提交
//对out_iter赋值时，可以忽略解引用和递增运算，即循环可被重写为
for(auto e : vec)
    out_iter = e;
//运算符*和++实际上对ostream_iterator对象不做任何事情。但是在第一种形式的写法中，流迭代器的使用与其他迭代器的使用保持一致，方便改写和理解。
//使用copy来打印vec中的元素
copy(vec.begin(), vec.end(), out_iter);
```

**使用流迭代器处理类类型**

可以为任何定义了输入运算符(>>)的类型创建istream_iterator对象，也可以为有输出运算符(<<)定义ostream_iterator对象。

#### 10.4.3 反向迭代器

递增一个反向迭代器(++it)会移动到前一个元素，递减类似。除了forward_list之外，其他容器都支持反向迭代器，可以通过调用rbegin、rend、crbegin和crend成员函数来获得反向迭代器。

```c++
//通过反向迭代器将vector整理为递减
sort(vec.rbegin(), vec.rend());
```

**反向迭代器和其他迭代器间的关系**

```c++
//通过调用reverse_iterator的base成员函数来完成将反向迭代器rcomma换回一个普通迭代器 
auto rcomma = find(line.crbegin(), line.crend(), ',');
//输出最后一个逗号后面的单词，不转换该单词会反向输出
cout << string(rcomma.base(), line.cend()) << endl;
```

值得注意的是，rcomma和rcomma.base()指向不同的元素，rcomma指向最后一个逗号，而rcomma指向最后一个逗号之后的第一个字符，line.crbegin()和line.cend()也是如此。

从技术上来讲，普通迭代器和反向迭代器的关系反映了左闭右开的区间范围。反向迭代器的目的是表示元素范围，而这些范围是不对称的，这导致当从一个普通迭代器初始化一个反向迭代器，或是给一个反向迭代器赋值时，结果迭代器与原迭代器指向的并不是相同的元素。

### 10.5 泛型算法结构

任何算法的最基本的特性是它要求其迭代器提供哪些操作。算法所要求的迭代器操作可以分为5个`迭代器类别`，每个算法都会对它的每个迭代器参数指明需提供哪类迭代器

**表10.5 迭代器类别**

| 输入迭代器     | 只读，不写；单遍扫描，只能递增       |
| -------------- | ------------------------------------ |
| 输出迭代器     | 只写，不读；单遍扫描，只能递增       |
| 前向迭代器     | 可读写；多遍扫描，只能递增           |
| 双向迭代器     | 可读写；多遍扫描，可递增递减         |
| 随机访问迭代器 | 可读写，多遍扫描，支持全部迭代器运算 |

#### 10.5.1 5类迭代器

*对于向一个算法传递错误类别的迭代器问题，很多编译器不会给出任何警告或提示*

**输入迭代器**：可以读取序列中的元素。输入迭代器必须支持

- 用于比较两个迭代器的相等或不相等运算符(==， !=)
- 用于推进迭代器的前置和后置递增运算(++)
- 用于读取元素的解引用运算符(*)；解引用只会出现在赋值运算符的右侧
- 箭头运算符，即解引用迭代器，并提取对象的成员

**输出迭代器**：可以看作输入迭代器功能上的补集——只写不读元素。输出迭代器必须支持

- 用于推进迭代器的前置和后置递增运算(++)
- 解引用运算符(*)，只出现在赋值运算符的右侧

**前向迭代器**：可以读写元素

**双向迭代器**：可以正向、反向读写序列中的元素

**随机访问迭代器**：提供在常量时间内访问序列中任意元素的能力。此类迭代器除了支持双向迭代器的所有功能，还支持：

- 用于比较两个迭代器相对位置的关系运算符(<, <=, >, >=)
- 迭代器和一个整数值的加减运算(+, +=, -, -=)，计算结果是迭代器在序列中前进（或后退）给定整数个元素后的位置
- 用于两个迭代器上的减法运算符(-)，得到两个迭代器的距离
- 下标运算符(iter[n])，与*(iter[n])等价

#### 10.5.2 算法形参模式

大多数算法具有如下4种形式之一：

```c++
alg(beg, end, other args);
alg(beg, end, dest, other args);
alg(beg, end, beg2, other args);
alg(beg, end, beg2, end2, other args);
```

dest参数是一个表示算法可以写入的目的位置的迭代器。如果dest是一个直接指向容器的迭代器，那么算法将输出数据写到容器中已存在的元素内。一般来说，dest被绑定到插入迭代器或ostream_iterator。

#### 10.5.3 算法命名规范

**_if版本的算法**

接受一个元素值的算法通常有另一个不同名的版本，该版本接受一个谓词代替元素值。

```c++
find(beg, end, val);//查找输入范围中val第一次出现的位置
find(beg, end, pred);//查找第一个令pred为真的元素
```

**区分拷贝元素的版本和不拷贝的版本**

写到额外目的空间的算法都在名字后面附加一个_copy

```c++
reverse(beg, end);
reverse_copy(beg, end, dest);
```

### 10.6 特定容器算法

链表类型list和forward_list定义了几个成员函数形式的算法，它们定义了独有的sort, merge, remove, reverse, unique。链表类型定义的其他算法的通用版本可以用于链表，但代价太高。

**表10.6 list和forward_list成员函数版本的算法** 这些操作都返回void

| lst.merge(lst2)<br />lst.merge(lst2, comp) | 将来自lst2的元素合并入lst。lst和lst2都必须是有序的。元素将从lst2中删除。在合并之后，lst2变为空。第一个版本使用<运算符；第二个版本使用给定的比较操作 |
| :----------------------------------------- | :----------------------------------------------------------- |
| lst.remove(val)<br />lst.remove_if(pred)   | 调用erase删除与给定值相等(==)或令一元谓词为真的每个元素      |
| lst.reverse()                              | 反转lst中元素的顺序                                          |
| lst.sort()<br />lst.sort(comp)             | 使用<或给定比较操作排序元素                                  |
| lst.unique()<br />lst.unique(pred)         | 调用erase删除同一个值的连续拷贝。第一个版本使用==；第二个版本使用给定的二元谓词 |

**splice成员**

链表类型还定义了splice算法。此算法是链表数据结构所特有的。

**表10.7 list和forward_list的splice成员函数的参数 **

| lst.splice(args) 或 flst.splice_after(args) |                                                              |
| ------------------------------------------- | ------------------------------------------------------------ |
| (p, lst2)                                   | p是一个指向lst中元素的迭代器，或一个指向flst首前位置的迭代器。函数将lst2的所有元素移动到lst中p之前的位置或是flst中p之后的位置。将元素从lst2中删除。lst2的类型必须与lst或flst相同，且不能是同一个链表 |
| (p, lst2, p2)                               | p2是一个指向lst2中位置的有效的迭代器。将p2指向的元素移动到lst中，或将p2之后的元素移动到flst中。lst2可以是与lst或flst相同的链表 |
| (p, lst2, b, e)                             | b和e必须表示lst2中的合法范围。将给定范围中的元素从lst2移动到lst或flst。lst2与lst（或flst）可以是相同的链表，但p不能只想给定范围中的元素 |

**链表特有的操作会改变容器**

链表特有版本与通用版本间的一个至关重要的区别是链表版本会改变底层的容器。

## 11 关联容器

标准库提供8个关联容器。无序容器则定义在头文件unordered_map和unordered_set中。

**表11.1 关联容器类型**

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

### 11.2 关联容器概述

#### 11.2.1 定义关联容器

可以从一个值范围来初始化关联容器

```c++
vector<int> ivec;
set<int> iset(ivec.cbegin(), ivec.cend());
multiset<int> miset(ivec.cbegin(), ivec.cend());
```

#### 11.2.2 关键字类型的要求

关键字类型必须定义元素比较的方法。在实际编程中，如果一个类型定义了“行为正常”的<运算符，则它可以用作关键字类型。

**使用关键字类型的比较函数**

用来组织一个容器中元素的操作的类型也是该容器类型的一部分。在尖括号中出现的每个类型，就仅仅是一个类型而已。当创建一个容器（对象）时，才会以构造函数参数的形式提供真正的比较操作（其类型必须与尖括号中指定的类型相吻合）。

```c++
bool compareIsbn(const Sales_data& lhs, const Sales_data& rhs) {
    return lhs.isbn() < rhs.isbn();
}
//为了使用自定义操作，定义multiset时必须提供两个类型；关键字类型Sales_data，以及比较操作——应该是一种函数指针类型。可以指向compareIsbn
//bookstore中多条记录可以有相同的ISBN
//bookstore中的元素以ISBN的顺序进行排序
multiset<Sales_data, decltype(compareIsbn)*> boookstore(compareIsbn);
//用decltype来指出自定义操作的类型。用compareIsbn来初始化bookstore对象。
```

#### 11.2.3 pair类型

标准库类型pair 定义在头文件unility中pair是一个用来生成特定类型的模板。pair的默认构造函数对数据成员进行值初始化。与其他标准库类型不同，pair的数据成员是pulic的。两个成员分别命名为first和second。

**表11.2 pair上的操作**

| pair<T1, T2> p;                                        | p是一个pair，两个类型分别为T1和T2的成员都进行了值初始化      |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| pair<T1, T2> p(v1, v2)<br />pair<T1, T2> p = (v1, v2); | p是一个成员类型为T1和T2的pair；first和second成员分别用v1和v2进行初始化 |
| make_pair(v1, v2);                                     | 返回一个用v1和v2初始化的pair。pair的类型从v1和v2的类型推断出来 |
| p.first                                                | 返回p的名为first的（公有）数据成员                           |
| p.second                                               | 返回p的名为second的（公有）数据成员                          |
| p1 relop p2                                            | 关系运算符(<. >, <=, >=)按字典序定义                         |
| p1 == p2<br />p1 != p2                                 | 当first和second成员分别相等时，两个pair相等，相等性判断利用元素的==运算符实现 |

**创建pair对象的函数**

### 11.3 关联容器操作

除表9.2中列出的类型，关联容器还定义了下表中列出的类型。

**表11.3 关联容器额外的类型别名**

| key_type    | 此容器类型的关键字类型                                       |
| ----------- | ------------------------------------------------------------ |
| mapped_type | 每个关键字关联的类型；只适用于map                            |
| value_type  | 对于set，同key_type；对于map，为pair<const key_type, mapped_type> |

对于set类型，key_type和value_type是一样的。在map中，由于不能改变一个元素的关键字，因此这些pair的关键字部分是const的。

```c++
set<string>::value_type v1;//v1是一个string
set<string>::key_type v2;//v2是一个string
map<string, int>::value_type v3;//v3是一个pair<const string, int>
map<string, int>::key_type v4;//v4 是一个string
map<string, int>::mapped_type v5;//v5是一个int
```

**set的迭代器是const的**

**关联容器和算法**，通常不对关联容器使用泛型算法。在实际编程中，如果真要对一个关联容器使用算法，不是将它作为一个源序列，就是当作一个目的位置。

#### 11.3.2 添加元素

**向map添加元素**

```c++
word_count.insert({word, 1});
word_count.insert(make_pair(word, 1));
word_count.insert(pair<string, size_t>(word, 1));
word_count.insert(map<string, size_t>::value_type(word, 1));
```

**表11.4 关联容器insert操作**

| c.insert(v)<br />c.emplace(args)       | v是value_type类型的对象；args用来构造一个元素<br />对于map和set，只有当元素的关键字不在c中时才插入（或构造）元素。函数返回一个pair，包含一个迭代器，指向具有指定关键字的元素，以及一个指示插入是否成功的bool值。<br />对于multimap和multiset，总会插入（或构造）给定元素，并返回一个指向新元素的迭代器 |
| -------------------------------------- | ------------------------------------------------------------ |
| c.insert(b, e)<br />c.insert(i1)       | b和e是迭代器，表示一个c::value_type类型值的范围；i1是这种值的花括号列表。函数返回void<br />对于map和set，只插入关键字不在c 中的元素。对于multimap和multiset，则会插入范围中的每个元素 |
| c.insert(p, v)<br />c.emplace(p, args) | 类似insert(v)(或emplace(args) )，但将迭代器p作为一个提示，指出从哪里开始搜索新元素应该存储的位置。返回一个迭代器，指向具有给定关键词的元素。 |

#### 11.3.3 删除元素

对于保存不重复关键字的容器，erase的返回值总是0或1.若返回值为0，则表明想要删除的元素并不在容器中。对于允许重复关键字的容器，删除元素的数量可能大于1。

**表11.5 从关联容器删除元素**

| c.erase(k)    | 从c中删除每个关键字为k的元素。返回一个size_type值，指出删除元素的数量 |
| ------------- | ------------------------------------------------------------ |
| c.erase(p)    | 从c中删除迭代器p指定的元素。p必须指向c中一个真实元素，不能等于c.end()。返回一个指向p之后元素的迭代器，若p指向c中的尾元素，则返回c.end() |
| c.erase(b, e) | 删除迭代器b和e所表示的范围中的元素。返回e                    |

#### 11.3.4 map的下标操作

map下标运算符接受一个索引，获取与此关键字相关联的值。如果关键字并不在map中，会为它创建一个元素并插入到map中，关联值将进行值初始化。

```c++
map<string, size_t> word_count;
word_count["Anna"] = 1;
//会执行如下操作
//在word_count中搜索关键词为Anna的元素，未找到
//将一个新的关键字-值对插入到word_count中。关键字是一个const string，保存Anna值进行值初始化。
//日去除新插入的元素，并将值1赋予它
```

**表11.6 map和unordered_map的下标操作**

| c[k]    | 返回关键字为k的元素；如果k不在c中，添加一个关键字为k的元素，对其进行值初始化 |
| ------- | ------------------------------------------------------------ |
| c.at(k) | 访问关键字为k的元素，带参数检查；如果k不在c中，抛出一个out_of_range异常 |

*与vector与string不同，map的下标运算符返回的类型与解引用map迭代器得到的类型不同*

**表11.7 在一个关联容器中查找元素的操作**

lower_bound和upper_bound不适用于无序容器

下标和at操作只适用于非const的map和unordered_map

| c.find(k)        | 返回一个迭代器，指向第一个关键字为k的元素，若k不在容器中，则返回尾后迭代器 |
| ---------------- | ------------------------------------------------------------ |
| c.count(k)       | 返回关键字等于k的元素的数量。对于不允许重复关键字的容器，返回值永远是0或1 |
| c.lower_bound(k) | 返回一个迭代器，指向第一个关键字不小于k的元素                |
| c.upper_bound(k) | 返回一个迭代器，指向第一个关键字大于k的元素                  |
| c.equal_range(k) | 返回一个迭代器pair，表示关键字等于k的元素的范围。若k不存在，pair的两个成员均等于c.end() |

**在multimap或multiset中查找元素**

```c++
string search_item("Aliain de Botton");
auto entries = authors.count(search_item);
auto iter = authors.find(earch_item);
while(entries) {
    cout << iter->second << endl;
    ++iter;
    --entries;
}

//面向迭代器的解决方法
for(auto beg = authors.lower_bound(search_item),
         end = authors.upper_bound(search_item);
    beg != end;++beg) 
    cout << beg->second << endl;

//equal_range若未找到匹配元素，则两个迭代器都指向关键字可以插入的位置
for(auto pos = authors.equal_range(earch_item);
         pos.first != pos.second;++pos.first)
    cout << pos.first->second << endl;
```

### 11.4 无序容器

**无序容器管理操作**

| 桶接口                 |                                                              |
| ---------------------- | ------------------------------------------------------------ |
| c.bucket_count()       | 正在使用的桶的数目                                           |
| c.max_bucket_count()   | 容器能容纳的最多的桶的数量                                   |
| c.bucket_size(n)       | 第n个桶中有多少个元素                                        |
| c.bucket(k)            | 关键字为k的元素在哪个桶中                                    |
| 桶迭代                 |                                                              |
| local_iterator         | 可以用来访问桶中元素的迭代器类型                             |
| const_local_iterator   | 桶迭代器的const版本                                          |
| c.begin(n), c.end(n)   | 桶n的首元素迭代器和尾后迭代器                                |
| c.cbegin(n), c.cend(n) | 与上类似，返回const_local_iterator                           |
| 哈希策略               |                                                              |
| c.load_factor()        | 每个桶的平均元素数量，返回float值                            |
| c.max_load_factor()    | c试图维护的平均桶大小，返回float值。c会在需要时添加新的桶，以使得load_factor <= max_load_factor |
| c.rehash(n)            | 重组存储，使得bucket_count >= n且bucket_count > size/max_load_factor |
| c.reserve(n)           | 重组存储，使得c可以保存n个元素不必rehash                     |

**无序容器对关键字类型的要求**

默认情况下，无序容器使用关键字类型的==元素符来比较元素，还使用一个hash<key_type>类型的对象来生成每个元素的哈希值。因为标准库为内置类型（包括指针）提供了hash模板，所以可以直接定义关键字是内置类型（包括指针类型）的无序容器。

但是对于自定义类型的关键字，无法定义其无序容器，除非提供其hash模板版本。

```c++
//自定义类型的hash
size_t hasher(const Sales_data& sd) {
    return hash<string>()(sd.isbn);
}
bool eqOp(const Sales_data& lhs, const Sales_data& rhs) {
    return lhs.isbn() == rhs.isbn();
}
using SD_multiset = unodered_multiset<Sales_data, 
                   decltype(hasher)*, decltype(eqOp)*>;
//参数是桶大小、哈希函数指针和相等性判断运算符指针
SD_multiset bookstore(42, hasher, eqOp);

//如果类型定义了==元素安抚，则可以只重载哈希函数
unordered_set<Foo, decltype(FooHash)*> fooSet(10, FooHash);
```

## 12 动态内存

动态分配的对象的生存期与它们在哪里创建是无关的，只有当显式地被释放时，这些对象才会销毁。

静态内存用来保存局部static对象、类static数据成员以及定义在任何函数之外的变量。栈内存用来保存定义在函数内的非static对象。程序用`堆`来存储`动态分配`的对象。动态对象的生存期由程序来控制，即当动态对象不再使用时，代码必须显式销毁它们。

### 12.1 动态内存和智能指针

新标准库提供了两种智能指针类型来管理动态对象。智能指针与常规指针的区别是它负责自动释放所指的对象。shared_ptr允许多个指针指向同一个对象；unique_ptr则“独占”所指向的对象。标准库还定义了名为weak_ptr的伴随类，它是一种弱引用，指向shared_ptr所管理的对象。这三种类型都定义在memory头文件中。

#### 12.1.1 shared_ptr类

默认初始化的智能指针中保存一个空指针。

**表12.1 shared_ptr 和unique_ptr 都支持的操作**

| shared_ptr<T> sp<br />unique_ptr<T> up | 空智能指针，可以指向类型为T的对象                            |
| -------------------------------------- | ------------------------------------------------------------ |
| p                                      | 将p用作一个条件判断，若p指向一个对象，则为true               |
| *p                                     | 解引用p，获得它指向的对象                                    |
| p->mem                                 | 等价于(*p).mem                                               |
| p.get()                                | 返回p中保存的指针。若智能指针呢释放了其对象，返回的指针所指向的对象也就消失了 |
| swap(p, q)<br />p.swap(q)              | 交换p 和q 中的指针                                           |

**表12.2 shared_ptr独有的操作**

| make_shared<T>(args) | 返回一个shared_ptr，指向一个动态分配的类型为T的对象，使用args初始化此对象 |
| -------------------- | ------------------------------------------------------------ |
| shared_ptr<T> p(q)   | p 是shared_ptr q的拷贝；此操作会递增q中的计数器。q中的指针必须能转换为T* |
| p = q                | p 和q 都是shared_ptr，所保存的指针必须能互相转换。此操作会递减p的引用技术，递增q的引用计数；若p的引用计数变为0，则将其管理的原内存释放 |
| p.unique()           | 若p.use_count()为1，返回true，否则返回false                  |
| p.use_count()        | 返回与p 共享对象的智能指针数量；可能很慢，主要用于调试       |

**make_shared函数**

```c++
    shared_ptr<int> p3 = make_shared<int>(42);//42
    shared_ptr<string> p4 = make_shared<string>(10, '9');//"999999999"
    shared_ptr<int> p5 = make_shared<int>();//int 0
    auto p6 = make_shared<vector<string> >();//指向空vector<string>
```

**shared_ptr的拷贝和赋值**

```c++
    shared_ptr<int> p = make_shared<int>(42);//42
    auto q(p);//p和q指向相同对象，此对象有两个引用者
```

每个shared_ptr都有一个关联的计数器，称为`引用计数`。无论何时拷贝一个shared_ptr，计数器都会递增，当shared_ptr赋予一个新值或shared_ptr被销毁，计数器就会递减。一旦一个shared_ptr的计数器变为0，它就会自动释放自己所管理的对象。

shared_ptr通过析构函数自动销毁所管理的对象，并且自动释放相关联的内存。

```c++
//返回一个shared_ptr，指向一个动态分配的对象
shared_ptr<Foo> factory(T arg) {
    return make_sahred<Foo>(arg);
}

void use_factory(T arg) {
    shared_ptr<Foo> p = factory(arg);//使用p
}//p离开了作用域，它指向的内存会被自动释放掉

shared_ptr<T> use_factory(T arg) {
    shared_ptr<T> p = factory(arg);//use p
    return p;//引用计数进行递增
}//p离开了作用域，但其指向的内存不会被释放掉
```

*记得 erase删除容器中不需要的智能指针*

程序使用动态内存处于以下三种原因之一：

- 程序不知道自己需要使用多少对象，比如容器
- 程序不知道所需对象的准确类型
- 程序需要在多个对象间共享数据

```c++
//多个对象间共享数据的类
class StrBlob {
public:
    typedef vector<string>::size_type size_type;
    StrBlob();
    StrBlob(initializer_list<string> i1);
    size_type size() const {return data->size();}
    bool empty() const {return data->empty();}
    //add or delete element
    void push_back(const string& t) {data->push_back(t);}
    void pop_back();
    //visit element
    string& front();
    string& back();
private:
    shared_ptr<vector<string> > data;
    void check(size_type i, const string &msg) const;
};

StrBlob::StrBlob(): data(make_shared<vector<string> >()) {}
StrBlob::StrBlob(initializer_list<string> i1):
         data(make_shared<vector<string> >(i1)){}

void StrBlob::check(StrBlob::size_type i, const string &msg) const {
    if(i >= data->size())
        throw out_of_range(msg);
}

string &StrBlob::front() {
    check(0, "front on empty StrBlob");
    return data->front();
}

string &StrBlob::back() {
    check(0, "back on empty StrBlob");
    return data->back();
}

void StrBlob::pop_back() {
    check(0, "pop_back on empty StrBlob");
    data->pop_back();
}
```

#### 12.1.2 直接管理内存

运算符new分配内存，delete释放new分配的内存。在类型名之后跟空括号可进行值初始化。对于定义了构造函数的类类型来说，如string，值初始化是没有意义的，无论数目形式，对象都会通过默认构造函数来初始化。

```c++
int *pi1 = new int; //默认初始化，*pi1的值未定义
int *pi2 = new int();//值初始化为0

//可以使用auto从初始化器推断像分配的对象类型，只有当括号中仅有一个初始化器时才能使用auto
auto p1 = new auto(obj);//p指向一个与obj类型相同的对象
```

用new分配const对象是合法的，动态分配的const对象必须进行初始化。对于定义了默认构造函数的类类型，其const动态对象可以隐式初始化。new分配的const对象时返回的指针是指向const的。

**内存耗尽**

默认情况下，如果new不能分配所要求的内存空间，会抛出类型为bad_alloc的异常。可以通过改变使用new的方式来阻止异常抛出。

```c++
int *p2 = new (nothrow) int;//如果分配失败，new返回一个空指针
```

这种形式的new称为`定位new`。定位new表达式允许想new传递额外的参数。bad_alloc和nothrow都定义在头文件new中

**释放动态内存**

`delete表达式`将动态内存归还系统，它执行两个动作：销毁给定的指针指向的对象，释放对应的内存。

**指针值和delete**

传递给delete的指针必须指向动态分配的内存，或者是一个空指针。释放一块并非new分配的内存，或者将相同的指针值释放很多次，其行为是未定义的。

```c++
//释放const对象
const int *pci = new const int(1024);
delete pci;
```

**动态对象的生存期直到被释放时为止**

与类类型不同，内置类型的对象被销毁时什么也不会发生。特别是，当一个指针离开其作用域时，它所指向的对象什么也不会发生。如果这个指针指向的是动态内存，那么内存将不会被自动释放。

> 使用new和delete管理动态内存存在三个常见问题：
>
> 1、忘记delete内存。忘记释放动态内存会导致“内存泄漏”问题，因为这种内存永远不可能被归还给自由空间了。
>
> 2、使用已经释放掉的对象。通过在释放内存后将指针置为空，优势可以检测出这种错误。
>
> 3、同一块内存释放两次。

**delete之后重置指针值**

当delete一个指针后，指针值就变得无效了，但在很多机器中指针仍然保存着（已经释放了的）动态内存的地址。在delete之后，指针就变成了`空悬指针`，即指向一块曾经保存数据对象但现在已经无效的内存的指针。为了避免空悬指针的问题，在指针即将离开其作用域之前释放掉它所关联的内存。如果需要保留指针，而一再delete之后给指针赋值nullptr。

然而，如果由多个指针指向相同的内存，在delete内存后重置指针的方法只对该指针有效，对于其他指向（已释放的）内存的指针时没有作用的。

#### 12.1.3 shared_ptr和new结合使用

可以用new返回的指针来初始化智能指针，接受指针参数的智能指针构造函数时explicit的。所以将内置指针转换为智能指针，必须使用直接初始化形式来初始化智能指针。

```c++
shared_ptr<int> p2(new int(1024));
```

默认情况下，用来初始化智能指针的普通指针必须指向动态内存，因为智能指针默认使用delete释放它所关联的对象。可以将智能指针绑定到指向其他类型的资源的指针上，但是这样必须提供自己的操作来替代delete。

**表12.3 定义和改变shared_ptr的其他方法**

| shared_ptr<T> p(q)                           | p管理内置指针q所指向的对象；q必须指向new分配的内存，且能够转换为T*类型 |
| -------------------------------------------- | ------------------------------------------------------------ |
| shared_ptr<T> p(u)                           | p从unique_ptr u那里接管了对象的所有权；将u置空               |
| shared_ptr<T> p(q, d)                        | p接管了内置指针q所指向的对象的所有权。q必须能转换为T*类型。p将使用可调用对象d来代替delete |
| shared_ptr<T> p(p2, d)                       | p时shared_ptr p2的拷贝，唯一的区别是p将用可调用对象d来代替delete |
| p.reset()<br />p.reset(q)<br />p.reset(q, d) | 若p是唯一指向其对象的shared_ptr，reset会释放此对象。若传递了可选的参数内置指针q，会令p指向q，否则会将p置空。若还传递了参数d，将会调用d而不是delete来释放q |

**不要混合使用普通指针和智能指针**

shared_ptr可以协调对象的析构，但这仅限于其自身的拷贝之间。当shared_ptr被销毁，它所指向的内存会被释放，此时指向该内存的普通指针就变成空悬指针。

不能delete get返回的指针。特别是，永远不要用get初始化另一个智能指针或者为另一个智能指针赋值。

```c++
shared_ptr<int> p(new int(42));//引用计数1
int* q = p.get();//不能让q被delete
{
    shared_ptr<int>(q);//未定义，两个独立的shared_ptr指向相同的内存
}//q被销毁，所指向的内存被释放
int foo = *p;//未定义，p所指向的内存已经被释放了
```

```c++
p.reset(new int(1024));//p指向一个新对象，重置引用计数
```

#### 12.1.4 智能指针和异常

即使是程序块产生异常，只要智能指针的对象被销毁，它所指向的内存就会被释放掉。而普通指针在delete之前都不会被释放掉。

**智能指针和哑类**

如果在资源分配和释放之间发生了异常，程序也会发生资源泄漏。

```c++
struct destination;
struct connection;//information for connect
connection connect(destination*);//open connection
void disconnect(connection);//close connection
void end_connection(connection* p) {disconnect(*p);}

void f(destination& d) {
    connection c = connect(&d);//建立连接
    shared_ptr<connect> p(&c, end_connection);//添加删除器
    //当f退出时（即使是由于异常而退出），connection会被正确关闭
}
```

> 使用智能指针的基本规范：
>
> 不适用相同的内置指针值初始化(或reset)多个智能指针。
>
> 不delete get()返回的指针
>
> 不使用get()初始化或reset另一个智能指针
>
> 如果使用get()返回的指针，当最后一个对应的智能指针销毁后，指针就变得无效了
>
> 如果使用智能指针管理的资源不是new分配的内存，需要传递给它一个删除器

#### 12.1.5 unique_ptr

某个时刻只能有一个unique_ptr指向一个给定对象。当unique_ptr被销毁时，它所指向的对象也被销毁。当定义一个unique_ptr时，需要将其绑定到一个new返回的指针上。

```c++
unique_ptr<double> p1;
unique_ptr<int> p2(new int(42));

//由于一个unique_ptr拥有它指向的对象，因此unique_ptr不支持普通的拷贝或赋值操作
unique_ptr<string> p1(new string("gsudfhg"));
unique_ptr<stirng> p2(p1);//error
unique_ptr<string> p3;p3 = p2;//error
```

**表12.4 unique_ptr操作**

| unique_ptr<T> u1<br />unique_ptr<T, D> u2 | 空unique_ptr，可以指向类型为T的对象。u1会使用delete来释放它的指针；u2会使用一个类型为D的可调用对象来释放它的指针 |
| ----------------------------------------- | ------------------------------------------------------------ |
| unique_ptr<T, D> u(d)                     | 空unique_ptr，指向类型为T的对象，用类型为D的对象d代替delete  |
| u = nullptr                               | 释放u指向的对象，将u置为空                                   |
| u.release()                               | u放弃对指针的控制权，返回指针，并将u置空                     |
| u.reset()                                 | 释放u指向的对象                                              |
| u.reset(q)<br />u.reset(nullptr)          | 如果提供了内置指针q，令u指向这个对象，否则将u置空            |

```c++
unique_string> p2(p1.release());//将所有权从p1转移给p2

p2.release();//错误，p2不会释放内存，并且丢失了指针
auto p = p2.release();//正确，必须delete p
```

**传递unique_ptr参数和返回unique_ptr**

不能拷贝unique_ptr的规则有一个例外：可以拷贝或赋值一个将要被销毁的unique_ptr。

```c++
unique_ptr<int> clone(int p) {
    return unique_ptr<int>(new int(p));
}
unique_ptr<int> clone(int p) {
    unique_ptr<int> ret(new int (p));
    //...
    return ret;
}
```

编译器都知道要返回的对象将要被销毁，在此情况下，编译器执行一种特殊的“拷贝”，在13.6.2节介绍。

> 向后兼容：auto_ptr
>
> 标准库的较早版本包含了名为auto_ptr的类，它具有unique_ptr的部分特性，但不是全部。特别是不能再容器中保存auto_ptr，也不能从函数中返回auto_ptr。
>
> 虽然auto_ptr仍是标准库的一部分，但推荐使用unique_ptr

**向unique_ptr传递删除器**

unique_ptr默认情况下用delete释放它指向的对象，也可以重载unique_ptr中默认的删除器，但是unique_ptr管理删除器的方式与shared_ptr不同，原因在16.1.6节介绍。重载一个unique_ptr中的删除器会影响到unique_ptr类型以及如何构造（或reset）该类型的对象。

```c++
//p指向一个类型位objT的对象，并使用一个类型为delT的对象释放objT对象
//调用一个名为fcn的delT类型对象
unique_ptr<objT, delT> p (new objT, fcn);

void f(destination &d) {
    connection c = connect(&d);
    unique_ptr<connection, decltype(end_connection)*>
        p(&c, end_connection);
}
```

#### 12.1.6 weak_ptr

weak_ptr是一种不控制所指向对象生存期的智能指针，它指向由一个shared_ptr所管理的对象。将一个weak_ptr绑定到一个shared_ptr不会改变shared_ptr的引用计数。一旦最后一个指向对象的shared_ptr被销毁，对象就会被释放。

**表12.5 weak_ptr**

| weak_ptr<T> w     | 空weak_ptr可以指向类型为T的对象                              |
| ----------------- | ------------------------------------------------------------ |
| weak_ptr<T> w(sp) | 与shared_ptr sp指向相同对象的weak_ptr。T必须能转换为sp指向的类型 |
| w = p             | p可以是一个shared_ptr或一个weak_ptr。赋值后w与p共享对象      |
| w.reset()         | 将w置空                                                      |
| w.use_count()     | 与w共享对象的shared_ptr的数量                                |
| w.expired()       | 若w.use_count()为0，返回true，否则返回false                  |
| w.lock()          | 如果expired为true，返回空shared_ptr，否则返回指向w对象的shared_ptr |

**核查指针类**

weak_ptr的一个用途，是为StrBlob类定义一个伴随指针类

```c++
class StrBlobPtr {
public:
    StrBlobPtr(): curr(0){};
    StrBlobPtr(StrBlob& a, size_t sz = 0):
                wptr(a.data), curr(sz) {};
    string& deref() const;
    StrBlobPtr& incr();
private:
    //if check success,return shared_ptr which point to vector
    shared_ptr<vector<string> >
            check(size_t, const string&) const;
    weak_ptr<vector<string> > wptr;
    size_t curr;
};

shared_ptr<vector<string> > 
        StrBlobPtr::check(size_t i, const string &msg) const {
    auto ret = wptr.lock();
    if(!ret)
        throw runtime_error("unbound StrBlobPtr");
    if(i >= ret->size())
        throw runtime_error("msg");
    return ret;
}

string& StrBlobPtr::deref() const {
    auto p = check(curr, "dereference past end");
    return (*p)[curr];
}

StrBlobPtr& StrBlobPtr::incr() {
    check(curr, "increment past end of StrBlobPtr");
    ++curr;
    return *this;
}
```

### 12.2 动态数组

C++语言定义了另一种new表达式语法，可以分配并初始化一个对象数组。标准库中包含一个名为allocator的类，允许将分配和初始化分离。

#### 12.2.1 new和数组

```c++
//分配get_size个int
int* pia = new int[get_size()];//pia指向第一个int
//方括号中的大小必须是整型，但不必是常量
typedef int arrT[42];//arrT表示42个int的数组类型
int* p = new arrT;//分配一个42个int的数组，同int* p = new int[42];
```

> 要记住所说的动态数组并不是数组类型

**初始化动态分配对象的数组**

```c++
int* pia = new int[10];//未初始化
int* pia = new int[10]();//全部值初始化为0
string* psa = new string[10];//空string
string* psa2 = new string[10]();//空string

//新标准中，可以提供元素初始化器的花括号列表
int* pia3 = new int[4]{1, 2, 3, 4};
string* psa3= new string[10] = {"a", "an", "the", string(3, 'd')};
```

如果初始化器数目大于大于元素数目，则new表达式失败，不会分配任何内存。

**动态分配一个空数组是合法的**

```c++
    char arr[0];//错误，不能定义长度为0的数组
    char* cp = new char[0];//正确，但cp不能解引用
```

用new分配一个大小为0的数组时，new返回一个合法的非空指针。此指针保证与new返回的其他任何指针都不相同，辞职真如同尾后指针可以像尾后迭代器一样使用该指针。但此指针不能解引用。

**释放动态数组**

```c++
delete p;//p必须指向一个动态分配的对象或为空
delete [] pa;//pa必须指向一个动态分配的数组或为空
```

第二条语句销毁pa指向的数组中的元素，并释放对应的内存。数组中的元素按逆序销毁。

**智能指针和动态数组**

标准库提供了一个可以管理new分配的数组的unique_ptr版本。

```c++
//up指向一个包含10个未初始化int的数组
unique_ptr<int[]> up(new int[10]);
up.release();//自动用delete[]销毁其指针
```

当unique_ptr指向一个数组时，不能使用点或箭头成员运算符，可以使用下标运算符来访问数组中的元素。

**表12.6 指向数组的unique_ptr**

指向数组的unique_ptr不支持成员访问运算符（点或箭头运算符）

其他unique_ptr操作不变

| unique_ptr<T[]> u    | u可以指向一个动态分配的数组，数组元素类型为T                |
| -------------------- | ----------------------------------------------------------- |
| unique_ptr<T[]> u(p) | u指向内置指针p所指向的动态分配的数组。p必须能转换为类型为T* |
| u[i]                 | 返回u拥有的数组中位置i 处的对象<br />u必须指向一个数组      |

shared_ptr不支持管理动态数组。如果希望使用shared_ptr管理一个动态数组，必须提供自己定义的删除器

```c++
shared_ptr<int> sp(new int[10], [](int *p) {delete[] p;});
sp.reset();
```

默认情况下，shared_ptr使用delete销毁它所指向的对象。如果此对象是一个动态数组，对其使用delete所产生的问题和释放一个动态数组指针时忘记[]产生的问题一样。

```c++
//shared_ptr未定义下标运算符，并且不支持指针的算术运算
for(size_t i = 0;i != 10;++i)
    *(sp.get() + i) = i;//使用get获取内置指针
```

#### 12.2.2 allocator类

new在灵活性上的局限，其中一方面表现在它将内存分配和对象构造组合在了一起。类似的，delete将对象析构和内存释放组合在了一起。而在一些情况下，我们希望将内存分配和对象构造分离，这意味着我们可以分配大块内存，只有在真正需要时才真正执行对象创建操作（同时付出一定开销）。

没有默认构造函数的类就不能动态分配数组

**allocator类**

标准库`allocator`类定义在头文件memory中，它将内存分配和对象构造分离开。它提供一种类型感知的分配方法，它分配的内存时原始的，未构造的。当一个allocator对象分配内存时，它会根据给定的对象类型来确定恰当的内存大小和对其位置。

```c++
allocator<string> alloc;//可分配string的allocator对象
auto const p =  alloc.allocate(n);//分配n个未初始化的string
```

**表12.7 标准库allocator类及其算法**

| allocator<T> a       | 定义了一个名为a的allocator对象，可以为类型为T的对象分配内存  |
| -------------------- | ------------------------------------------------------------ |
| a.allocate(n)        | 分配一段原始的，未构造的内存，保存n个类型为T的对象           |
| a.deallocate(p, n)   | 释放从T*指针p中地址开始的内存，这块内存保存了n个类型为T的对象；p必须是先前由allocate返回的指针，且n必须是p创建时所要求的大小。在调用deallocate之前，用户必须对每个在这块内存中创建的对象调用destroy |
| c.construct(p, args) | p必须是一个类型为T*的指针，指向一块原始内存；arg被传递给类型为T的构造函数，用来在p指向的内存中构造一个对象 |
| a.destroy(p)         | p为T*类型的指针，此算法对p指向的对象执行析构函数             |

**allocator分配未构造的内存**

construct成员函数接受一个指针和零个或多个额外参数，在给定位置构造一个元素。额外参数用来初始化构造的对象。

```c++
auto q = p;//q指向最后构造的元素之后的位置
alloc.construct(q++);//*q为空字符串
alloc.construct(q++, 10, 'c');//*q为cccccccccc
alloc.construct(q++, "hi");

while(q != p)//只能对真正构造了的元素进行destroy操作
    alloc.destroy(--q);//释放真正构造的string

//释放内存
alloc.deallocate(p, n);
```

**表12.8 allocator算法**

这些函数在给定目的位置创建元素，而不是由系统分配内存给它们

| uninitialized_copy(b, e, b2)   | 从迭代器b和e指出的输入范围中拷贝元素到迭代器b2指定的未构造的原始内存中。b2指向的内存必须足够大，能容纳输入序列中元素的拷贝 |
| ------------------------------ | ------------------------------------------------------------ |
| uninitialized_copy_n(b, n, b2) | 从迭代器b指向的元素开始，拷贝n个元素到b2开始的内存中         |
| uninitialized_fill(b, e, t)    | 从迭代器b和e指定的原始内存范围中创建对象，对象的值均为t的拷贝 |
| uninitialized_fill_n(b, n, t)  | 从迭代器b指向的内存地址开始创建n个对象。b必须指向足够大的未构造的原始内存，能够容纳给定数量的对象 |

```c++
auto p = alloc.allocate(v.size() * 2);
auto q = uninitialized_copy(vi.begin(), vi.end(), p);
uninitialized_fill_n(q, vi.size(), 42);
```

与copy不同，uninitialized_copy在给定目的位置构造元素。依次uninitialized_copy调用会返回一个指针，指向最后一个构造的元素之后的位置。

### 12.3 使用标准库：文本查询程序

```c++
class QueryResult;
class TextQuery {
public:
    using line_no = std::vector<std::string>::size_type;
    TextQuery(std::ifstream&);
    QueryResult query(const std::string&) const;
private:
    std::shared_ptr<std::vector<std::string> > file;
    std::map<std::string,
        std::shared_ptr<std::set<line_no> > > wm;
};

TextQuery::TextQuery(std::ifstream& is) : file(new std::vector<std::string>) {
    std::string text;
    while (getline(is, text)) {
        file->push_back(text);
        int n = file->size() - 1;//当前行号
        std::istringstream line(text);//将行文分解为单词
        std::string word;
        while (line >> word) {
            //如果单词不在wm，在下标对应项添加
            auto& lines = wm[word];//lines是一个shared_ptr
            if (!lines)//第一次进行值初始化，为空指针
                lines.reset(new std::set<line_no>);
            lines->insert(n);
        }
    }
}
```

