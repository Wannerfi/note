[toc]
## cpp版的编译运行
项目的源码版本只开放了cpp，而lua版在release下有可执行文件
clone 项目下来，打开trunk文件夹，用vs2019编译即可。
当计算机用户不是 Administrator（管理员） 时，需要如下设置编译配置。否则后面打开cmd执行exe文件时，会弹出另一个窗口执行程序，导致输出重定向失败。
![](./tscancode%E6%BA%90%E7%A0%81check%E6%B5%81%E7%A8%8B/1.png)

编译完后打开bin/debug 文件夹，在该文件夹下打开cmd，输入以下命令即可运行，看到在当前文件夹下err.txt 即为执行结果
```cmd
tscancode --xml ../../../samples/cpp/arrayindexthencheck.cpp 2>err.txt
```

## 程序流程
先说明一下部分文件内容，想看懂大致流程最好这几个文件先过一遍
| 文件名 | 内容 |
| -- | -- |
| setting | 单例，保存程序整个运行过程的配置 |
| tscexecutor | tscancode 库的一个使用例子，在此程序中是实际入口 |
| cmdlineparser | 解析命令行，这里能看到更多隐藏的参数，-h文档并不完整 |


大概流程从 main 进去还是比较清晰的。
- parseFromArgs。先解析命令行并加载配置 cfg.xml。这里把配置写死在 cfg/cfg.xml 了。
- check_internal。进入检查
- - 先加载一些.cfg
- - 执行 analyze，然后进行check，这两个过程各自使用了多线程，执行相同函数，但是参数不同
- - uninit。最后取消初始化，返回退出代码

整个流程中重要是这个check

## check过程
这里的一些全局单例我还没看懂是什么意思：CGlobalTokenizer。
**多线程**
check先对文件进行一些筛选和排序的预处理，最重要的语句是
`unsigned ret = multi_thread(TscThreadExecutor::threadProc);`，这个语句，在multi_thread 中使用多线程，处理函数 threadProc。最后收集错误报告。

**每个线程执行的内容**
TscThreadExecutor::threadProc 函数在线程中实例化了TscanCode（该类在main流程进去不久就已经实例化过一次，但是跟这里没啥关系，别混了），并执行analyze和check，下面进入这个具体的check，走到TscanCode::processFile，这里对文件进行处理，并执行真正进行代码检测的函数。

**TscanCode::processFile**
这里先对要检测的代码进行预处理，即`preprocess`，然后略去一些看不懂的。。。（获取用户自定义的配置？），然后进入`checkFile()`进行检测。

**checkFile**
进入checkFile后，先获取Tokenizer（分词器？反正这里有结构化后的代码），然后下面有如下格式的循环。
```cpp
for(it = Check::instances().begin(); it != Check::instances().end(); ++it) {}
```
这个就是遍历所有的已注册Check进行操作。

**Check::instances 的注册**
instances 可以获取一个静态Check 列表，里面包括了所有已注册的Check 类。按照一般思路，都是通过静态变量进行实例的注册。找半天才发现在cpp文件中。以`checkbool.cpp`为例，在cpp文件的开头有以下内容
```cpp
// Register this check class (by creating a static instance of it)
namespace {
    CheckBool instance;
}
```
在静态变量实例化的时候，会走到基类`Check`的构造函数，将指向派生类实例的指针按命名顺序，插入到静态Check 列表中。大致内容如下：
```cpp
Check::Check(name)
{
    // 按命名排序
    for(it: instances()) {
        if(it->name() > name) {
            insert(i, this);
            return ;
        }
    };
    push_back(this);
}
```


## 未解决问题，后面看懂再更
没看懂cfg.xml 配置与check类之间的联系。
processFile 还没看懂。