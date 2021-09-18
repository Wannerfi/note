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
函数是带有词法定界的第一类值
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

## 第9章 协同程序
在任一指定时刻只有一个协同程序在运行

协同基础
```lua
--- 创建协同，成功则返回thread类型
co = coroutine.create(function ()
    print("hi");
end )
print(co) --> thread: 0000000000e5ca98

--- 协同有三个状态：挂起态、运行态、停止态
--- 刚创建成功时，为挂起态
print(coroutine.status(co)) --> suspended
--- 由挂起态变为运行态
coroutine.resume(co); --> hi
--- 任务完成，进入停止态
print(coroutine.status(co)) --> dead
--- yield函数将正在运行的代码挂起
co = coroutine.create(function ()
    for i = 1, 2 do
        print("co", i);
        coroutine.yield();
    end
end)
coroutine.resume(co); --> co	1
print(coroutine.status(co)); --> suspended
coroutine.resume(co); --> co	2
print(coroutine.resume(co)); --> true
--- 激活终止态协程会得到错误
print(coroutine.resume(co)); --> false	cannot resume dead coroutine
--- resume运行在保护模式下，如果协程内部存在错误，Lua并不会抛出错误，而是将错误返回给resume函数

--- 通过resume-yield来交换数据,数据由yield传给resume
co = coroutine.create(function (a, b, c)
    print("co", a, b, c);
    coroutine.yield(a + 1, b + 2, c + 3)
end );
res, a, b, c = coroutine.resume(co, 1, 2, 3); --> co	1	2	3
print(a, b, c) --> 2	4	6
--- Lua的协同为不对称协同
```

生产者-消费者
```lua
--- 消费者驱动类型
function producer()
    while true do
        local x = io.read();
        send(x);
    end
end

function consumer()
    while true do
        local x = receive();
        io.write(x, "\n");
    end
end

function receive()
    local status, value = coroutine.resume(producer);
    return value;
end
function send(x)
    coroutine.yield(x);
end
```

用作迭代器的协同
```lua
--- 生成全排列
function permgen(a, n)
    if n == 0 then
        coroutine.yield(a);
    end
    for i = 1, n do
        a[n], a[i] = a[i], a[n]
        permgen(a, n - 1)
        a[n], a[i] = a[i], a[n]
    end
end
--- 定义迭代工厂
function perm(a)
    local n = #a;
    local co = coroutine.create(function () permgen(a, n) end)
    return function ()
        local code, res = coroutine.resume(co);
        return res;
    end
end
for p in perm({"a", "b", "c"}) do
    printResult(p); --- 自行实现打印表
end
```

用轮询和及时中断的方法解决协程的阻塞问题

## 第11章 数据结构
table是Lua中唯一的数据结构

```lua
--- 数组
a = {1, 2, 3};
a = {{1, 1, 1}, {2, 2, 2}};
--- 链表
list = {next = list, value = v};
--- 队列和双向队列
Queue = {}
function Queue.new()
    return {first = 0, last = -1};
end
function Queue.pushLeft(queue, value)
    local first = queue.first - 1;
    queue.first = first;
    queue[first] = value;
end
function Queue.pushRight(queue, value)
    local last = queue.last + 1;
    queue.last = last;
    queue[last] = value;
end
function Queue.popLeft(queue)
    local first = queue.first;
    if first > queue.last then
        error("list is empty");
    end
    local value = queue[first];
    queue[first] = nil;
    queue.first = first + 1;
    return value;
end
function Queue.popRight(queue)
    local last = queue.last;
    if queue.first > queue.last then
        error("list is empty");
    end
    local value = queue[last];
    queue[last] = nil;
    queue.last = last - 1;
    return value;
end
a = Queue.new();
Queue.pushLeft(a, 3)
Queue.pushLeft(a, 4)
for i, val in pairs(a) do
    print(i, val)
end
--[[
last	-1
-1	3
-2	4
first	-2
--]]
--- 集合
function Set(list)
    local set = {};
    for _, val in pairs(list) do
        set[val] = true;
    end
    return set;
end
reserved = Set({"a", "a", "a", "b"})
for _, val in pairs(reserved) do
    print(_);
end
--[[
a
b
--]]
```

table.concat(t, "\n") 以换行符为分隔符连接table中的字符串

## 第12章 数据文件与持久化
以安全的方式引用任意字符串
```lua
function serialize(o)
    if type(o) == "string" then
        io.write(string.format("%q", o));
    end
end
serialize("[]") --> "[]"
```

保存不带循环的table，带循环的递归记录出现过的值即可
```lua
function serialize(o)
    if type(o) == "number" then
        io.write(o);
    elseif type(o) == "string" then
        io.write(string.format("%q", o));
    elseif type(o) == "table" then
        io.write("{\n");
        for k, v in pairs(o) do
            io.write(string.format(" [%q] = ", k));
            serialize(v);
            io.write(",\n");
        end
        io.write("}");
    else
        error("cannot serialize a " .. type(o));
    end
end
serialize({a = 12, b = "Lua", 3, {"b"}})
--[[
{
 [1] = 3,
 [2] = {
 [1] = "b",
},
 ["a"] = 12,
 ["b"] = "Lua",
}
--]]
```

## 第13章 Metatables and Metamethods
Metatables允许改变table的行为。Lua中每个表都可以有Metatable，默认为无
```lua
a = {};
b = {};
setmetatable(a, b); --- b 为 a的metatable
assert(getmetatable(a) == b)
```

表也可以是自身的metatable，用来描述私有行为

算术运算的Metamethods
```lua
--- 集合并
Set = {}
Set.mt = {}
function Set.new(t)
    local set = {}
    setmetatable(set, Set.mt);
    for _, val in pairs(t) do
        set[val] = true;
    end
    return set;
end

function Set.union(a, b)
   local res = Set.new({});
    for k in pairs(a) do res[k] = true; end
    for k in pairs(b) do res[k] = true; end
    return res;
end

function Set.intersection(a, b)
    local res = Set.new({})
    for k in pairs(a) do
        res[k] = b[k];
    end
    return res;
end

function Set.tostring(set)
    local ret = {};
    for k in pairs(set) do
        table.insert(ret, k);
    end
    return "{" .. table.concat(ret, " ") .. "}"
end

function Set.print(s)
    print(Set.tostring(s));
end

a = Set.new({"a", "b", "c", 1});
b = Set.new({1, 2});
print(getmetatable(a), getmetatable(b)) --> table: 0000000000e59060	table: 0000000000e59060

--- 给metatable增加__add函数，并集
Set.mt.__add = Set.union;
Set.print(a + b); --> {1 b c 2 a}
--- 相乘运算定义交集操作
Set.mt.__mul = Set.intersection;
Set.print((a + b) * b); --> {1 2}
--- 其他运算符，__sub(-), __div(/), __unm(-), __pow(^), __concat(连接)

--- 当两个操作数有不同的metatable，按参数先后顺序查找__add域，并将第一个找到的__add域作为metamethod
--- Lua不关心混合类型，无法运算将报错
```

关系运算符的Metamethods
```lua
--- 关系运算符__eq(==), __lt(<), __le(<=)
Set.mt.__le = function (a, b)
    for k in pairs(a) do
        if not b[k] then return false; end
    end
    return true;
end

Set.mt.__lt = function (a, b)
    return a <= b and not (b <= a);
end

Set.mt.__eq = function (a, b)
    return a <= b and b <= a;
end
```

库自定义的Metamethods
```lua
--- print函数总是调用tostring来格式化输出
Set.mt.__tostring = Set.tostring;
print(a); --> {1 a b c}
--- setmetatable/getmetatable函数也会使用metafield，可以设置__metatable的值隐藏metatable
Set.mt.__metatable = "not your business";
print(getmetatable(a)) --> not your business
setmetatable(a, {}); --> error
```

## 第14章 环境
Lua将环境本身存储在一个全局变量_G中，_G._G等于_G。
> Lua5.2开始取消环境表的概念，增加_Env来管理环境

```lua
--- 打印当前环境中所有的全局变量的名字
for n in pairs(_G) do
    print(n)
end
a = 3
print(_G["a"]); --> 3
```

声明全局变量
```lua
--- 改变全局变量的行为，这样任何企图访问一个不存在的全局变量的操作都会引起错误
setmetatable(_G, {
    __newindex = function (_, n)
        error("attempt to write to undeclared variable"..n, 2);
    end,

    __index = function (_, n)
        error("attempt to read undeclared variable"..n, 2);
    end
})

--- 使用rawset 添加变量, 可以绕过metamethod的行为约束
rawset(_G, "b", 3);
print(b) --> 3
--- 使用rawget 对变量取值
if rawget(_G, "a") == nil then
    print(rawget(_G, "b"))
end
```

局部环境
```lua
--- 在5.3中增加了_ENV来管理环境表
--- _ENV 从load 开始（初始化为_G)，第一个chunk就被加上 _ENV这个upvalue，然后依次传递下去
--- 如果在某个chunk 中定义 local _ENV = {}，就相当于修改这个chunk下面的环境
--- Lua 在编译时会给变量名 var 变为 _ENV.var

a = 3
--- 封装旧环境
_ENV = {_G = _G};
a = 1;
_G.print(a, _G.a) -->1	3
```
```lua
a = 3;
--- 用继承的方式封装
local newgt = {};
setmetatable(newgt, {__index = _G})
_ENV = newgt;
a = 10;
print(a, _G.a); --> 10	3
```

当创建一个新函数时，将默认继承该函数所在的环境变量

## 第15章 Packages
包的实现写在complex.lua下，与测试程序文件夹不同
```lua
--- test.lua
kk = require("complex")
c = kk.new(2, 1);
print(c.r, c.i); --> 2	1
```

包的实现
```lua
--- 基本方法
local P = {};
P.r, P.i = 0, 1;

function P.new(r, i) return {r = r, i = i}; end

function P.add(c1, c2)
    return P.new(c1.r + c2.r, c1.i + c2.i);
end

function P.sub(c1, c2)
    return P.new(c1.r - c2.r, c1.i - c2.i);
end

function P.mul(c1, c2)
    return P.new(c1.r * c2.r - c1.i * c2.i, c1.r * c2.i + c1.i * c2.r);
end

function P.inv(c)
    local n = c.r * c.r + c.i * c.i;
    return P.new(c.r / n, -c.i / n);
end

--- 私有成员
local function checkComplex(c)
    if not ((type(c) == "table")) and
    tonumber(c.r) and tonumber(c.i) then
        error("bad complex number", 3);
    end
end

complex = P;
return complex;
```
```lua
--- 将成员区分为公有或私有，到此有两种方法（下面还有设置环境的方法）
--- 一种是传统地用local 声明的为私有，否则为公有
--- 另一种是将公有成员打包return，如此案例，local 声明可以忽略

local P = {};
P.r, P.i = 0, 1;

local function new(r, i) return {r = r, i = i}; end

local function add(c1, c2)
    return P.new(c1.r + c2.r, c1.i + c2.i);
end

local function sub(c1, c2)
    return P.new(c1.r - c2.r, c1.i - c2.i);
end

local function mul(c1, c2)
    return P.new(c1.r * c2.r - c1.i * c2.i, c1.r * c2.i + c1.i * c2.r);
end

local function inv(c)
    local n = c.r * c.r + c.i * c.i;
    return P.new(c.r / n, -c.i / n);
end

local function checkComplex(c)
    if not ((type(c) == "table")) and
    tonumber(c.r) and tonumber(c.i) then
        error("bad complex number", 3);
    end
end

complex = {
    new = new,
    add = add,
    sub = sub,
    mul = mul,
    inv = inv
};

return complex;
```

require 命令不会将相同的package加载多次

```lua
--- 使用全局表，可把需要的函数先声明为local，
--- 或者直接把全局变量添加到环境里
--- 在此环境内，函数默认添加complex.前缀
--- 在最后的return就可以区分共有和私有，所以local回归环境内局部变量的本意

local print = print;
_ENV = {}

local P = {};
P.r, P.i = 0, 1;

function new(r, i) return {r = r, i = i}; end

function add(c1, c2)
    return P.new(c1.r + c2.r, c1.i + c2.i);
end

function sub(c1, c2)
    return P.new(c1.r - c2.r, c1.i - c2.i);
end

function mul(c1, c2)
    return P.new(c1.r * c2.r - c1.i * c2.i, c1.r * c2.i + c1.i * c2.r);
end

function inv(c)
    local n = c.r * c.r + c.i * c.i;
    return P.new(c.r / n, -c.i / n);
end

function checkComplex(c)
    if not ((type(c) == "table")) and
    tonumber(c.r) and tonumber(c.i) then
        error("bad complex number", 3);
    end
end

complex = {
    new = new,
    add = add,
    sub = sub,
    mul = mul,
    inv = inv
};

return complex;
```

可以在同一个文件内定义多个packages，将每个package放在一个do代码块内

函数只有被实际使用的时候才会自动加载。当自动加载package时，会自动创建一个新的空表表示package并设置表的 __index metamethod来完成自动加载。当调用没有被加载的函数时， __index 被触发去加载该函数，当发现函数已经被加载，__index不会被触发

## 第16章 面向对象程序设计
```lua
--- 定义方法的时候带上self或this参数，表示方法作用的对象
Account = {balance = 0}
function Account.withdraw(self, v)
    self.balance = self.balance - v;
end
a = Account;
Account = nil;
a.withdraw(a, 100);
print(a.balance) --> -100

--- 通过冒号操作符隐藏self参数的声明
--- 冒号只是一种方便的语法，与dot语法可相互切换
Account = {balance = 0}
function Account:withdraw(v)
    self.balance = self.balance - v;
end
a = Account;
Account = nil;
a:withdraw(10);
print(a.balance) --> -10
```

Lua不存在类的概念，可以通过metatable来实现继承
> 有些语言可以基于原型（prototype）来效仿类的概念。这些语言中对象没有类，有一个prototype（原型），在Lua中具体为metatable，是关于表的表，描述并规范了表的行为。当调用不属于对象的某些操作i时，会最先到prototype中查找这些操作

类
```lua
--- b是a的prototype，对象a调用不存在的成员都会到对象b中查找
setmetatable(a, b);
```
```lua
Account = {balance = 0,
    deposit = function (self, v)
        self.balance = self.balance + v;
    end
}

function Account:new(o)
    o = o or {};
    --- 可以不创建额外的表作为metatable，用self（此时为Account）
    setmetatable(o, self);
    self.__index = self;
    return o;
end

function Account:withdraw(v)
    self.balance = self.balance - v;
end

a = Account:new({balance = 0;});
--- 如同调用getmetatable(a).__index.deposit(a, 100);
a:deposit(100);
print(a.balance); --> 100

--- 继承默认值
b = Account:new();
print(b.balance) --> 0
```

继承
```lua
Account = {balance = 0}

function Account: new(o)
    o = o or {};
    setmetatable(o, self);
    self.__index = self;
    return o;
end

function Account: deposit(v)
    self.balance = self.balance + v;
end

function Account: withdraw(v)
    if v > self.balance then
        error("insufficient funds");
    end
    self.balance = self.balance - v;
end

--- 派生子类SpecialAccount，添加变量limit
SpecialAccount = Account:new({limit = 1000});

--- 重写withdraw函数
function SpecialAccount:withdraw(v)
    if v - self.balance >= (self.limit or 0) then
        error("insufficient funds");
    end
    self.balance = self.balance - v
end
--- SpecialAccount从Account继承了new方法，执行new的时候，self参数指向SpecialAccount
--- 即s的metatable是SpecialAccount，__index也是SpecialAccount
s = SpecialAccount:new();
print(getmetatable(s) == SpecialAccount) -- true
s:withdraw(100);
print(s.limit, s.balance); --> 1000	-100
```

多重继承
```lua
Account = {balance = 0};
function Account:deposit(v)
    self.balance = self.balance + v;
end
Name = {name = "init"}
function Name:getName()
    return self.name;
end

--- 多重继承可以理解成在多个table里面查找一个key
--- classes存放多个父类表，key为要查找的字段
local function search(key, classes)
    for i = 1, #classes do
        local v = classes[i][key];
        if v then return v end
    end
end

function createClass(...)
    local parent = {...};
    local child = {};

    --- 设置类的metatable
    --- 当__index元方法为函数，参数为元表所属的table，以及要查找的字段名
    setmetatable(child, {__index = function (table, key)
        return search(key, parent);
        end }
    );

    --- 创建对象
    function child: new(o)
        o = o or {};
        setmetatable(o, child);
        child.__index = child;
        return o;
    end

    return child;
end

nameAccount = createClass(Account, Name);
print(nameAccount:getName());
```

> Lua不提供私有性访问机制，在设计上就没有打算被用来进行大型的程序设计。所以Lua避免太冗余和过多人为限制。如果不想访问对象内的一些东西就不要访问（If you do not want to access something inside an object, just do not do it.）

Lua本身不提供私有性机制，但是可以用别的方式去实现（比如上文用local限制访问权限）。设计的基本思想是，每个对象用两个表来表示：一个描述状态，一个描述操作（接口）。对象本身通过第二个表来访问，状态则保存在方法的闭包内
```lua
function newAccount(initalBalance)
    local self = {balance = initalBalance};
    local withdraw = function(v)
        self.balance = self.balance - v;
    end

    local deposit = function(v)
        self.balance = self.balance + v;
    end

    local getBalance = function () return self.balance  end

    return {
        withdraw = withdraw;
        deposit = deposit;
        getBalance = getBalance;
    }
end
```

Single-Method的对象实现方法
```lua
--- 就是闭包
function newObject(value)
    return function (action, v)
        if action == "get" then return value;
        elseif action == "set" then value = v;
        else error("invalid action");
        end
    end
end
f = newObject(10);
print(f("get")); --> 10
f("set", -1);
print(f("get")); --> -1
```

## 第17章 Weak表
表的weak由metatable的__mode域指定。在域存在时，必须为字符串：对于这个字符串，若包含小写字母'k'，则table的keys就是weak；若包含小写字母'v'，则table的values就是weak

当key或value被垃圾回收器收集后，整个入口（entry）都将从table消失，即内存回收
```lua
weakTable = {}
weakTable[1] = function() print("A") end
weakTable[2] = function() print("B") end
weakTable[3] = {10, 20, 30}

--- 设置为弱表
setmetatable(weakTable, {__mode = "v"})

print(#weakTable) --> 3

--- 强引用第一个元素
ele = weakTable[1]
--- 强制垃圾回收
collectgarbage()
print(#weakTable) --> 1
--- 释放引用
ele = nil
collectgarbage()
print(#weakTable) --> 0
```

要注意，只有对象才可以从一个weak table中被回收。比如number和bool类型就不会被回收，在上例中将'v'改为'k'可验证。

记忆函数
```lua
--- 建立正在使用的颜色的缓存
local result = {};
setmetatable(result, {__mode = "v"});
function createRGB(r, g, b)
    local key = r .. "-" .. g .. "-" .. b;
    if result[key] then
        return result[key];
    else
        local newColor = {red = r, green = g, blue = b};
        result[key] = newColor;
        return newColor;
    end
end
```

关联对象属性。Lua本身使用这种技术保存数组的大小

带有默认值的表
```lua
--- 关联对象属性的方法
local defaults = {};
setmetatable(defaults, {__mode = "k"})

local mt = {__index = function (t) return defaults[t] end}

function setDefault(t, d)
    defaults[t] = d;
    setmetatable(t, mt);
end

t = {1};
setDefault(t, 3);
print(defaults[t]); --> 3

--- 第二个方法
--- 针对不同metatable 进行优化
--- metas 维护不同的 metatable，metatable 与 table的关系为一对多
metas = {};
setmetatable(metas, {__mode = "v"})
setDefault = function (t, d)
    local mt = metas[d];
    if mt == nil then
        mt = {__index = function() return d end}
        metas[d] = mt;
    end
    setmetatable(t, mt);
end
```

## 第18章 数学库
```lua
print(math.sin(math.pi / 2))
math.randomseed(os.time())
print(math.random())
```

## 第19章 Table库
```lua
--- 数组大小
--- #统计长度统计的是有效key，这里#a = 3对应的value分别是3, nil, {"a"}
a = {3, nil, {"a"}, string = "str", number = 1}
print("#a:", #a);
for k, v in pairs(a) do
    print(k, v)
end
--[[
#a:	3
1	3
3	table: 0000000000198d70
number	1
string	str
--]]

--- 插入
table.insert(a, 3, 4);

--- sort
--- ipairs使用key的顺序遍历，而pairs使用自然存储顺序遍历
a = {"a", "d", "b"}
table.sort(a);
```

## 第20章 String库
```lua
str = "string?";
print(string.len(str)); --> 7
print(string.rep(str, 3)) --> string?string?string?
print(string.lower(str)) --> string?
print(string.upper(str)) --> STRING?
print(string.sub(str, 2, -2)); --> tring
print(string.format("<%s>,%d", "string", 10)); --> <string>,10
print(string.byte("ba", 1)); --> 98 ASCII码
```

出于程序大小方面的考虑，Lua不使用POSIX规范的正则表达式（regexp）

```lua
s = "hello world"
--- 返回匹配串开始索引和结束索引的位置
print(string.find(s, "el"))
--- 返回替换后的串，进行替换的次数
print(string.gsub("Lua is cute", "cute", "great"))
```

模式。字符类的大写形式表示小写所代表的集合的补集
| . | %a | %c | %d | %l | %p | %s | %u | %w | %x | %z |
| - | - | - | - | - | - | - | - | - | - | - |
| 任意字符 | 字母 | 控制字符 | 数字 | 小写字母 | 标点字符 | 空白符 | 大写字母 | 字母和数字 | 十六进制数字 | 代表0的字符 |

| % | 特殊字符的转义字符（注意是特殊字符的转义，Lua的转义字符为\\） |
| - | - |
| [] | 字符集，匹配方括号内的任意字符 |
| \[0-9] | 匹配十进制数 |
| ^ | 在字符集开始处表示补集，\[%S]同 \[\^%s]；\^开头的模式表示匹配从目标串的第一个字符开始 |
| $ | $结尾的模式表示匹配到目标串的最后一个字符结束 |

修饰符
| + | 匹配前一字符1次或多次，进行最长匹配 |
| - | - |
| * | 匹配前一字符0次或多次，进行最长匹配 |
| - | 匹配前一字符0次或多次，进行最短匹配 |
| ? | 匹配前一字符0次或1次 |
```lua
print(string.gsub("Lua is cute!", "%A", ">>")) --> Lua>>is>>cute>>	3
print(string.gsub("abcdw", "[aw]", ">>")) --> >>bcd>>	2

--- 匹配括号之间的空白 "%(%s*%)"
--- 查找C程序中的注释，可能会用 "/%*.*%*/"，由于".*"进行的是最长匹配，该模式将匹配程序中第一个"/*"和最后一个"*/"之间的所有部分
s = [[
/* x */
int x;
/* y */
int y;
]]
sub, num = string.gsub(s, "/%*.*%*/", "<COMMENT>");
print(sub.."====="..num);
--[[
<COMMENT>
int y;
=====1
--]]

sub, num = string.gsub(s, "/%*.-%*/", "<COMMENT>");
print(sub.."====="..num);
--[[
<COMMENT>
int x;
<COMMENT>
int y;
=====2
--]]

--- "%b"用来匹配对称的字符，"%bxy"，x作为匹配的开始，y作为匹配的结束
--- 匹配()内的字符串
print(string.gsub("a (enclosed (in) parentheses) line", "%b()", "!")); --> a ! line	1
```

捕获（Capture）：使用模式串的一部分匹配目标串的一部分。将想捕获的模式用圆括号括起来就指定一个capture
```lua
--- 整个模式代表：一个字母序列，后面是任意多空白，然后是"="，后面是任意多空白，最后是字母序列
--- find 先返回匹配串的索引下标，然后返回子模式匹配的捕获部分
pair = "name = Anna"
print(string.find(pair, "(%a+)%s*=%s*(%a+)")) --> 1	11	name	Anna
date = "2021/9/18";
print(string.find(date, "(%d+)/(%d+)/(%d+)")); --> 1	9	2021	9	18

--- 向前引用，匹配字符串中单引号或双引号引起来的子串
--- 模式串为 (["'])(.-)%1，
s = [[then he said: "it's all right!"]]
print(string.find(s, "([\"'])(.-)%1")); --> 15	31	"	it's all right!

--- "\\(%a+){(.-)}"， 将LaTeX风格转换成XML风格
s = [[\command{some text}]]
print(string.gsub(s, "\\(%a+){(.-)}", "<%1>%2<%1>")); --> <command>some text<command>	1

--- 将捕获值作为函数的传参，返回值作为gsub的替换串
print(string.gsub("$name is $status, isn't it?", "$(%w+)",
    function (n)
         return "function";
    end )) --> function is function, isn't it?	2

--- 收集字符串中所有单词
words = {}
string.gsub(s, "(%a+)", function(w)
    table.insert(words, w);
end);
```

一些技巧
```lua
--- 重复匹配单个字符的模式70次
pattern = string.rep("[^\n]*", 70) .. "[^\n]*";
--- 忽略大小写
string.gsub(s, "%a", function (c)
    return string.format("[%s%s]", string.lower(c), string.upper(c));
end );
```

## 第21章 IO库
```lua
--- 简单IO，IO标准输入输出文件
--- read函数的参数：
--- "*all" 读取整个文件
--- "*line" 读取下一行，默认参数
--- "*number" 从串中转换出一个数值
--- num 读取num个字符到串
a, b, c = io.read("*number", "*number", "*number")
io.write(a, b, c)

--- 完全IO模式
f = io.open("test.lua", "r");
s = f:read("*all");
io.write(s)
f:close();

--- IO库提供三种预定义句柄：io.stdin, io.stdout, io.stderr
--- 带参io.input(handle)改变当前输入文件
io.output():write("write");

--- 行读取的基础上限制字节数
--- rest 保存了可能被段划分切断的行
BUFSIZE = 10;
lines, rest = f:read(BUFSIZE, "*line");

--- 函数filehandle:seek(whence,offset)设置文件的当前存取位置
--- 获取文件大小
function fsize(file)
    local current = file:seek();
    local size = file:seek("end");
    file:seek("set", current);
    return size;
end
```

## 第22章 操作系统库