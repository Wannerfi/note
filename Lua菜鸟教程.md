`print("Hello World1")`

----

单行注释
`--`

多行注释
```lua
--[[
    多行注释
--]]
```

默认情况下，变量总认为是全局的，没有初始化的全局变量的值为nil
```lua
print(b)
b = 10
print(b)
```

| 数据类型 | 描述 |
|-|-|
| nil | 表示无效值（false） | 
| boolean | true or false，只有false和nil为false，其他都为true |
| number | double |
| string | 一对单引号或双引号表示 |
| function | 函数 |
| userdata | 表示任意存储在变量中的C数据结构 | 
| thread | 表示执行的独立线路，用于执行协同程序 |
| table | 关联数组 |

nil，在判断语句中为false，把变量置为nil，会释放内存，触发内存回收机制

```lua
tab1 = {key1 = 'val1', key1 = 'val2', key2 = 'val2', 'val2'}
for k, v in pairs(tab1) do
    print(k .. ' - ' .. v)
end
print(tab1.key1 .. ' - ' .. tab1[1])
```

type()返回值为string
```lua
print(type(X) == "nil")
```

数字字符串的算术操作，类型会转化为number
```lua
print('234' + '12')
```

字符串连接用..
使用#计算字符串的长度，或者表的长度（索引中断即停止计数）
```lua
print(#('23a4' .. '23'))
```

table
默认初始索引一般以1开始
```lua
local t1 = {}
local t1 = {"a", "b"}
for i, val in pairs(t1) do
    print(i..' '..val)
end

arr = {}
for i = 1, 10 do
    arr[i] = i 
end
for key, val in pairs(arr) do
    print(key .. ' ' .. val)
end
```

函数被看做“第一类值”，可以存在变量里
```lua
function f(n)
    if n == 0 then
        return 1
    else
        return n * f(n - 1)
    end
end
print(f(5))
p = f
print(p(6))
```

通过匿名函数的方式传参
```lua
function testFun(tab, fun)
    for k, v in pairs(tab) do
        print(fun(k, v))
    end
end

tab = {k1 = "v1", k2 = "v2", "v3"}
testFun(tab, function(key, val)
                return key..'='..val;
            end)
```

userdata
一种用户自定义数据，可以将任意C/C++的任意数据类型的数据存储到Lua变量中

---
local声明局部变量
`local a, b = 10, 388
print(a, b)`
多余变量补足nil，多余值被忽略

对table的索引使用了`[]`，也提供了`.`操作
`t[i], t.i`

循环
```lua
while a < 20 do
    print(a)
    if a > 15 then
        break;
    end
    a = a + 1
end

-- 数值for
-- 初始值，上限，步长。步长默认1
for i = 1, 10, 1.3 do
    print(i)
end
-- for的三个表达式在循环开始前一次性求值，此后不再进行求值
function f(a)
    print('I am f')
    return 5
end

function p()
    print('I am p');
    return 1.1
end

for i = 1, f(i), p() do
    print(i)
end
-- 泛型for
a = {'one', 'two', 'three'}
for i, val in ipairs(a) do
    print(i, val)
end

-- repeat
b = 0
repeat
    print(b)
    b = b + 1
until(b > 5)

-- goto
local a = 1
::label:: print('I am label')
a = a + 1
if a < 4 then
    goto label
end
```

判断
```lua
if a > 100 then
    print(a)
elseif a > 80 then
    print('good')
elseif a > 70 then
    print('emm')
else
    print('laji')
end
```

函数
```lua
function max(a, b)
    if a > b then
        return a, b
    else
        return b, a
    end
end

//可变参数
function f(a, b, ...)
    local arg = {...}
    print(#arg)
    for i, val in pairs(arg) do
        print(i, val)
    end
end
select('#", ...) -- 返回可变参数的长度
select(n, ...) -- 从n开始计算长度
```

运算符
| 符号 | 说明 |
|-|-|
| ~= | 不等于 |
| and | a为false返回a，否则返回b |
| or | a为true返回a，否则返回b |
| not | ! |

\[\[]]表示多行字符串

字符串格式化 `string.format()`
> %c - 接受一个数字, 并将其转化为ASCII码表中对应的字符
%d, %i - 接受一个数字并将其转化为有符号的整数格式
%o - 接受一个数字并将其转化为八进制数格式
%u - 接受一个数字并将其转化为无符号整数格式
%x - 接受一个数字并将其转化为十六进制数格式, 使用小写字母
%X - 接受一个数字并将其转化为十六进制数格式, 使用大写字母
%e - 接受一个数字并将其转化为科学记数法格式, 使用小写字母e
%E - 接受一个数字并将其转化为科学记数法格式, 使用大写字母E
%f - 接受一个数字并将其转化为浮点数格式
%g(%G) - 接受一个数字并将其转化为%e(%E, 对应%G)及%f中较短的一种格式
%q - 接受一个字符串并将其转化为可安全被Lua编译器读入的格式
%s - 接受一个字符串并按照给定的参数格式化该字符串  
(1) 符号: 一个+号表示其后的数字转义符将让正数显示正号. 默认情况下只有负数显示符号.
(2) 占位符: 一个0, 在后面指定了字串宽度时占位用. 不填时的默认占位符是空格.
(3) 对齐标识: 在指定了字串宽度时, 默认为右对齐, 增加-号可以改为左对齐.
(4) 宽度数值
(5) 小数位数/字串裁切: 在宽度数值后增加的小数部分n

字符串匹配 `string.find(), string.match()`
> .(点): 与任何字符配对
%a: 与任何字母配对
%c: 与任何控制符配对(例如\n)
%d: 与任何数字配对
%l: 与任何小写字母配对
%p: 与任何标点(punctuation)配对
%s: 与空白字符配对
%u: 与任何大写字母配对
%w: 与任何字母/数字配对
%x: 与任何十六进制数配对
%z: 与任何代表0的字符配对
%x(此处x是非字母非数字字符): 与字符x配对. 主要用来处理表达式中有功能的字符(^$()%.[]*+-?)的配对问题, 例如%%与%配对
[数个字符类]: 与任何[]中包含的字符类配对. 例如[%w_]与任何字母/数字, 或下划线符号(_)配对
[^数个字符类]: 与任何不包含在[]中的字符类配对. 例如[^%s]与任何非空白字符配对

模式条目 ~~这是正则表达式？~~
> 单个字符类跟一个 '\*'， 将匹配零或多个该类的字符。总是匹配尽可能长的串；
单个字符类跟一个 '+'， 将匹配一或更多个该类的字符。总是匹配尽可能长的串；
单个字符类跟一个 '-'， 将匹配零或更多个该类的字符。总是匹配尽可能短的串；
单个字符类跟一个 '?'， 将匹配零或一个该类的字符。
%n， 这里的 n 可以从 1 到 9； 这个条目匹配一个等于 n 号捕获物（后面有描述）的子串。
%bxy， 这里的 x 和 y 是两个明确的字符； 这个条目匹配以 x 开始 y 结束， 且其中 x 和 y 保持 平衡 的字符串。 意思是，如果从左到右读这个字符串，对每次读到一个 x 就 +1 ，读到一个 y 就 -1， 最终结束处的那个 y 是第一个记数到 0 的 y。 举个例子，条目 %b() 可以匹配到括号平衡的表达式。
%f[set]， 指 边境模式； 这个条目会匹配到一个位于 set 内某个字符之前的一个空串， 且这个位置的前一个字符不属于 set。

数组
```lua
arr = {}
for i = 1, 3 do
    arr[i] = {}
    for j = 1, 3 do
        arr[i][j] = i * j
    end
end
for i = 1, 3 do
    for j = 1, 3 do
        print(arr[i][j])
    end
end
```

闭包
```lua
function newCounter()
    local i = 0
    return function()     -- anonymous function
        i = i + 1
        return i
    end
end
c1 = newCounter()
print(c1()) --> 1
print(c1()) --> 2
-- 在匿名函数内部grades是成为外部的局部变量(external local variable)，或者upvalue
-- 匿名函数使用upvalue i保存它的技术，虽然i 已经超出了作用范围，但是Lua用闭包的思想处理了这种情况
-- 简单地说，闭包是一个函数以及它的upvalues
-- 如果再次调用newCounter，将创建一个新的局部变量i，得到作用在新的变量i上的新闭包
c2 = newCounter()
print(c2()) --> 1
-- c1，c2是作用在同一局部变量的不同实例上的两个不同的闭包
-- 从技术上来讲，闭包指值而不是指函数，函数仅仅是闭包的一个原型声明，一般来说还是用函数指代闭包
```

Lua迭代器
```lua
-- 泛型for迭代器
-- 泛型for在循环过程中保存了三个值：迭代器函数，恒定状态和控制变量
-- 初始化过程，先计算<explist>，<explist>返回迭代器函数、恒定状态和控制变量
-- 将恒定状态和控制变量作为参数调用迭代函数
-- 将迭代函数返回的值赋给变量列表
-- 直到返回的第一个值为nil结束，否则回到调用迭代函数那一步
for var_1, ..., var_n in <explist> do <block> end
-- 等同于
do
    local _f, _s, _var = <explist>
    while true do
        local var_1, ..., var_n = _f(_s, _var)
        _var = var_1
        if _var == nil then break end
        <block>
    end
end

-- 无状态的迭代器，就是一种自身不保存任何状态的迭代器，因此可以避免创建闭包话费额外的代价
-- 这类迭代器的代表为ipairs
arr = {'one', 'tow', 'three'}
for i, v in ipairs(arr) do
    print(i, v)
end
-- ipairs和迭代函数可以在lua中这样实现
function iter(a, i)
    i = i + 1
    local v = a[i]
    if v then
        return i, v
    end
end
function ipairs(a)
    return iter, a, 0
end
-- ipairs只能遍历下标为整数的值，pairs能遍历table里所有值

-- Lua库中实现pairs是用next
function pairs(t)
    return next, t, nil
end

-- 多状态迭代器
-- 迭代器需要保存多个状态信息时，最简单的方法时使用闭包
-- 还有一种方法是将所有的状态信息封装到table中，将table作为迭代器的状态常量
-- 这种情况下可以将所有的信息存放在table中，此时迭代函数不需要第二个参数（通常来说）
function iterator(state)
    while state.line do
        -- search for next word
        local s, e = string.find(state.line, "%w+", state.pos)
        if s then
            state.pos = e + 1
            return string.sub(state.line, s, e)
        else
            state.line = io.read()
            state.pos = 1
        end
    end
    -- 返回迭代函数和初始状态
    function allwords()
        local state = {line = io.read(), pos = 1}
        return iterator, state
    end
end
-- https://www.shouce.ren/api/lua/5/_48.htm
```

table（表）
```lua
-- 表的赋值语句会使两个变量指向同一块内存
-- 直到没有变量指向这块内存时，它才会被释放
table = {'a'}
p = table
table[2] = 'b'
for k, val in pairs(p) do
    print(k, val)
end
table = nil
for k, val in pairs(p) do
    print(k, val)
end
```
table常用方法
| 方法 | 说明 |
|-|-|
| table.concat(table, [, sep, [, start[, end]]]) | 将分隔符seq插入到指定的table的数组部分 |
|table.insert(table, [pos,] value) | 在table数组部分指定位置pos插入值为value的元素 | 
| table.remove(table[, pos]) | 删除pos(默认数组部分的最后一位)位置的元素并返回其值 |
| table.sort(table, [comp]) | 默认升序排序 |

Lua模块
模块类似于一个封装库。Lua的模块是由变量、函数等已知元素组成的table
```lua
--包的简单方法：包内每个对象前包都加包名作为前缀
local P = {}
test = P
P = {r = 0, i = 1}

function P.new(r, i)
    return {r = r, i = i}
end

function P.add(c1, c2)
    return P.new(c1.r + c2.r, c1.i + c2.i)
end

-- 将私有部分定义为局部变量来实现包的私有成员
local function checkComplex(c)
    if not ((type(c) == "table") and
              tonumber(c.r) and tonumber(c.i)) then
        error("bad complex number", 3)
    end
end

-- 上述私有部分写法冗余，公私有方法切换麻烦，有一个方法可以解决
-- 将package内所有函数都声明为local，最终将公有的函数放置在结尾的列表中
--test = {
--    new = new,
--    add = add,
--}
return test
```

require 函数
`require("test")`。require命令使用文件而不是packages

Lua元表（Metatables）
Metatables允许改变table的行为。当Lua试图对两个表进行相加时，会检查两个表是否有一个表有Metatable，并且检查Metatable是否有__add域。若存在__add函数(Metamethod)，就计算结果。
一组相关的表可以共享一个metatable（描述他们共同的行为）。一个表也可以是自身的metatable（描述其私有行为）
```lua
t = {}
print(getmetatable(t))
t1 = {}
setmetatable(t, t1) -- t1是t的元表
```
```lua
-- 算数运算符
Set = {}
Set.mt = {}
function Set.new(t)
    local set = {}
    setmetatable(set, Set.mt) -- set new创建的集合都有相同的metatable
    for _, l in ipairs(t) do set[l] = true end
    return set
end

function Set.union(a, b)
    local res = Set.new{}
    for k in pairs(a) do res[k] = true end
    for k in pairs(b) do res[k] = true end
    return res
end

function Set.intersection(a, b)
    local res = Set.new{}
    for k in pairs(a) do
        res[k] = b[k]
    end
    return res
end

function Set.tostring(set)
    local s = "{"
    local sep = ""
    for e in pairs(set) do
        s = s .. sep .. e
        sep = ","
    end
    return s .. "}"
end

function Set.print (s)
    print(Set.tostring(s))
end
Set.mt.__add = Set.union
Set.mt.__mul = Set.intersection

s1 = Set.new{10, 20, 30, 40}
s2 = Set.new{30, 1}
Set.print((s1 + s2) * s1)
-- 选择metamethod的原则：如果第一个参数存在带有__add域的metatable，则它作为metamethod，与第二个参数无关
-- 若没有，且第二个参数存在__add域的metatable，则采用第二个参数的作为metamethod，否则报错

-- 关系运算符
-- __eq(等于), __lt(小于), __le(小于等于)
Set.mt.__le = function(a, b)
    for k in pairs(a) do
        if not b[k] then return false end
    end
    return true
end

Set.mt.__eq = function(a, b)
    return a <= b and b <= a
end

-- 库定义
Set.mt.__tostring = Set.tostring
Set.mt.__metatable = "nor your business" -- 隐藏元表

```
```lua
-- 访问表的不存在的域，会触发lua解释器取查找__index metamethod，如果不存在，返回nil
-- 该例的原型是一种继承：创建具有默认值的表。第一种方法时实现一个表的构造器；
-- 第二种方法是创建一个新的窗口取继承一个原型窗口的缺少域
-- 首先实现一个原型和构造函数，他们共享一个metatable
Window = {}
Window.prototype = {x = 0, y = 0, width = 100, height = 100}
Window.mt = {} -- metatable
-- constructor function
function Window.new(o)
    setmetatable(o, Window.mt)
    return o
end
Window.mt.__index = function(table, key)
    return Window.prototype[key]
end
w = Window.new{x = 10, y = 20}
print(w.width) --> 100
-- __index可以是函数，也可以是一个表
-- 当它是函数，table和缺少的域作为参数调用这个函数
-- 当它是表，就查表，所以__index改写如下
Window.mt.__index = Window.prototype
-- 关于prototype与metatable的关系，参照下方知乎，以下为个人理解
-- prototype只是metatable可以实现的功能之一，实际上 __index = metatable 即可实现prototype的功能
-- metatable的本质就如同它的名字，关于表的表
-- 它解释关于table的数据与功能，例如表的运算法则
-- https://www.zhihu.com/question/60382531

-- __newindex用于表的更新，当给表缺少的域赋值，解释器会查找__newindex metamethod
-- 如果存在函数则调用函数，如果是表，则对指定的表赋值，而不是原始的表

-- 有默认值的表
local key = {} -- unique key
local mt = {__index = function(t) return t[key] end}
function setDefault(t, d)
    t[key] = d
    setmetatable(t, mt)
end
t = {}
setDefault(t, 10)
print(t.x)


```
Lua协同程序
协同程序有自己的堆栈，自己的局部变量，有自己的指令指针（IP，instruction pointer），但与其它协同程序共享全局变量等很多信息。
在任一指定时刻只有一个协同程序在运行
```lua
-- 通常情况下，create的参数是匿名函数
co = coroutine.create(function() print("hi") end)
print(co) --> thread: 000000000030d588
-- 协同有三个状态，挂起态suspended，运行态running，停止态dead。初始为挂起态
print(coroutine.status(co)) -- 检查状态
coroutine.resume(co) -- 运行态running
print(coroutine.status(co)) -- 任务完成，进入dead

-- 协同的强大体现在yield函数，它将正在运行的代码挂起
co = coroutine.create(function()
    for i = 1, 3 do
        print("co", i)
        coroutine.yield()
    end
end)
coroutine.resume(co)
coroutine.resume(co)
coroutine.resume(co)
coroutine.resume(co)
print(coroutine.status(co)) -- dead
print(coroutine.resume(co)) -- false
-- resume运行在保护状态下，协同程序内部的错误只会返回给resume函数

co = coroutine.create(function(a, b)
    coroutine.yield(a, b)
    return a + b
end)
print(coroutine.resume(co, 1, 2)) --true	1	2;数据由yield传给resume，true表明调用成功，true后面为yield的参数
print(coroutine.resume(co, 0, 0)) --true	3;挂起后传参无效，直到代码结束，返回值也会传给resume
-- Lua的协同成为不对称协同，指“挂起一个正在执行的协同函数”与“使一个被挂起的协同再次执行的函数”是不同的
-- 有的语言提供对称协同，即使用同一个函数负责“执行与挂起间的状态切换”

-- 生产者-过滤器-消费者
function receive(prod)
    local status, value =  coroutine.resume(prod)
    print(status, value)
    return value
end

function send(x)
    coroutine.yield(x)
end

function producer()
    return coroutine.create(function ()
        while true do
            local x = io.read() --produce new value
            send(x)
        end
    end)
end

function filter(prod)
    return coroutine.create(function ()
        local line = 1
        while true do
            local x = receive(prod)
            x = string.format("%5d %s", line, x)
            send(x)
            line = line + 1
        end
    end)
end

function consumer(prod)
    while true do
        local x = receive(prod)
        io.write(x, "\n")
    end
end

consumer(filter(producer()))
```

```lua
-- Permutations
function permgen(a, n)
    if n == 0 then
        coroutine.yield(a)
    else
        for i = 1, n do
            a[n], a[i] = a[i], a[n] --swap (i-th, last)
            permgen(a, n - 1)
            a[n], a[i] = a[i], a[n]
        end
    end
end

function printResult(a)
    for i, v in ipairs(a) do
        io.write(v, " ")
    end
end

function perm(a)
    local n = #a
    local co = coroutine.create(function() permgen(a, n)  end)
    return function() -- iterator
        local code, res = coroutine.resume(co)
        print(code)
        return res
    end
end

for p in perm{"a", "b", "c"} do
    print(printResult(p))
end
-- perm函数使用的模式：将一对协同的resume的调用封装在一个函数内部。对此，Lua提供了一个wrap函数。
-- 当这个函数被调用时将resume协同。wrap中resume协同的时候不会返回错误代码作为第一个返回结果，一旦有错误发生，将抛出错误
-- 对此，可以重写perm()
for p in perm{"a", "b", "c"} do
    print(printResult(p))
end
```

文件IO
```lua
-- 简单模式 拥有一个当前输入文件和一个当前输出文件，并提供针对这些文件相关的操作
-- 使用io.input ip.output函数来改变当前文件
io.write(math.sin(3), "asdf", 'd')
-- write不附加任何额外的字符到输出，并且使用当前输出文件，print使用标准输出，会自动调用参数的tostring方法

--read函数读取串，由参数控制读取内容
-- "*all" 读取整个文件
-- "*line" 读取下一行
--"*number" 从串转换出一个数组
--num 读取num个字符到串
t = io.read("*line")
print(t)

-- 完全模式 使用外部的文件句柄来实现，以一种面向对象的形式，将所有的文件操作定义为文件句柄的方法
-- 模仿C语言中fopen函数
file = io.open("test.txt", "r")
local t = file:read("*all") --读取调用要用冒号
io.stdout:write("write")
file:close()
```

异常处理
```lua
assert()
error()

-- 异常处理
-- pcall在保护模式下执行函数内容，同时捕获所有的异常和错误
-- 若正常运行，返回true以及"被执行函数的返回值，否则返回nil和错误信息
local status, err = pcall(function() error({code=121})  end)
print(err.code) --> 121

--获取当前运行的traceback
print(debug.traceback())
```

debug
```lua
--debug库并不给你一个可用的Lua调试器，而是给你提供一些为Lua写一个调试器的方便
--应该尽可能少使用debug库，一、一些函数性能较低；
--二、破坏了一些语言的真理，比如不能访问函数内的局部变量

-- debug库由两种函数组成：自省(introspective)和hooks
-- 自省函数可以检查运行程序的某些方面，活动函数栈、当前执行代码行号、本地变量名和值
-- Hooks可以跟踪程序的执行情况
-- getinfo包含的域太多，需要时自行查找
function traceback()
    local level = 1
    while true do
        local info = debug.getinfo(level, "Sl")
        if not info then break end
        if info.what == "C" then -- is a C function?
            print(level, "C function")
        else
            print(string.format("[%s]:%d", info.short_src, info.currentline))
        end
        level = level + 1
    end
end
traceback()
--[test.lua]:17
--[test.lua]:28
--3	C function
```

面向对象
```lua
-- Lua不存在类的概念，但是每个对象都可能有一个prototype(原型)，可以效仿类（或者用metatable实现）
-- 让b作为a的prototype
setmetatable(a, {__index = b})

Account = {balance = 0}
-- 冒号的效果相当于在函数定义和函数调用的时候，增加一个额外的隐藏参数
-- 等同于Account.new(self, o)
function Account:new(o)
    o = o or {}
    setmetatable(o, self)-- 此时self等于Account，Account为a的metatable
    self.__index = self -- __index是一个metamethod，绑定在metatable上的
    return o
end

function Account:deposit(v)
    self.balance = self.balance + v
end

a = Account:new({balance = 10})-- a继承了Account
a:deposit(100)
print(a.balance)

b = a:new{limit = 1000} -- self参数指向a，__index也指向a，b是a的继承，a是Account的继承
-- Lua中的面向对象不需要创建一个新类取指定一个新的行为，直接在对象中实现特殊的行为

-- 多重继承的关键在于：将函数用作__index。
-- 当一个表的metatable存在一个__index函数时，如果Lua调用一个原始表中不存在的函数，Lua将调用这个__index指定的函数。
-- 这样可以用__index实现在多个父类中查找子类不存在的域
local function search (k, plist)
    for i=1, table.getn(plist) do
        local v = plist[i][k]    -- try 'i'-th superclass
        if v then return v end
    end
end

function createClass (...)
    local c = {}
    setmetatable(c, {__index = function (t, k)
        return search(k, arg) -- arg指传参，__index调用元方法，第二个参数是键
    end})
    c.__index = c
    function c:new (o)
        o = o or {}
        setmetatable(o, c)
        return o
    end
    return c
end

-- 关于类的私有性在Lua模块那一部分有说明。主要的设计思想是将开放的接口放置在return的闭包内
return {
    getA = getA
    getB = getB
}

-- Single-Method对象的实现方法
function newObject(value)
    return function(action, v)
        if action == "get" then return value
        elseif action == "set" then value = v
        else error("invalid action")
        end
    end
end

d = newObject(0)
print(d("get"))
d("set", 10)
print(d("get"))
```