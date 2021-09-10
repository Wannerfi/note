[Lua程序设计](http://luabook.yinyst.com/)
lua5.1 及以上版本后，部分语法较以前有变，笔记为5.4.3

## 第1章 起点
全局变量无需声明

无初始化的全局变量为 `nil`

```lua
print(b)
b = nil --- 删除全局变量

--[[
多行注释
--]]
```

## 第2章 类型和值
函数type可以检测类型

所有值都可以作为条件，除了false和nil为假，其他为真（包括0和空串）

使用 `[[...]]` 表示字符串，可包含换行，可嵌套且不会解释转义序列，首字符为换行符会被自动忽略

string 和 number 之间会进行隐式的类型转换。需要比较时最好进行显式转换

```lua
print(10 .. "" == "10") --> true
```

函数是第一类值（同其他变量）

userdata可以将 C 数据存放在 Lua 变量中，除了赋值和相等比较外没有预定义的操作

## 第3章 表达式
不相等 ~=

如果两个值类型不同，则两者不同。nil只和自己相等。

Lua通过引用比较tables、userdata、functions。即当且仅当两者表示同一个对象时相等。

逻辑运算符 `and or not`

只有 false 和 nil 为假，其他为真

and 和 or 的运算结果与两个操作数有关
```lua
print(nil and 13)   --> nil
print(false or 5)   --> 5
(a and b) or c      --> a ? b : c
```

优先级自己打括号

not 的结果只返回 false 或者 true

表的下标从 1 开始
```lua
a = {i = 0, "Hello", s = "string"}
print(a[1]) --> Hello
```
```lua
--- 反序存储标准输入，并输出
list = nil;
cnt = 1;
for line in io.lines() do
    list = {next = list, value = line};
    if(cnt <= 0) then
        break;
    end
    cnt = cnt - 1;
end

while list do
    print(list.value)
    list = list.next
end
```
```lua
--- 一般初始化方式
a = {["+"] = "add", ["-"] = "sub", [1] = "233"}
print(a["+"], a[1]) --> add 233
```

构造列表中的逗号分隔符可用分号替代

## 第4章 基本语法
变量个数多余值的个数时，值按nil补足；变量个数少于值个数时，多余值被忽略
```lua
a, b = 0
print(a, b) --> 0 nil
```

使用local创建局部变量
```lua
a = 1;
b = 2;
c = 1;
do
    local a2 = 2 * a;
    local d = math.sqrt(b ^ 2 - 4 * a * c);
    x1 = (-b + d) / a2
    x2 = (-b - d) / a2
end
print(x1, x2)
```

控制结构语句
```lua
if conditions then
    a = a;
elseif conditions then
    b = b;
else
    c = c;
end

while conditions do
    a = a;
end

repeat
    a = a;
until conditions;

--- 数值for
--- step 参数默认为 1
--- 三个表达式(beginNumber, endNumber, step)只会在循环开始前被计算一次
--- 控制变量 var 为自动被声明的局部变量，只在循环内有效
--- 循环过程中不要改变控制变量 var 的值，行为未定义
for var = beginNumber, endNumber, step do
    loop_part = loop_part;
end

--- 范型for
--- 控制变量为局部变量
--- 不要修改控制变量的值
for k in pairs(t) do
    print(k)
end
```

当一个函数自然结束时，结尾会有一个默认的return

break 和 return 只能出现在 block 的结尾

## 第5章 函数
```lua
function func_name(args)
    statememts = statememts;
end

--- 当 args 只有一个参数，且参数为字符串或者表构造时，()可省略
print "Hello"; --> print("Hello");
```

函数实参和形参的匹配与赋值语句类似，多余值被忽略，缺少值补充nil

多返回值
```lua
function f()
    return 1, 2
end
```

当表达式调用函数时
```lua
--- 仅当函数作为表达式最后一个参数时，函数才会尽可能返回多个值。
--- 其他情况仅返回第一个值
function foo2()
    return 'a', 'b';
end
x, y, z = 10, foo2();
print(x, y, z) --> 10	a	b
x, y, z = foo2(), 10;
print(x, y, z) --> a	10	nil
```

函数调用作为函数参数被调用时，和多值赋值是相同的
```lua
print(foo2(), 1) --> a	1
print(1, foo2()) --> 1	a	b
```

函数调用在表构造函数中初始化时，同多值赋值
```lua
a = {foo2(), 10}
b = {10, foo2()}
for i, val in ipairs(a) do
    print(val)
end
--[[
a
10
--]]

for i, val in ipairs(b) do
    print(val)
end
--[[
10
a
b
--]]
```

使用圆括号强制使调用返回一个值
```lua
print((foo2())) --> a
```

特殊函数unpack，参数为一个数组，返回数组所有元素，被用作实现范型调用机制
```lua
function f(a, b, c)
    print(a, b, c)
end
a = {"hello", "ll"}
f(table.unpack(a)) --> hello	ll	nil

--- 预定义的unpack由C实现，也可用Lua完成
function unpack(t, i)
    i = i or 1;
    if t[i] then
        return t[i], unpack(t, i + 1);
    end
end
```

使用 `...` 表示函数的可变参数，使用slect函数访问包括元素nil
```lua
function p(...)
    for i, v in ipairs({...}) do
        print(v);
    end
    print();
    print("len:", select("#", ...))
    print(select(1, ...));
end
p("hello", "world", nil, "?");
--[[
hello
world

len:	4
hello	world	nil	?
--]]
```

用表作为函数的唯一参数，可通过表索引指定参数

## 第6章 再论函数
函数时带有词法定界的第一类值
> **第一类值指**：函数和其他值（number，string）一样，可存为变量，放在表中，作为形参，作为函数返回值
**词法定界指**：嵌套的函数可以访问他外部函数中的变量

函数定义实际上是一个赋值语句
```lua
--- 这两个函数相同
function foo(x)
    return 2 * x;
end
foo = function (x) return 2 * x end
```

使用table标准库提供的排序函数
```lua
network = {
    {name = "c", IP = "1"},
    {name = "b", IP = "2"},
    {name = "a", IP = "3"},
}
table.sort(network, function (a, b)
    return a.name < b.name
end );
for i, v in ipairs(network) do
    print(v.name, v.IP)
end
--[[
a	3
b	2
c	1
--]]
```

闭包
```lua
--- 匿名函数内部的grades为外部的局部变量，即upvalue
names = {"P", "M", "K"};
grades = {P = 10, M = 7, K = 8};
function sortByGrade (names, grades)
    table.sort(names, function (n1, n2)
        return grades[n1] > grades[n2];
    end)
end

function newCounter()
    local i = 0;
    return function()
        i = i + 1;
        return i;
    end
end
c1 = newCounter();
print(c1()); --> 1
print(c1()); --> 2
c2 = newCounter();
print(c2()); --> 1
--- 匿名函数使用upvalue i保存计数
--- 简单说，闭包是一个函数以及它的upvalues，c1、c2是建立在同一个函数上的两个不同的闭包
--- 技术上讲，闭包指值而不是函数，函数仅仅是闭包的一个原型声明；
--- 在不会混淆的情况下可以用术语函数指代闭包
```

非全局函数
```lua
--- 表和函数
Lib = {}
Lib.foo = function (x, y) return x + y; end
Lib.goo = function (x, y) return x - y; end

Lib = {
    foo = function (x, y) return x + y; end
    goo = function (x, y) return x - y; end
}

function Lib.foo(x, y)
    return x + y;
end
function Lib.goo(x, y)
    return x - y;
end

--- 将函数保存在局部变量里，得到一个局部函数
--- 声明局部变量的两种方式
local f = function (...)
    statements = statements;
end
local function f(n)
    if n == 0 then
        return 1;
    end
    return n * f(n - 1); --- 此f 并非局部函数f，除非先声明，如下
end

local fact
fact = function (n)
    if n == 0 then
        return 1;
    end
    return n * fact(n - 1);
end 
```

当函数最后一个动作是调用另外一个函数时，为尾调用
```lua
function f(x)
    return g(x);
end
--- 正确的尾调用之后，程序不需要在栈中保留关于调用者的任何信息
--- 因此处理尾调用时不适用额外的栈，不会导致栈溢出
function foo(x)
    if(n > 0) then
        return foo(x - 1);
    end
end 
```

## 第7章 迭代器与范型for
迭代器和闭包
```lua
--- 简单迭代器
function list_iter(t)
    local i = 0;
    local n = #t;
    return function ()
        i = i + 1;
        if i <= n then
            return t[i];
        end
    end
end
--- list_iter是一个工厂，每次调用都会创建一个新的闭包。闭包保存内部局部变量(t, i, n)
--- 范型for为迭代循环处理所有的簿记：首先调用迭代工程；内部保留迭代函数；在每一个新的迭代处调用迭代函数；迭代器返回nil时结束循环
t = {10, 20, 30}
for element in list_iter(t) do
    print(element)
end

--- 实现一个迭代器：遍历文件内所有匹配的单词
function allwords()
    local line = io.read();
    local pos = 1;
    return function ()
        while line do
            local s, e = string.find(line, "%w+", pos);
            if s then
                pos = e + 1;
                return string.sub(line, s, e);
            else
                line = io.read();
                pos = 1;
            end
        end
        return nil;
    end
end
```

范型for的语义
```
for <var-list> in <exp-list> do
    <body>
end
```
范型for在自己内部保存三个值：迭代函数、状态常量、控制变量。下面是执行过程
- 初始化，计算in后面表达式的值，表达式应该返回三个值：迭代函数、状态常量、控制变量
- 将状态常量和控制变量作为参数调用迭代函数（对于for结构，状态常量没有用处、仅在初始化时获取他的值并传递给迭代函数）
- 将迭代函数返回的值赋给变量列表
- 若返回的第一个值为nil，循环结束，否则返回循环体
- 回到第二步再次调用迭代函数
```lua
for var_1, ..., var_n in explist do block end
--- 等价于
do
    local _f, _s, _var = explist;
    while true do
        local var_1, ..., var_n = _f(_s, _var);
        _var = var_1;
        if _var == nil then break end
        block
    end
end
```

无状态的迭代器是指不保留任何状态的迭代器。可以因为避免创建闭包花费额外的代价。无状态迭代器的典型例子是ipairs
```lua
--- 自定义ipairs和迭代函数
function iter(a, i)
    i = i + 1;
    local v = a[i];
    if v then
        return i, v;
    end
end
function ipairs(a)
    return iter, a, 0
end
```
Lua库中pairs使用next实现的原始方法
```lua
function pairs(t)
    return next, t, nil
end
--- 等价于
for k, v in next, t do
    print(v)
end
```

多状态的迭代器
迭代器需要保存多个状态信息，最简单的方法是使用闭包，还有一种方法是将所有的状态信息封装到table内，将table作为迭代器的状态常量
```lua
--- 真正的处理工作在迭代函数完成
function iterator(state)
    while state.line do
        local s, e = string.find(state.line, "%w+", state.pos);
        if s then
            state.pos = e + 1;
            return string.sub(state.line, s, e);
        else
            state.line = io.read();
            state.pos = 1;
        end
    end
    return nil
end

function allwords()
    local state = {line = io.read(), pos = 1};
    return iterator, state;
end
```

应该尽可能写无状态的迭代器，循环的时候由for来保存状态，不用额外创建对象。如果不能用无状态的迭代器实现，应该尽量使用闭包，创建闭包的代价和处理速度都比table快

## 第8章 编译·运行·错误信息
Lua会先将代码预编译成中间码然后再执行

load() 中将每个chunk都作为一个匿名函数处理
```lua
local f = function () i = i + 1; end; --- 等同于
local f = load("i = i + 1;");
```

执行load() 之后，chunk 被编译了但还没有被定义，只有运行chunk 才是定义。

函数的定义是发生在运行时的赋值而不是编译时

load编译时不关心词法范围，使用全局变量

require和dofile完成相同的功能但有两点不同
- require 会搜索目录加载文件
- require 会避免加载同一文件

require的路径是一个模式列表，每一个模式指明一个虚文件名转成实文件名的方法。匹配时会将文件名代替 `?`
虚文件名样例 `?;?.lua`
调用 `require(l)` 会尝试打开文件 `l;l.lua`

动态链接库 `package.loadlib(libname, funcname);`

自定义错误 `error("My Error")` 
`assert()` assert 会先处理两个参数，然后才调用函数。第一个参数若没问题，则不做任何事情，第二个参数为错误信息。

异常和错误处理
```lua
--- pcall在保护模式下执行函数内容，同时捕获所有的异常和错误
--- 若一切正常，pcall返回true以及 被执行函数的返回值；否则返回nil和错误信息
--- 错误信息不一定仅为字符串
local status, err = pcall(function () error({code = 121}); end);
print(err.code)
```

pcall 返回错误信息时，已经释放了保存错误发生情况的栈信息。xpcall接受两个参数：调用函数、错误处理函数。xpacll在栈释放以前调用错误处理函数。debug.debug能查看错误发生时的情况，debug.traceback能创建更多的错误信息。