[toc]

# 参考
[Actors](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Actors/)
UE4.26.2 源码
# Actor文档
## 定义
所有可以放入关卡的对象都是Actor。这个定义很重要！
Actor不直接保存变换（位置、旋转和缩放）数据。如Actor 的根组件存在，则使用它的变换数据。

## 组件
某种意义上，Actor可被视为包含组件的容器。Actor 的其他主要功能是在游戏进程中在网络上进行属性复制和函数调用
组件的主要类型：
- UActorComponent：基础组件，可作为Actor 的一部分被包含。需要时可进行Tick 。ActorComponents与特定的Actor关联，但不存在与场景中的任意特定位置。通常用于概念上的功能，如AI或解译玩家输入。最适用于抽象行为。
- USceneComponent：是拥有变换的ActorComponents。变换指场景中的位置，旋转和缩放定义。SceneComponents能以层级的方式相互附加。Actor的变换取自位于层级根部的SceneComponent。支持基于位置的行为，这类行为不需要集合表示，如弹簧臂、摄像机、物理力和约束等。
- UPrimitiveComponent：是拥有一类图像表达（网格体、粒子系统等）的SceneComponent。拥有集合表示的场景组件，通常用于渲染视觉元素或与物理对象发生碰撞或重叠。包括静态或骨架网格体、粒子系统、胶囊体碰撞体积。

Actor支持拥有一个SceneComponent的层级。每个Actor也拥有一个`RootComponent`属性作为Actor根的组件。Actor自身不含变换，依赖于其组件的变换，具体点是根组件的变换。其他附加的组件拥有其附加到的组件的变换。
Actor及其层级的范例
- 根-SceneComponent：场景中设置Actor基础位置的基础场景组件。
- - StaticMeishComponent：表示金矿石的网格体。
- - - ParticleSystemComponent：金矿石的闪烁例子发射器
- - - AudioComponent：金矿石的金属声发射器
- - - BoxComponent：碰撞盒体，用作拾取黄金重叠事件的触发器。

## Ticking
所有Actor均可通过Tick() 函数默认被tick
ActorComponents能够默认被更新，使用TickComponent() 函数

**Tick组**

Ticking根据tick组发生。Actor或组件的tick组用于确定在帧中何时进行tick。每个tick组将完成对指定的每个actor和组件的tick，然后再开始下一个tick组。actors或组件还可以设置tick依赖性，即其他特定actor或组件的tick函数完成后他们才会进行tick。

**tick组按运行顺序排序**

| Tick组            | 引擎活动                                                     |
| ----------------- | ------------------------------------------------------------ |
| TG_PrePhysics     | 帧的开始                                                     |
| TG_DuringPhysics  | 开始物理模拟并更新引擎的物理数据。物理数据来自上一帧或当前帧 |
| TG_PostPhysics    | 此步骤开始时物理模拟已经完成，引擎使用当前帧的数据           |
| n/a               | 处理隐藏操作、tick世界时间管理器、更新摄像机、更新关卡流送体积域和流送操作 |
| TG_PostUpdateWork | n/a                                                          |
| n/a               | 处理之前在帧中创建的actor的延迟生成。完成帧并渲染            |

TG_PostPhysics 可用于武器或运动追踪。渲渲染此帧时所有物理对象将位于它们的最终位置，一帧延迟也会明显

TG_PosUpdateWork

- 在TG_PostPhysics之后运行，它的基函数是将最靠后的信息送入粒子系统
- TG_PosUpdateWork在摄像机更新后发生。如特效必须知晓摄像机朝向的准确位置，可将控制这些特效的actor放置于此
- 也可用于在帧中绝对最靠后运行的游戏逻辑

**Tick依赖性**

`AddTickPrerequisiteActor`和`AddTickPrerequisiteComponent`函数将设置存在函数调用的actor或组件等待tick

## 生命周期
实例化Actor有三种主要路径。Actor 的销毁路径都是相同的
**从磁盘加载**

已位于关卡中的Actor使用此路径，如LoadMap发生时，AddToWorld（从流关卡或子关卡）被调用时。

- 包/关卡中的Actor从磁盘中进行加载
- PostLoad - 在序列化Actor从磁盘加载完后被调用。此处可执行自定义版本化或修复操作。PostLoad 与 PoatActorCreated互斥
- InitializeActorsForPlay
- 为未初始化的Actor执行RouteActorInitialize（包含无缝行程携带）
- - PreInitializeComponents - Actor组件上调用InitializeComponent之前调用
  - InitializeComponent - Actor上定义的每个组件的创建辅助函数
  - PostInitializeComponent - Actor的组件初始化后调用
- BeginPlay - 关卡开始后调用

**play in editor**

该路径的Actor是从编辑器复制来的

- 编辑器中的Actor被复制到新场景中
- PostDuplicate 被调用
- InitializeActorsForPlay
- 为未初始化的Actor执行RouteActorInitialize（包含无缝行程携带）
- - PreInitializeComponents - Actor组件上调用InitializeComponent之前调用
  - InitializeComponent - Actor上定义的每个组件的创建辅助函数
  - PostInitializeComponent - Actor的组件初始化后调用
- BeginPlay - 关卡开始后调用

**生成**

这是生成（实例）Actor时的路径

- SpawnActor被调用
- PostSpawnInitialize
- PostActorCreated - 创建后被生成的Actor调用，发生构建函数类行为。PostLoad 与 PoatActorCreated互斥
- ExecuteConstruction
- - OnConstruction - Actor 的构建。蓝图Actor 的组件在此创建，蓝图变量在此初始化
- PostActorConstruction
- - PreInitializeComponents - Actor组件上调用InitializeComponent之前调用
  - InitializeComponent - Actor上定义的每个组件的创建辅助函数
  - PostInitializeComponent - Actor的组件初始化后调用
- OnActorSpawned 在UWorld上播放
- BeginPlay被调用

**延迟生成**

将任意属性设为“Expose on Spawn”即可延迟Actor 的生成

- SpawnActorDeferred - 生成程序化Actor，在蓝图构建脚本之前进行额外设置
- SpawnActor中所有操作发生；PostActorCreated之后发生以下操作：
- - 通过一个有效但不完整的Actor实例设置/调用多个“初始化函数”
  - FinishSpawningActor - 调用后对Actor进行最终化，在Spawn Actor行中选取ExecuteConstruction

**销毁Actor**

Actor通常不会被垃圾回收，因为场景对象保存一个Actor引用的列表。调用Destroy() 可显式销毁Actor，将其从关卡中移除并标记为“待销毁”，这说明其在下次垃圾回收中被清理之前都将存在。

**垃圾回收**

对象被标记待销毁一段时间后，垃圾回收会将其从内存中实际移除，释放资源。

销毁时调用以下函数

- BeginDestroy - 对象可利用此机会释放内存并处理其他多线程资源（即图像线程代理对象）。与销毁相关的大多数游戏性功能应该在`EndPlay`中更早地被处理。
- IsReadyForFinishDestroy - 调用此函数以确定对象是否可被永久解除分配。
- FinishDestroy - 销毁对象。



# Actor源码

