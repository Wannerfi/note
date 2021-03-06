[TOC]



# Primer c++第五版笔记1(到第七章完)

~~这是笔记，所以别问为什么写的跟书上一摸一样了~~



## 2 变量和基本类型

### 2.3 复合类型

#### 2.3.1 引用

定义引用时，程序把引用和它的初始值 ` 绑定`在一起，而不是将初始值拷贝给引用。一旦初始化完成，引用将和它的初始值一直绑定在一起。因为无法令引用重新绑定到另外一个对象，因此引用必须初始化。

> 引用并非对象，相反地，它只是为一个已经存在的对象所起的另外一个名字。

#### 2.3.2 指针

指针是`指向`另外一种类型的复合类型，实现了对其他对象的间接访问，与引用有很多不同，其一，指针本身就是一个对象，允许对指针赋值和拷贝，而且在指针的生命周期内它可以先后指向几个不同的对象，其二，指针无须在定义时赋初值。

> int* &r = p;//r是对指针p的引用
>
> 理解r的类型，最简单的办法是从右到左阅读r的定义。离变量名最近的符号对变量的类型有最直接的影响，因此r是一个引用，声明符的其余部分用以确定r引用的类型是什么，此例中的符号*说明r引用的是一个指针。

### 2.4 const限定符

**只能在const类型的对象上执行不改变其内容的操作**

> 默认状态下，const对象仅在文件内有效
>
> 只在一个文件中定义const，在其他多个文件中声明并使用它。解决办法是，对于const变量不管是声明还是定义都添加extern关键字，这样只需定义一次就可以了

#### 2.4.1 const的引用

把引用绑定到const对象上，称为`对常量的引用（reference to const）`，与普通引用不同的是，对常量的引用不能被用作修改它所绑定的对象。

```c++
double dval = 3.14;
const int &ri = dval;
//编译器把上述代码编程如下形式
const int temp = dval;
const int &ri = temp;//此时ri绑定到一个临时量上

const double &r2 = dval;//不允许通过r2改变dval的值
```

#### 2.4.2 指针和const

`指向常量的指针`不能用于改变其所指对象的值。

```c++
const double cpi = 3.14;
double pi = 3.14;
const double *ptr = &pi;
ptr = &pi;//允许常量指针指向非常量对象，但是不能通过该指针改变对象的值
```

> **所谓指向常量的指针或引用，不过是指针或引用自己觉得指向了常量，所以不去改变所指对象的值**

```c++
int a = 0;
int *const ptr = &a;//必须初始化，*放在cosnt关键字之前说明指针是一个常量，不变的是指针本身的值而非指向的值。
```

**想弄清声明的含义最有效的办法是从右向左阅读**

```c++
const double pi = 3.1415926;
const double *const pip = &pi;//pip是一个常量指针，指向双精度浮点型常量
```

#### 2.4.3 顶层const

**顶层const**表示指针本身是一个常量，**底层const**表示指针所指的对象是一个常量。

```c++
int i = 0;
int *const p1 = &i; //p1不能改变，顶层const
const int ci = 42; //ci不能改变，顶层const
const int *p2 = &ci; //p2可以改变，底层const
const int *const p3 = p2; //靠右的const为顶层const，靠左的是底层const
const int &r = ci; //用于声明引用的const都是底层const
```

#### 2.4.4 constexptr和常量表达式

**常量表达式**（const expression）是指值不会改变并且在编译过程就能得到计算结果的表达式。 

允许声明变量为**constexpr**类型一边由编译器来验证变量的值是否是一个常量表达式。

```c++
constexpr int sz = size();//只有当size()是一个constexpr函数时才是一条正确的声明语句
constexpr int mf = 20;
constexpr int limit = mf + 1;
```

声明constexpr时用到的类型必须时字面值类型。

自定义类，string类型不属于字面值类型。算术类型、引用和指针都属于字面值类型。一个constexpr指针的初始值必须是nullptr或者0，或者时存储在某个固定地址中的对象。

**对于constexpr指针，限定符constexpr仅对指针有效，对指针所指的对象无关**

```c++
const int *p = nullptr; // 指向常量的指针
constexpr int *q = nullptr; //指向整数的指针常量
```

### 2.5 处理类型

#### 2.5.1 类型别名

```c++
typedef double wages; //wages是double的同义词
typedef wages base, *p; //base是wages的同义词，p是double *的同义词
```

**别名声明**

```c++
using db = double;//db是double的同义词
```

```c++
typedef char *pstring;
const pstring cstr = 0;//cstr是指向char的指针常量
const char *cstr = 0;//是对const pstring cstr的错误理解
```

> **这种理解是错误的！声明语句中用到pstring时，其基本数据类型是指针。但是用char * 重写声明语句后，数据类型就变成了char，* 成为了声明符的一部分。这样改写的结果是，const char成了基本数据类型。前者声明了一个指向char的指针常量，改写后则声明了一个指向const char 的指针**

 #### 2.5.2 auto类型说明符

使用引用其实是使用引用的对象，特别是当引用被当作初始值时，真正参与初始化的其实是引用对象的值，且以引用对象的类型作为auto的类型。

auto一般会忽略掉顶层const，同时会保留底层const。**设置一个类型为auto的引用时，初始值中的顶层常量属性仍然保留，因为此时的常量就不是顶层常量了，如g**

```c++
int i = 0;
const int ci = i, &cr = ci;
auto c = cr;//c是一个整数（ci本身是一个顶层const）
auto d = &i;//d是一个整型指针
auto e = &ci//e是一个指向整数常量的指针（对常量对象取地址是一种底层const）
    
//将auto类型设置为顶层const
const auto f = ci;//ci的推演类型是int，f是const int

//引用的类型为auto，原来的初始化规则仍然适用
auto &g = ci;//g是一个整型常量引用
auto &h = 42;//错误，不能为非常引用绑定字面值
const auto &j = 42;//正确，可以为常量引用绑定字面值
```

#### 2.5.3 decltype类型指示符

**decltype**选择并返回操作数的数据类型，在此过程中，编译器分析表达式并得到它的类型，却不实际计算表达式的值

```c++
decltype(f()) a;//a的类型就是函数f的返回类型
```

若decltype 使用的表达式是一个变量，则返回该变量的类型（包括顶层const和引用在内）

```c++
const int ci = 0, &cj = ci;
decltype(ci) x = 0; //const int
decltype(cj) y = x; //const int &
```

*需要指出的是，引用从来都作为其所指对象的同义词出现，只有用在decltype处是一个例外*

如果decltype使用的表达式不是一个变量，则decltype返回表达式结果对应的类型

```c++
int i = 42, *p = &i, &r = i;
decltype(r + 0) b;//b是一个未初始化的int
decltype(*p) c;//错误，c是int &，必须初始化
```

因为 r 是一个引用，因此decltype(r)的结果是引用类型。将 r 作为表达式的一部分，如 r + 0，显然这个表达式的结果是一个叫具体值而非一个引用。

**如果表达式的内容是解引用操作，则decltype将得到引用类型**，因为解引用指针可以得到指针所指的对象，而且还能给这个对象赋值。所以decltype(*p) 的结果类型是int &，不是int。

相比起auto，decltype的结果类型与表达式形式密切相关。对于decltype所用的表达式来说，如果变量名加上了一对括号，则得到的类型与不加括号时会有不同。如果decltype使用的是一个不加括号的变量，则得到的结果就是该变量的类型；如果给变量加上了一层或多层括号，编译器就会把它当成一个表达式。变量是一种可以作为赋值语句左值的特殊表达式，所以的decltype就会得到引用类型：

```c++
//decltype((variable))的结果永远是引用
//decltype(variable)的结果只有当variable本身就是引用时才是引用
decltype((i)) d;//错误，d为int &,必须初始化
decltype(i) e;//正确，e为未初始化的int
```

#### 2.6.3 编写自己的头文件

预处理器 #include

头文件保护符

**#define**指令把一个名字设定为预处理变量，另外两个指令则分别检查某个指定的预处理变量是否已经定义

**#ifdef**当且仅当变量已定义时为真

**#ifndef**当且仅当变量未定义时为真

一旦检查结果为真，则执行后续操作直到遇到**#endif**指令为止

> 预处理变量无视C++语言中关于作用域的规则



## 3 字符串、向量和数组

### 3.1 命名空间的using声明

```c++
 using namespace::name;//using声明
```

**头文件不应包含using声明**

> 这是因为头文件的内容会拷贝到所引用它的文件中去，如果头文件里有某个using声明，那么每个使用了该头文件的文件就都会有这个声明。对于某些程序，可能会产生名字冲突。

### 3.2 标准库类型string

#### 3.2.1 定义和初始化string对象

初始化string对象的方式

```c++
string s1;//初始化空串
string s2(s1);//s2是s1的副本
string s2 = s1;//同上
string s3("value");//s3是字面值"value"的副本
string s3 = "value";//同上
string s4(n, 'c');//初始化为n个字符c组成的串
```



|                | string的操作                                    |
| -------------- | ----------------------------------------------- |
| os << s        | 将s写到输出流os，返回os                         |
| is >> s        | 从is中读取字符串赋给s，字符串以空白分隔，返回is |
| getline(is, s) | 从is中读取一行赋给s，返回is                     |
| s.empty()      | s为空返回true，否则返回false                    |
| s.size()       | 返回s中字符的个数                               |
| s[n]           | 返回s中第n个字符的引用，位置从0开始             |
| s1 + s2        | 返回s1和s2连接后的结果                          |
| s1 = s2        | 用s2的副本代替s1中原来的字符                    |
| s1 == s2       | 判断s1的字符是否跟s2完全相同，大小写敏感        |
| s1 != s2       | 判断s2的字符是否跟s2不同，大小写敏感            |
| <, <=, >, >=   | 按字典序比较，大小写敏感                        |



```c++
int main() {
    string s1, s2;
    cin >> s1 >> s2;
    cout << s1 << s2;
    return 0;
}
/*
输入”     Hello World!   “
输出”HelloWorld!“
忽略开头的空白，读取第一个真正的字符，直到下一处空白为止
*/
```

```c++
/*
读取的时候遇到换行符为止（换行符也被读进来）
然后把所读内容存入到string对象（不存换行符），换行符被丢弃
*/
int main() {
    string line;
    while(getline(cin, line)) {
        cout << line << endl;
    }
    return 0;
}
```

```c++
string s = string() + " Hello " + "," + " World! ";
//string()放在最后面会产生错误，因为字符串与string是不同的类型，没有加法运算
```

#### 3.2.3 处理string对象中的字符

|             | cctype头文件中的函数                             |
| ----------- | ------------------------------------------------ |
| isalnum(c)  | 当c是字母或数字时为真                            |
| isalpha(c)  | 当c时字母时为真                                  |
| iscntrl(c)  | 当c是控制字符时为真                              |
| isdigit(c)  | 当c是数字时为真                                  |
| isgraph(c)  | 当c不是空格但可打印时为真                        |
| islower(c)  | 当c时小写字母时为真                              |
| isprint(c)  | 当c是可打印字符时为真（即c是空格或具有可视形式） |
| ispunct(c)  | 当c是标点符号时为真                              |
| isspace(c)  | 当c是空白时为真                                  |
| isupper(c)  | 当c是大写字母时为真                              |
| isxdigit(c) | 当c是十六进制数字时为真                          |
| tolower(c)  | 输出c的小写字母                                  |
| toupper(c)  | 输出c的大写字母                                  |

基于范围的for

```c++
for(declaration : expression) 
    statement
```

```c++
int main() {
    string s("some thing");
    decltype(s.size()) punct_cnt = 0;//string::size_type
    for(auto c : s) {
        if (ispunct(c)) {
            punct_cnt++;
        }
    }
    return 0;
}
```

### 3.3 标准库类型vector

#### 3.3.1 定义和初始化vector对象

```c++
vector<T> v1;//空vector，元素为T类型
vector<T> v2(v1);//包含v1所有元素的副本
vector<T> v2 = v1;//同上
vector<T> v3(n, val);//包含n个值为val的元素
vector<T> v4(n);//包含n个重复地执行了值初始化的对象
vector<T> v5{a, b, c...};//列表初始化
vector<T> v5 = {a, b, c...};//同上
```

*一般来说，使用圆括号是用来构造vector对象的，花括号可以表述成列表初始化该vector对象。也就是说，初始化过程会尽可能地把花括号内的值当成元素初始值的列表来处理，只有在无法执行列表初始化时才会考虑其他初始化方式*

```c++
vector<string> v7{10};//10个默认初始化的元素
vector<string> v8{10, "hi"};//10个值为“hi”的元素
```

确认无法执行列表初始化后，编译器会尝试用默认值初始化vector对象

#### 3.3.2 向vector对象中添加元素

**范围for语句体内不应改变其所遍历序列的大小**

**vector对象（以及string对象）的下标运算符可用于访问已存在的元素，而不能用于添加元素**

> **这种通过下标访问不存在的元素的行为非常常见，而且会产生很严重的后果。所谓的缓冲区溢出指的就是这类错误**

### 3.4 迭代器介绍

```c++
vector<int> v;
vector<int>::iterator it;//能读写
vector<int>::const_iterator it;//只读
v.cbegin();
v.cend();
//箭头运算符，把解引用和成员访问两个操作结合在一起
it -> empty()//同 (*it).empty()
```

**但凡是使用了迭代器的循环体，都不要向迭代器所属的容器添加元素**

```c++
//迭代器支持的运算
iter + n;
iter - n;
iter += n;
iter -= n;
iter1 - iter2;//之间的距离,type:difference_type
>, >=, <, <=;//得到距离关系
```

### 3.5 数组

*定义数组的时候必须指定数组的类型，不允许用auto关键字由初始值列表推断类型。另外跟vector一样，数组的元素应为对象，因此不存在引用的数组*

```c++
int *ptrs[10];//含有10个整型指针的数组
int (*Parray)[10];//Parray指向一个含有10个整数的数组
int (&arrRef)[10] = arr;//arrRef引用一个含有10个整数的数组
```

对ptrs来说，从右往左理解：首先定义的是一个大小为10的数组，它的名字是ptrs，然后知道数组中存放的是指向int的指针。

对于Parray，从右往左理解不太合理。对于数组，由内向外阅读比从右往左好多了。首先是圆括号括起来的部分，*Parray意味着Parray是一个指针，观察右边可以直到Parray是一个指向大小为10的数组的指针，最后观察左边，直到数组中的元素是int。最终含义为Parray是一个指针，指向一个int数组，数组中包含10个元素。同理，（&arrRef）表示arrRef是一个引用，它引用的对象是一个大小为10的数组，数组中元素的类型是int

```c++
int *(&arry)[10] = ptrs;//arry是数组的引用，该数组含有10个指针
```

由内向外，首先知道arry是一个引用，然后观察右边直到arry引用的对象是一个大小为10的数组，最后观察左边知道，数组的元素类型是指向int的指针。

> 数组下标定义为i额size_t类型，是一种机器相关的无符号类型。它被设计得足够大以便能表示内存中任意对象的大小。在cstddef头文件中定义了size_t类型

```c++
int ia[] = {0, 1, 2, 3, 4};

auto ia2(ia);//ia2是一个整型指针，指向ia的第一个元素
/*
尽管ia是由10个整数构成的数组，但当使用ia作为初始值时，编译器实际执行的初始化过程类似于下面的形式：
*/
auto ia2(&ia[0]);//显然ia2的类型是 int *
/*
必须指出的是，当使用decltype关键字时上述转换不会发生，decltype(ia)返回的类型是由5个整数构成的数组
*/
//ia3是一个含有10个整数的数组
decltype(ia) ia3 = {0, 1, 2, 3, 4};
```

**指针也是迭代器**

```c++
int arr[] = {0, 1, 2, 3};
int *p = arr; //p指向arr的第一元素
++p;//p指向arr[1]

//头文件iterator
int *beg = begin(arr);//指向arr首元素的指针
int *last = end(ia);//指向arr尾元素的下一位置的指针

//两指针相减的结果的类型是一种名为ptrdiff_td标准库类型，和size_t一样,ptrdiff_t是一种带符号的，定义在cstddef头文件中的机器相关的类型
```

**内置的下标运算符所用的索引值不是无符号类型**

#### 3.5.5 与旧代码的接口

```c++
string s = "string";
auto *p = s.c_str();//返回C风格的字符串
```

**使用数组初始化vector对象**

```c++
int int_arr[] = {0, 1, 2, 3, 4};
vector<int> ivec(begin(int_arr), end(int_arr));
vector<int> subivec(int_arr + 1, int_arr + 4);
```

### 3.6 多维数组

**严格来说，C++语言中没有多维数组，所谓的多维数组其实就是数组的数组**

```c++
//遍历数组
int arr[10][20], cnt = 0;
for(auto &i : arr) {
	for(auto &j : i) {
        j = cnt++;
	}
}
cout << arr[0][10] << endl;
```

外层循环选用引用类型。虽然循环中没有任何写操作，但是使用引用类型是为了避免数组被自动转成指针。假设不用引用类型，那么程序将无法通过编译。因为 i 不是引用类型，所以编译器初始化 i 时会自动将这些数组形式的元素（和其他类型的数组一样）转换成指向该数组内首元素的指针。这样的到的 i 的类型是 int *，显然内层的循环就不合法了。

**使用范围for语句处理多维数组，除了最内层的循环外，其他所有循环的控制变量都应该是引用类型**

```c++
int arr[3][4] = {};
for(auto &i : arr) {
	for(auto j : i) {
		cout << j << " ";
	}
}
for(auto p = arr; p != end(arr);++p) {
    for(auto q = *p;q != end(*p);++q) {
        cout << *q << " ";
    }
}

//将类型“4个整数组成的数组”命名为int_array
using int_array = int[4]//typedef int int_array[4]
for(int_array *p = arr;p != end(arr);++p) {
	for(int *q = *p;q != end(*p);++q) {
        cout << (*q) << " ";
    }
    cout << endl;
}
```

## 4 表达式

### 4.1 基础

*C++定义了一元运算符和二元运算符，作用于一个运算对象的运算符是一元运算符，如取地址符（&）和解引用符（\*)；作用于两个运算对象的运算符是二元运算符，如相等运算符（==）和乘法运算符（\*）。此外，还有一个作用于三个运算对象的三元运算符。函数调用也是一种特殊的运算符，它对运算对象的数量没有限制*

**左值和右值**

C++表达式不是左值就是右值。左值可以位于赋值语句的左侧，右值则不能

一个左值表达式的求值结果是一个对象或者一个函数，然而以常量对象为代表的某些左值实际上不能作为赋值语句的左侧运算对象。当一个对象被用作右值的时候，用的是对象的值（内容）；当对象被用作左值的时候，用的是对象的身份（在内存中的位置）。

一个重要原则（有一种例外）：在需要右值的地方可以用左值来代替，但是不能把右值当成左值（也就是位置）使用。当一个左值被当作右值使用时，实际使用的是它的内容（值）。

使用关键字decltype的时候，左值和右值也有所不同。如果表达式的求值结果是左值，decltype作用于该表达式（不是变量）得到一个引用类型。比如，假定p的类型是int \*，因为解引用运算符生成左值，所以decltype(\*p)的结果是int &。另一方面，因为取地址运算符生成右值，所以decltype(&p)的结果是int \**，即结果是是一个指向整型指针的指针。

### 4.2 算术运算符

算术运算符的运算对象和求值结果都是右值

布尔值不应该参与运算

```c++
bool b = true;
bool a = -b;//true,-b为-1，转换为bool为true
```

在除法运算中，如果两个运算对象的符号相同则商为正（如果不为0的话），否则商为负。C++11标准规定商一律向0取整。

### 4.3 逻辑和关系运算符

逻辑和关系运算符，运算对象和求值结果都是右值

```c++
if (val == true) {}//当且仅当val为1时条件才为真
//true被转换为int型的1
```

> **除非必须，否则不用递增减运算符的后置版本**
>
> 前置版本的递增运算符避免了不必要的工作，它把值加1后直接返回改变了的运算对象。与之相比，后置版本需要将原始值存储下来以便于返回这个未修改的内容。如果不需要修改前的值，那么后置版本的操作就是一种浪费。
>
> 对于相对复杂的迭代器类型，这种额外的工作就消耗巨大了。

```c++
//后置运算符的优先级高于解引用运算符
auto pbeg = v.begin();
while(pbeg != v.end() && *pbeg >= 0)
    cout << *pbeg++ << endl;//等同*(pbeg++)
```

### 4.9 sizeof运算符

```c++
//无须提供一个具体的对象
//对引用类型执行sizeof运算得到被引用对象所占空间的大小
//对指针执行sizeof运算得到指针本身所占空间的大小
//对解引用指针执行sizeof运算得到指针所指的对象所占空间的大小，指针不需有效
//对数组得到整个数组所占空间的大小，sizeof运算不会把数组转换成指针来处理
//对string对象或vector对象执行sizeof运算只返回该类型固定部分的大小，不会计算对象中的元素占用了多少空间
sizeof(type);
sizeof expr;

sizeof *p;//p所指类型的空间大小
constexpr size_t sz = sizeof(arr)/sizeof(*arr)//数组元素的大小
```

### 4.10 逗号运算符

对于逗号运算符来说，首先对左侧的表达式求值，然后将求值结果丢弃掉。逗号运算符真正的结果是右侧表达式的值。如果右侧运算对象是左值，那么最终的求值结果也是左值。

### 4.11 类型转换

数组转换成指针。当数组被用作decltype关键字的参数，或者作为取地址符（&）、sizeof及typeid等运算符的运算对象时，转换不会发生。用引用来初始化数组，转换也不会发生。

#### 4.11.3 显式转换

**命名的强制类型转换**

```c++
cast_name<type>(expression);
//cast-name是static_cast, dynamic_cast, const_cast, reinterpret_cast中的一种
```

**static_cast**

任何具有明确定义的类型转换，只要不包含底层const，都可以使用static_cast

```c++
int j = 0, *p;
double slope = static_cast<double>(j);
double *dp = static_cast<double *>(p);
```

**const_cast**

const_cast只能改变运算对象的底层const

```c++
const char *pc;
char *p = const_cast<char *>(pc);//正确，但是通过p写值是未定义的行为
//如果对象本身不是一个常量，使用强制类型转换获得写权限是合法的行为
//如果对象本身不是一个常量，使用const_cast执行写操作就会产生未定义的后果
```

> 只有const_cast能改变表达式的常量属性，使用其他形式的命名强制类型转换改变表达式的常量属性都将引发编译器错误。同样的，也不能用const_cast改变表达式的类型

```c++
const char *cp;
const_cast<string>(cp);//错误，const_cast只改变常量属性
```

**reinterpret_cast**

reinterpret_cast通常为运算对象的位模式提供较低层次上的重新解释。可用于位长度相同的类型转换，当位长度不同时会引发糟糕的后果。

## 5 语句

#### 5.3.2 switch语句

switch语句先对括号内的表达式求值，表达式的值转换成整数类型，然后与每个case标签的值比较

**case标签必须是整型常量表达式**

```c++
// 统计元音出现次数
switch (ch) {
    case 'a':case 'e': case 'i':case 'o':case 'u':
		cnt++;break;
	default:;
}
```

#### 5.6.1 throw表达式

程序的异常检测部分使用throw表达式引发一个异常

头文件stdexcept

```c++
throw runtime_error("runtime error")
```

#### 5.6.2 try语句块

```c++
try {
    program-statements
} catch (exception-declaration) {
    handler-statements
} catch (exception-declaration) {
    handler-statement
}//...
```

#### 5.6.3 标准异常

<stdexcept>定义的异常类

| exception        | 最常见的问题，只报告异常的发生，不提供额外信息 |
| ---------------- | ---------------------------------------------- |
| runtime_error    | 只有在运行时才能检测出的问题                   |
| range_error      | 运行时错误：生成的结果超出了有意义的值域范围   |
| overflow_error   | 运行时错误：计算上溢                           |
| underflow_error  | 运行时错误：计算下溢                           |
| logic_error      | 程序逻辑错误                                   |
| domain_error     | 逻辑错误：参数对应的结果值不存在               |
| invalid_argument | 逻辑错误：无效参数                             |
| length_error     | 逻辑错误：试图创建一个超出该类型最大长度的对象 |
| out_of_range     | 逻辑错误：使用一个超出有效范围的值             |

## 6 函数

#### 6.1.2 函数声明

在函数的声明中经常省略形参的名字，但是写上形参的名字有助于使用者更好地理解函数的功能。

函数的三要素（返回类型、函数名、形参类型）描述了函数的接口，说明了调用该函数所需的全部信息。函数声明也称作函数原型。

### 6.2 参数传递

#### 6.2.2 传引用参数

> 如果函数无需改变引用形参的值，最好将其声明为常量引用

因为常量不能赋值给非常量引用，所以使用非常量引用的话，能传递的实参类型会受到限制

#### 6.2.5 main:处理命令行选项

```c++
int main(int arge, char *argv[]){}
//第一个形参argc表示数组中字符串的数量，第二个形参是数组
//当实参传给main函数后，argv的第一个元素指向程序的名字
//接下来的元素依次传递命令行提供的实参，最后一个指针之后的元素值保证为0
```

#### 6.2.6 含有可变形参的函数

**initializer_list形参**

如果函数的实参类型未知但是全部实参的类型都相同，可以使用 initializer_list 类型的形参。其类型定义在同名的头文件中，提供的操作如下表

| initializer_list<T> lst;             | 默认初始化；T类型元素的空列表                                |
| ------------------------------------ | ------------------------------------------------------------ |
| initializer_list<T> lst{a, b, c...}; | lst的元素数量跟初始值一样多；lst的元素是对应初始值的副本；列表中的元素是const |
| lst2(lst)                            | 拷贝或赋值一个initialize_list对象不会拷贝列表中的元素；拷贝后，原始列表和副本共享元素 |
| lst2 = lst                           | 同上                                                         |
| lst.size()                           | 列表中的元素数量                                             |
| lst.begin()                          | 返回指向lst中首元素的指针                                    |
| lst.end()                            | 返回指向lst中尾元素下一个位置的指针                          |

initializer_list对象中的元素永远是常量值，无法改变

```c++
//使用如下形式编写输出错误信息的函数，使其可以作用于可变数量的实参
void error_msg(initializer_list<string>(i1)) {
    for(const auto & beg : i1)
        cout << beg << " ";
    cout << endl;
}

int main() {
    string what, hey;
    error_msg({"aaaa", what, hey});
    return 0;
}
```

**省略符形参**

省略符形参是为了便于c++程序访问某些特殊的C代码而设置的，这些代码使用了名为varargs的C标准库功能。通常，省略符形参不应用于其他目的。C编辑器文档会描述如何使用varargs

> 省略符形参应该仅仅用于C/C++通用的类型。应该特别注意的是，大多数类类型的对象在传递给省略符形参时都无法正确拷贝

```c++
void foo(parm_list, ...);
void foo(...);
```

省略符形参只能出现在形参列表的最后一个位置。省略符形参对应的实参无须类型检查。在第一种形式中，形参声明后面的逗号是可选的。

**不要返回局部对象的引用或指针**

**引用返回左值**

```c++
char &get_val(string &str, string::size_type ix) {
    return str[ix];
}

int main() {
    string s("a value");
    get_val(s, 0) = 'A';//将s[0]的值改为A
    cout << s << endl;
    return 0;
}
```

> main函数不能调用它自己

#### 6.3.3 返回数组指针

```c++
typedef int arrT[10];
using arrT = int[10];
arrT *func(int i);//返回一个指向含有10个整数的数组的指针

//返回数组指针的函数形式如下
Type (*function(parameter_list))[dimension];
int (*func(int i))[10]//表示解引用func的调用将得到一个大小是10的int型数组
```

**尾置返回类型**

函数真正的返回类型跟在形参列表之后，在本应该出现返回类型的地方放置auto

```c++
auto func(int i) -> int(*)[10];
```

**使用decltype**

```c++
int odd[5] = {0};
decltype(odd) *arrPtr(int i) {
    return &odd;
}//返回一个指向含有5个整数的数组的指针，decltype的结果是数组
```

### 6.4 函数重载

> 如果同一作用域内的几个函数名字相同但形参列表不同，称之为重载函数
>
> main函数不能重载
>
> 不允许两个函数形参列表一样但是返回类型不同
>
> 顶层const不影响传入函数的对象。一个拥有顶层const的形参无法和另一个没有顶层const的形参区分开来。底层const可以区分，比如指向常量的指针或者引用

局部变量不能作为默认实参

```c++
int a = 10;
void f(int p = a);
void foo() {
    int a = 20;
    f()//此时a = 10，局部变量的a跟传递给f的默认实参没有关系
}
```

**constexpr函数通常被隐式指定为内联函数**

**内联函数和constexpr函数通常定义在头文件内**

#### 6.5.3 调试帮助

**assert预处理宏**。所谓预处理宏其实是一个预处理变量，行为有点类似于内联函数。在cassert头文件中

```c++
assert(expr);
```

首先对expr求值，如果表达式为假，assert输出信息并终止程序的执行。如果表达式为真，assert什么也不做

**NDEBUG预处理变量**。assert的行为依赖于一个名为NDEBUG的预处理变量的状态，如果定义了NDEBUG，则assert什么也不做。默认状态下没有定义NDEBUG，此时assert将执行运行时检查

除了用于assert外，也可以使用NDEBUG编写自己的条件调试代码。

```c++
//如果NDEBUG未定义，将执行#ifndef和#endif之间的代码，如果定义了NDEBUG，这些代码将被忽略掉

#ifndef NDEBUG
cerr << __func__ << ": array size is " << size << endl;
#endif
/*
__func__ 输出当前调试的函数的名字
__FILE__ 存放文件名的字符串字面值
__LINE__ 存放当前行号的整型字面值
__TIME__ 存放文件编译时间的字符串字面值
__DATE__ 存放文件编译日期的字符串字面值
```

### 6.7 函数指针

```c++
bool lengthCompare(const string &, const string &){...};
bool (*pf)(const string &, const string &);//pf指向一个函数，该函数的参数是两个const string的引用，返回值是bool类型

//使用函数指针
pf = lengthCompare;//pf指向名为lengthCompare的函数
pf = &lengthCompare;//同上
pf("hello", "goodbye");//等同调用lengthCompare函数
(*pf)("hello", "goodbye");//同上

//形参看起来是函数类型，实际上却是当成指针使用
void useBigger(const string &s1, const string &s2, bool (*pf)(const string &, const string &)){}

//使用decltype简化代码，需要注意的是decltype返回函数类型，不会将函数类型自动转换成指针类型
int mulf(int a, int b) {
    return a*b;
}
int solve(decltype(mulf) pf, int a, int b) {//和数组类似，不能定义函数类型的形参，但是形参被转换成指向函数的指针
    return pf(a, b);
}

//返回指向函数的指针
using F = int(int *, int);//函数类型
using PF = int (*)(int *, int);//指针类型
int (*f1(int))(int *, int);//由内向外看，f1有形参列表，f1是函数，f1前面有*，
//所以f1返回一个指针，然后发现指针类型本身也包含形参列表，所以指针指向函数
//该函数的返回类型是int
auto f1(int) -> int (*)(int *, int);//同上
```



## 7 类

### 7.1 定义抽象数据类型

```c++
struct Sales_data {
    std::string isbn() const {return bookNo;}
    Sales_data &combine(const Sales_data &);
    double avg_price() const;
    std::string bookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};
//非成员接口函数
Sales_data add(const Sales_data &, const Sales_data &);
std::ostream &print(std::ostream &, const Sales_data &);
std::istream &read(std::ostream &, Sales_data &);
```

**引入this**

> 成员函数通过一个名为this的额外的隐式参数访问调用它的那个对象。当我们调用一个成员函数时，用请求该函数的对象地址初始化this，例如调用total.isbn()
>
> 编译器负责把total的地址传递给isbn的隐式形参this，可以等价认为编译器将该调用重写成
>
> Sale_data::isbn(&total)//伪代码

**引入const成员函数**

> isbn函数中，在参数列表之后紧随const关键字，这里const的作用是修改隐式this指针的类型
>
> 默认情况下，this的类型是指向类类型非常量版本的指针常量。例如在Sales_data成员函数中，this 的类型是Sales_data *const。所以this不能绑定到一个常量对象，即不能在常量对象上调用普通的成员函数
>
> 紧跟在参数列表后面的const表示this是一个指向常量的指针。像这样使用const的成员函数被称作**常量成员函数**

```c++
Sales_data& Sales_data::combine(const Sales_data &rhs) {
    units_sold += rhs.units_sold;
    revenue += rhs.revenue;
    return *this;
}//返回调用该函数的对象
//内置的赋值运算把它的左侧运算对象当成左值返回，为了保持一致，
//combine函数必须返回引用类型
```

#### 7.1.3 定义类相关的非成员函数

> 一般来说，如果非成员函数是类接口的组成部分，则这些函数的声明应该与类在同一个头文件内

```c++
istream &read(istream &is, Sales_data &item) {
    double price = 0;
    is >> item.bookNo >> item.units_sold >> price;
    item.revenue = price * item.units_sold;
    return is;
}

ostream &print(ostream &os, const Sales_data &item) {
    os << item.isbn() << " " << item.units_sold << " "
       << item.revenue << " " << item.avg_price();
    return os;
}//IO类属于不能被拷贝的类型，只能通过引用来传递
```

#### 7.1.4 构造函数

> 构造函数的名字和类名相同。和其他函数不一样的是，构造函数没有返回类型；除此之外类似于其他的函数，构造函数也有一个（可能为空）的参数列表和一个（可能为空）的函数体。类可以包含多个构造函数，和其他重载函数差不多，不同的构造函数之间必须在参数数量或参数类型上有所区别。
>
> 不同于其他成员函数，构造函数不能被声明成const的。当创建类的一个const对象时，直到构造函数完成初始化过程，对象才能真正取得其“常量”属性。因此，构造函数在const对象的构造过程中可以向其写值。

**合成的默认构造函数**

> 类通过一个特殊的构造函数来控制默认初始化过程，这个函数叫做**默认构造函数**。默认构造函数无须任何实参。默认构造函数其中一个特殊性是，如果类没有显式地定义构造函数，那么编译器就会隐式定义一个默认构造函数。
>
> 编译器创建的构造函数又被称为合成的默认构造函数

**某些类不能依赖于合成的默认构造函数**

> 只有当类没有声明任何构造函数时，编译器才会自动生成默认构造函数
>
> 如果类包含有内置类型或者复合类型的成员，默认构造函数可能使之值为未定义的（指针）
>
> 有的编译器不能为某些类合成默认的构造函数

```c++
struct Sales_data {
    Sales_data() = default;//构造函数
}
```

**=default的含义**

> 因为该构造函数不接受任何实参，所以它是一个默认构造函数。定义这个构造函数的目的仅仅是因为我们继续要其他形式的构造函数，也需要默认的构造函数
>
> 在参数列表后写上**=default**来要求编译器生成构造函数。和其他函数一样，如果=default在类的内部，则默认构造函数时内联的；如果在类的外部，则该成员默认情况下不是内联

**构造函数初始化列表**

```c++
//构造函数初始化列表
Sale_data(const std::string &s): bookNo(s) {}
Sale_data(const std::stirng &s, unsigned n, double p): bookNo(s), units_sold(n), revenue(p*n) {}
```

#### 7.1.5 拷贝、赋值和析构

*如果不主动定义拷贝、赋值、析构操作，则编译器会替我们合成他们*

**某些类不能依赖于合成的版本**，管理动态内存的类通常不能依赖于上述操作的合成版本（如果类包含vector/string成员，则其拷贝、赋值和销毁的合成版本应该能够正常工作）

### 7.2 访问控制与封装

> class关键字与struct关键字的唯一区别是默认的访问权限。struct关键字，定义在第一个访问说明符之前的成员是public，class为private

#### 7.2.1 友元

类可以允许其他类或者函数访问它的非公有成员，方法是令其他类或者函数成为它的友元

```c++
class Sales_data {
    friend Sales_data add(const Sales_data &, const Sales_data &);
    friend std::ostream &print(std::ostream &, const Sales_data &);
    friend std::istream &read(std::istream &, Sales_data &);
public:
    Sales_data() = default;
    Sales_data(const std::string &s, unsigned n, double p):
                bookNo(s), units_sold(n), revenue(p * n){ }
    Sales_data(const std::string &s): bookNo(s) {}
    Sales_data(std::istream &);
    std::string isbn() const {return bookNo;}
    Sales_data &combine(const Sales_data &);
private:
    std::string bookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};
Sales_data add(const Sales_data &, const Sales_data &);
std::ostream &print(std::ostream &, const Sales_data &);
std::istream &read(std::istream &, Sales_data &);
```

**友元的声明**

友元的声明仅仅指定了访问的权限，而非一个通常意义上的函数声明

通常把友元的声明与类本身放置在同一个头文件中（类的外部）

### 7.3 类的其他特性

#### 7.3.1 类成员再探

````c++
class Screen {
public:
    //隐藏使用string对象的细节
    //同 using pos = std::string::size_type
    //先定义后使用，通常在类开始的地方
    typedef std::string::size_type pos;
private:
    pos cursor = 0;
    pos hight = 0, width = 0;
    std::string contents;
};
````

**Screen类的成员函数**

```c++
class Screen {
public:
    typedef std::string::size_type pos;
    Screen() = default;//因为有Screen另一个构造函数，而且需要合成默认构造函数
    //所以本函数是必须的
    Screen(pos ht, pos wd, char c): height(ht), width(wd), contents(ht * wd, c) {}
    //cursor成员隐式使用了类内初始值
    char get() const {//读取光标处的字符
        return contents[cursor];//隐式内联
    }
    inline char get(pos ht, pos wd) const;//显式内联
    Screen &move(pos r, pos c);//能在之后被设为内联
private:
    pos cursor = 0;
    pos height = 0, width = 0;
    std::string contents;
};
```

可以在类的外部用inline关键字修饰函数的定义：

```c++
inline //在函数的定义处指定inline
Screen &Screen::move(pos r, pos c) {
    pos row = r * width;//计算行的位置
    cursor = row + c;//在行内将光标移动到指定的列
    return *this;//以左值的形式返回对象
};
//在类的内部声明成inline
//最好在类外也说明inline，使类容易理解
char Screen::get(pos r, pos c) const {
    pos row = r * width;//计算行的位置
    return contents[row + c];//返回给定列的字符
}
```

**可变数据成员**

需要修改类的某个成员，即使是在一个const成员函数内。可以通过声明符mutable关键字做到

一个可变数据成员永远不会是const，即使它是const对象的成员，因此一个const成员函数可以改变一个可变成员的值。

```c++
class Screen {
public:
	void some_member() const;
private:
    mutable size_t access_ctr;
}
void Screen::some_member() const {
    //尽管some_member为const成员函数，access_ctr的值也能改变
    ++access_ctr;//记录成员函数被调用的次数
}
```

**类数据成员的初始值**

```c++
class Window_mgr {
private:
    //默认情况下，一个Window_mgr包含一个标准尺寸的空白Screen
    std::vector<Screen> screen{Screen(24, 80, ' ')};
}
```

#### 7.3.2 返回*this的成员函数

```c++
class Screen {
public:
    Screen &set(char);
    Screen &set(pos, pos, char);
}
inline Screen &Screen::set(char c) {
    contents[cursor] = c;//设置当前光标所在位置的新值
    return *this;
};
inline Screen &Screen::set(pos r, pos col, char ch) {
    contents[r*width + col] = ch;
    return *this;//将this作为左值返回
}
```

**基于const的重载**

```c++
class Screen {
publc:
    Screen &display(std::ostream &os) {
        do_display(os);return *this;
    }
    const Screen &display(std::ostream &os) const {
        do_display(os);return *this;
    }
private://显示内容
    void do_display(std::ostream &os) const {os << contents;}
    //隐式被声明成内联函数，不会产生额外的开销
}
```

#### 7.3.3 类类型

```c++
Sales_data item1;
class Sales_data item1;//同上，继承于C
```

**类的声明**

```c++
class Screen;//先声明类且不暂定义它，前向声明
```

> 在声明之后定义之前是一个不完全类型
> 对于不完全类型，可以定义指向这种类型的指针或引用，也可以声明（但不能定义）以不完全类型作为参数或者返回类型的函数
> 先声明后定义
>
> 只有当类全部完成后类才算被定义，所以一个类的成员类型不能是类自己。然而，一旦一个类的名字出现后，它就被认为是声明过了（但尚未定义），因此类允许包含指向它自身类型的引用或指针



#### 7.3.4 友元再探

*友元函数能定义在类的内部，这样的函数是隐式内联的*

```c++
class Screan {
    friend class Window_mgr;
}
class Window_mgr {
public:
    using ScreenIndex = std::vector<Screen>::size_type;
    void clear(ScreenIndex);//按编号将指定的Screen重置为空白
private:
    std::vector<Screen> screens{Screen(24, 80, ' ')};
};

void Window_mgr::clear(ScreenIndex i) {
    Screen &s = screens[i];
    s.contents = string(s.height * s.width, ' ');
}
```

**友元函数不存在传递性，每个类负责控制自己的友元类和友元函数**

**在一个作用域内声明友元，并不代表该友元所在的作用域包括该作用域**

### 7.4 类的作用域

当成员函数定义在类的外部时，返回类型中使用的名字都位于类的作用域之外。这时，返回类型必须指明它是哪个类的成员。

```c++
class Window_mar{
public:
    ScreenIndex addScreen(const Screen &);
    //其他成员与之前的版本一致
}
Window_mgr::ScreenIndex//必须明确指定返回类型的作用域
Window_mgr::addScreen(const Screen &s) {
    screens.push_back(s);
    return screens.size() - 1;
}
```

**编译器处理完类中的全部声明后才会处理成员函数的定义**

**类型名要特殊处理**

> 一般内层作用域可以重新定义外层作用域中的名字。然而在类中，如果成员使用了外层作用域中的某个名字，而该名字代表一种类型，则类不能在之后重新定义该名字

```c++
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

### 7.5 构造函数再探

如果成员是const或者是引用的话，必须初始化

```c++
class ConstRef {
public:
    explicit ConstRef(int ii): i(ii), ci(ii), ri(i){}
private:
    int i;
    const int ci;
    int &ri;
};
```

**初始化和取值的区别，除了底层效率问题：前者直接初始化数据成员，后者则先初始化再赋值。更重要的是，一些数据成员必须被初始化**

**构造函数初始值列表只说明初始化成员的值，而不限定初始化的具体执行顺序。初始化的顺序与它们在类定义中的出现顺序一致**

#### 7.5.2 委托构造函数

*一个委托构造函数使用它所属类的其他构造函数执行它自己的初始化过程，或者说它把它自己的一些（或者全部）职责委托给了其他构造函数*

```c++
class Sales_data {
public:
    Sales_data(std::string s, unsigned cnt, double price): 
    bookNo(s), units_sold(cnt), revenue(cnt * price) {}
    //其余构造函数全部委托给另一个构造函数
    Sales_data(): Sales_data("", 0, 0) {}
    Sales_data(std::string s): Sales_data(s, 0, 0) {}
    Sales_data(std::istream &is): Sales_data() {read(is, *this)}//执行代码
    //然后控制权才会交还给委托者的函数体
}
```

#### 7.5.4 隐式的类类型转换

*如果构造函数只接受一个实参，则它实际上定义了转换为此类类型的隐式转换机制，这种构造函数有时称作转换构造函数*

**能通过一个实参调用的构造函数定义一条从构造函数的参数类型向类类型隐式转换的规则**

```c++
string null_book = "9-99-99";
item.combine(null_book);//先调用Sales_data的combine成员
//编译器用给定的string自动创建了一个Sales_data对象
//该临时Sales_data对象被传递给combine
//因为combine的参数是一个常量引用，所以可以给该参数传递一个临时量
```

**只允许一步类类型转换**

```c++
//错误，需要先将"9-99-99"转换成string，再从string转换成Sale_data
item.combine("9-99-99");
```

**类类型转换不是总有效**

```c++
item.combine(cin);//隐式把cin转换成Sales_data，这个转换执行了接受一个istream的Sales_data构造函数
//该构造函数通过读取标准输入创建了一个（临时的）Sales_data对象，随后将得到的对象传递给combine
//Sales_data对象是个临时量，一旦combine完成就不能在访问它了
//实际上，我们是构建了一个对象，先将它的值加到item，随后将其丢弃
```

**显式转换（抑制构造函数定义的隐式转换）**

```c++
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

> 关键字explicit只对一个实参的构造函数有效
>
> 只能在类内声明构造函数时使用explicit关键字
>
> explicit关键字声明构造函数时，它将只能以直接初始化的形式使用，而且不会再自动转换过程中使用该构造函数

#### 7.5.5 聚合类

聚合类使得用户可以直接访问其成员，并且具有特殊的初始化语法形式。当一个类满足如下条件时，我们说它是聚合的：

* 所有成员都是public
* 没有定义任何构造函数
* 没有类内初始值
* 没有基类，也没有virtual函数

```c++
struct Data {
    int ival;
    string s;
};
//提供一个花括号括起来的成员初始化列表，并用它初始化聚合类的数据成员
Data vall = {0, "Anna"};
```

#### 7.5.6 字面值常量

数据成员都是字面值类型的聚合类是字面值常量类。如果一个类不是聚合类，但它符合下述要求，则它也是一个字面常量类:

> 数据成员都必须是字面值类型
>
> 类必须至少含有一个constexpr构造函数
>
> 如果一个数据成员含有类内初始值，则内置类型成员的初始值必须是一条常量表达式；或者如果成员属于某种类类型，则初始值必须使用成员自己的constexpr构造函数
>
> 类必须使用析构函数的默认定义，该成员负责销毁类的对象

**constexpr构造函数**

尽管构造函数不能是const的，但是字面值常量类的构造函数可以是constexpr函数。事实上，一个字面值常量类必须至少提供一个constexpr构造函数

构造函数可以声明成=default的形式（或者是删除函数的形式）。否则，constexpr构造函数就必须既符合构造函数的要求（不能包含返回语句），又符合constexpr函数的要求（能拥有的唯一可执行语句就是返回语句）。即constexpr构造函数体一般来说应该是空的

constexpr构造函数必须初始化所有数据成员

```c++
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

### 7.6 类的静态成员

```c++
class Account {
public:
    void calculate() {amount += amount * interestRate;}
    static double rate() {return interestRate;}
    static void rate(double);
private:
    std::string owner;
    double amount;
    static double interestRate;
    static double initRate();
};//类的静态成员存在于任何对象之外，对象中不包含任何与静态数据成员有关的数据
//静态成员函数也不与任何对象绑定在一起，它们不包含this指针
//作为结果，静态成员函数不能声明成const

//使用作用域运算符直接访问静态成员
Account::rate();
//成员函数不用通过作用域运算符就能直接使用静态成员
class Account {
public:
    void calculate() {amount += amount * interestRate;}
private:
    static double interestRate;
    //其他成员与之前的版本一致
}
```

**定义静态成员**

static关键字只出现在类内部的声明语句

```c++
void Account::rate(double new Rate) {
    interestRate = newRate;
}//必须在类的外部定义和初始化每个静态成员。和其他对象一样，一个静态数据成员只能定义一次
```

**静态成员的类内初始化**

```c++
class Account {
private:
    static constexpr int period = 30;
    double daily_tbl[period];
}//如果preiod的唯一用途就是定义daily_tbl的维度，则不需要在Account外面专门定义period
//但是当需要把Account::period传递给一个接受const int&的函数时，必须定义period
constexpr int Account::period;//不带初始值的静态成员的定义
```

**特殊场景下的静态成员**

```c++
class Bar {
private:
    static Bar mem1;//静态成员可以是不完全类型
    Bar *mem2;//指针成员可以是不完全类型
    Bar mem3;//错误，数据类型必须是完全类型
}
//静态成员可以作为默认实参
```

