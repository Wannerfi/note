[toc]
# 参考
虚幻引擎源码 4.26.2
# 前言
> 对于本笔记出现的所有引用，读者均需考虑版本差异。本笔记只保证本笔记中出现的源码内容对应参考的版本号，不确定引用的其他内容的源码版本。

不管怎么说，虚幻引擎终究就只是个应用程序。应用程序的简单运行类似以下代码。Tick好比心跳，负责更新应用程序的状态。
```cpp
int main() {
    Application App;
    while(!App.bInterrupt) {
        App.Tick();
    }
    return 0;
}
```
**引擎不被编译成模块？**
UE4是分模块的，而且是被编译成DLL，但是为什么引擎却是应用程序？
答案在UBT，根据《大象无形：虚幻引擎源码浅析》，可以找到在UEBuildTarget.cs 文件下的`SetupBinaries()函数`，该函数将Launch模块设为Executable，并将模块注入为.exe文件的包含模块。

# UE4在Windows的启动
Windows下程序的入口在源码文件夹下`/Source/Runtime/Launch/Windows/LaunchWindows.cpp`，进去后能找到`WinMain函数`，这就是Windows下的启动函数。
下表为`WinMain函数`中出现的部分变量含义
| 变量名 | 含义 |
| -- | -- |
| GIsFirstInstance | 启动时需要创建一个实例吗 |
| GEnableInnerException | 使用本机C++中的异常处理程序吗 |

`WinMain函数`之后会进入`Launch.cpp/GuardedMain函数`，进入主程序的守护进程，该函数主要执行以下步骤（略去部分包装）：
- 执行预初始化。实际上是执行`FEngineLoop::PreInit()`
- 执行初始化。运行 `FEngineLoop::Init()`
- 循环执行Tick直到满足退出条件。
- 局部变量析构执行。EngineExit

后面标题将对应主要步骤进行介绍

**确保引擎退出函数总能执行**
`GuardedMain函数`有以下内容。局部变量`CleanupGuard`在后面也没有用到。这个局部变量在退出函数作用域的时候肯定会被析构，调用析构函数从而保证执行`EngineExit()`，也保证了在最后执行。
```cpp
int32 GuardedMain( const TCHAR* CmdLine )
{
    //...
    // make sure GEngineLoop::Exit() is always called.
	struct EngineLoopCleanupGuard 
	{ 
		~EngineLoopCleanupGuard()
		{
			EngineExit();
		}
	} CleanupGuard;
    //...
}
```
## 预初始化 EnginePreInit
预初始化主要分为Init，Post两步。这些步骤并不严格按照顺序，有些步骤在条件允许下会提前执行。
Init主要执行以下步骤：
- 初始化GLog，设定当前线程为游戏主线程
- 初始化控制台，设置GLog标准输出系统
- 判定引擎的启动模式：游戏模式还是服务器模式
- 初始化随机数生成器，初始化TaskGraph，线程池
- 加载其他的核心模块，包括执行函数`FEngineLoop::LoadPreInitModules`，去加载渲染，动画蓝图等模块
- 执行`AppInit()`
- 初始化RHI，渲染工具
- 初始化渲染器

Post主要执行以下步骤：
- 运行自动注册函数，确保所有UObject类都已注册并初始化默认属性
- 初始化纹理流送系统
- 在渲染线程启动前执行所有post appInit操作
- 启动渲染线程 `StartRenderingThread()`
- - 创建渲染线程的Tick

**Dedicated Servers在Init中只有一个线程**
```cpp
// we are only going to give dedicated servers one pool thread
if (FPlatformProperties::IsServerOnly())
{
    NumThreadsInThreadPool = 1;
}
```

## 初始化
`WinMain函数`下的初始化步骤都会运行`FEngineLoop::Init()`函数，该函数主要内容如下：
- 找出实际的UEngine 变体
- 广播。FCoreDelegates::OnPostEngineInit
- 调用所有加载到内存的模块的`PostEngineInit函数`
- GEngine->Start();

此时如果启动了编辑器，还会进行编辑器的初始化

## 循环执行Tick直到满足退出条件
```cpp
while( !IsEngineExitRequested() )
{
	EngineTick();
}
```
Tick会进入到`FEngineLoop::Tick()`，主要步骤按顺序如下：
- 更新 FApp::CurrentTime / FApp::DeltaTime
- 遍历`FWorldContext`，更新所有原始场景信息。`Scene->UpdateAllPrimitiveSceneInfos()`
- 开始RHI线程。`BeginFrameRenderThread()`
- 遍历`FWorldContext`，执行`Scene->StartFrame()`
- 轮询输入设备
- 执行游戏Tick。`GEngine->Tick(FApp::GetDeltaTime(), bIdleMode);`
- 更新Slate
- 更新RHI。`RHITick( FApp::GetDeltaTime() ); // Update RHI.`
- 同步游戏和渲染线程，允许一帧的滞后
- 清除前一帧之前排队等待清除的对象
- 结束RHI线程。`EndFrameRenderThread()`

要注意的是渲染的更新和游戏的更新是分开的。函数`FFrameEndSync::Sync`实现了游戏和渲染线程的同步