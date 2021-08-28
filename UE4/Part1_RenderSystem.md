[TOC]

# 一、UE4 的 GamePlay 架构

UObject 的整体继承关系如下：

![](./images/class_struct.png)



## 1. <span id="inherit">继承与组合关系</span>

UObject 在 Gameplay 架构里的继承关系如下：

> 查看原图更清晰

![](./images/GamePlayClass.png)

Gameplay 架构组合关系大体如下

> 查看原图更清晰

![](./images/GamePlay.jpg)



## 2. MVC 的数据处理方式

Gameplay 架构类按照 MVC 设计理念分类如下
![](./images/MVC.jpg)





# 二、UE4 的多线程

虽然 UE4 遵循 C++11 标准，但并没有使用 std::thread，而是自己实现了一套多线程机制，用法上很像 Java



## 1. 无序并行 AsyncTask 系统

FAsyncTask 模板类使用示例

```c++
// 1. 定义自己的 Task 类
class ExampleAsyncTask : public FNonAbandonableTask
{
    friend class FAsyncTask<ExampleAsyncTask>;

    int32 ExampleData;
    
    ExampleAsyncTask(int32 InExampleData): ExampleData(InExampleData) { }

    // Task 需要执行的具体执行的操作
    void DoWork() { }

    FORCEINLINE TStatId GetStatId() const
    {
        RETURN_QUICK_DECLARE_CYCLE_STAT(ExampleAsyncTask, STATGROUP_ThreadPoolAsyncTasks);
    }
};

// 2. 调用自定义的 Task 类实例
void Example()
{
    FAsyncTask<ExampleAsyncTask>* MyTask = new FAsyncTask<ExampleAsyncTask>( 5 );
    MyTask->StartBackgroundTask();
    //--or --
    MyTask->StartSynchronousTask();

    //to just do it now on this thread
    //Check if the task is done :
    if (MyTask->IsDone())
    {
    }

    //Spinning on IsDone is not acceptable( see EnsureCompletion ), but it is ok to check once a frame.
    //Ensure the task is done, doing the task on the current thread if it has not been started, waiting until completion in all cases.

    MyTask->EnsureCompletion();
    delete Task;
}
```



**异步**线程池执行的层级顺序

1. FAsyncTask 模板类实现了 IQueuedWork 的接口，将自定义的 Task 放入线程池中（多个 Task 被分配到多个线程中，**分配顺序和执行顺序无关**）
2. FQueuedThreadPool 控制多个线程资源的分配，包含多个 FQueuedThread 实例
3. FQueuedThread 控制单个线程的调度，包含 FRunnableThread
4. FRunnableThread 存储单个线程信息，包含 FRunnable
5. FRunnable 执行线程内方法，常被当作参数对象传递

> 查看原图更清晰

![](./images/thread0.jpg)



## 2. 有序并行 TaskGraph 系统

TaskGraph 使用示例

- FTaskThreadAnyThread 会将 FBaseGraphTask 按照**优先级**放到 IncomingAnyThreadTasks 数组里（这个优先级队列可以在运行时**可以修改**）
- FNamedTaskThread 会将  FBaseGraphTask 按照**优先级**放到 IncomingAnyThreadTasks 数组里（运行时**无法修改**）
- **任务依赖**：
  一个任务的执行可能依赖于多个事件对象，这些事件对象都触发之后才会执行这个任务
  而这个任务完成后，又可能触发其他事件，其他事件再进一步触发其他任务

```c++
// 1. 定义自己的 Task 类
// 如 FTickFunctionTask、FReturnGraphTask，不需要继承，要实现如下几个静态函数
class FMyTestTask
{
public:
    FMyTestTask() { }
    
    FORCEINLINE static TStatId GetStatId()
	{
		RETURN_QUICK_DECLARE_CYCLE_STAT(FMyTestTask, STATGROUP_TaskGraphTasks);
	}
    
    static const TCHAR*GetTaskName()
	{
		return TEXT("FMyTestTask");
	}
    
    // 在哪个线程上执行
	static ENamedThreads::Type GetDesiredThread()
	{
        // option:
        // - AnyThread: 无名称的任意 Task 线程
        // - StatThread、RHIThread、AudioThread、
        //   GameThread、ActualRenderingThread: 有名称的 Task 线程
		return ENamedThreads::AnyThread;
	}
	static ESubsequentsMode::Type GetSubsequentsMode()
	{
        // option:
        // - TrackSubsequents: 存在后续任务
        // - FireAndForget: 没有后续任务
		return ESubsequentsMode::TrackSubsequents;
	}

    // Task 需要执行的具体执行的操作
	void DoTask(ENamedThreads::Type CurrentThread, const FGraphEventRef& MyCompletionGraphEvent) { }
};

// 2. 调用自定义的 Task 类实例
void Example()
{
	FGraphEventRef iJoin = TGraphTask<FMyTestTask>::CreateTask(
		NULL,						// 该任务依赖事件数组
        ENamedThreads::GameThread   // 该任务需要执行的线程，默认是 AnyThread
    ).ConstructAndDispatchWhenReady();
    
    FTaskGraphInterface::Get().WaitUntilTaskCompletes(iJoin, ENamedThreads::GameThread_Local);
    
    // 如果当前的线程有 Task 任务，他就创建一个 ScopeEvent
    FScopedEvent Event;
	TriggerEventWhenTasksComplete(Event.Get(), Tasks, CurrentThreadIfKnown);
}
```



TaskGraph 执行的层级顺序

1. TGraphTask 模板类将自定义的 Task，交给 FTaskGraphInterfaceImplementation
2. FTaskGraphInterfaceImplementation 按照优先级排列 Task 后**通过事件调度** Task 到 FWorkThread 数组里
3. FWorkThread 是一个 Task（FTaskThreadBase）和 Task 运行线程（FRunnableThread）的**集合**
4. FTaskThreadBase  执行线程内方法，常被当作参数对象传递

> 查看原图更清晰

![](./images/thread1.jpg)

![](./images/thread2.jpg)



## 3. UE4 的线程调用

### 3.1 线程的执行顺序

![](./images/RenderThreads.png)

线程都附加在 TaskGraph 系统里，总体流程执行顺序：
其中 GameThread 为主线程，在 `RenderThread::StartRenderThread` 函数中先后创建 RHI 和 Render 线程

1. **GameThread（Main Thread）**
   执行 AI、碰撞、寻路、物理等游戏逻辑后通过宏定义 `ENQUEUE_RENDER_COMMAND` 生成各种 CMD（平台无关）传入渲染线程中
2. **RenderThread（渲染前端）**
   执行 `TEnqueueUniqueRenderCommandType` 等类型的 CMD
   将其转化成指定图形的调用 Graphical Command 传入 RHI 线程中（**可并行生成 CMD**）
   CMD 对象**种类不固定**的在运行时可根据不同的逻辑自定义
3. **RenderThread（渲染前端）**
4. **RHIThread（渲染后端 Render Hardware Interface）**
   执行 Graphical Command，将数据提交到 GPU 执行（**可并行处理 CMD**）
   CMD 对象**种类固定**的，通过宏定义 `FRHICOMMAND_MACRO` 声明后在调用



### 3.2 线程间的同步

GameThread 不可能领先于 RenderThread 超过一帧，否则 GameThread 会等待渲染线程处理完
**同步时机**：

- 同步仅限于 GameThread 和 RenderThread 之间，RenderThread  和 RHIThread 不需要同步
- 引擎循环的 `FEngineLoop::Tick` 末尾添加同步函数 `FFrameEndSync::Sync` 向渲染线程添加栅栏

```c++
// 渲染命令栅栏
class RENDERCORE_API FRenderCommandFence {
public:
    // 向渲染命令队列增加一个栅栏. bSyncToRHIAndGPU 是否同步 RHI 和 GPU 交换 Buffer, 否则只等待渲染线程.
    void BeginFence(bool bSyncToRHIAndGPU = false); 

    // 等待栅栏被执行. bProcessGameThreadTasks没有作用.
    void Wait(bool bProcessGameThreadTasks = false) const;

    // 是否完成了栅栏.
    bool IsFenceComplete() const;

private:
    mutable FGraphEventRef CompletionEvent; // 处理完成同步的事件
    ENamedThreads::Type TriggerThreadIndex; // 处理完之后需要触发的线程类型.
};
```



## 4. 线程间的数据交换

不同线程，同种数据的对应关系

| Game Thread            | Render Thread                              |
| :--------------------- | :----------------------------------------- |
| UWorld                 | FScene                                     |
| UPrimitiveComponent    | FPrimitiveSceneProxy / FPrimitiveSceneInfo |
| -                      | FSceneView / FViewInfo                     |
| ULocalPlayer           | FSceneViewState                            |
| ULightComponent        | FLightSceneProxy / FLightSceneInfo         |
| FVertexStreamComponent | FVertexStream / FVertexElement             |

尝试跨线程操作数据，将会引发不可预料的结果
有些对象作为线程间数据的传递者（FPrimitiveSceneProxy、FLightSceneProxy）会在**游戏线程**创建，在**渲染线程**执行，最后在**渲染线程**销毁



数据结构对比，详见 [一、1.继承与组合关系](#inherit)

```c++
// Game Thread
UWorld->ULevel->AActor->UActorComponent(UPrimitiveComponent)

// Render Thread
FSceneRenderer->FScene
    		  ->TArray<FViewInfo>

// 数据传递
// 1. UWorld -> FScene
//    UWorld 里有成员变量指针 FSceneInterface* Scene
//	  FScene 的构造函数内部总会让传入的 UWorld 对象的 Scene 指向 FScene 对象本身 

// 2. UPrimitiveComponent -> FPrimitiveSceneProxy
FPrimitiveSceneProxy::FPrimitiveSceneProxy(const UPrimitiveComponent* InComponent, FName InResourceName);
void FScene::AddPrimitive(UPrimitiveComponent* p) {
    p->CreateSceneProxy();
}
    
    
// 数据更新
// 在 UWorld 的 Tick 里，遍历所有可见的 Actor (这里用 UActorComponent 的子类 ULightComponent 来举例)
void ULightComponent::SendRenderTransform_Concurrent() {
    GetWorld()->Scene->UpdateLightTransform(this);
    Super::SendRenderTransform_Concurrent();
}
```





# 三、UE4 的渲染流程

## 1. Mesh Draw Pipeline

基本的图形 API 调用流程

1. 构造三角形顶点和索引数据
2. 创建 GPU 的资源并绑定
3. 清理图像 buffer 背景
4. 绘制三角形



商业游戏引擎在调用图形 API 前会有更多的操作

1. 遮挡剔除
2. 动态 / 静态合拼
   动态 Instance
3. 缓存状态和命令
4. 生成中间指令
5. 转译成图形 API 指令



**UE4.21 及之前**

1. 遍历场景的所有经过了可见性测试的 FPrimitiveSceneProxy 对象
2. 通过 FPrimitiveSceneProxy 收集不同的 FMeshBatch
3. 通过不同的渲染 Pass 中遍历 FMeshBatch 生成 Pass 对应的 RHICommandList 命令
4. 根据 RHICommandList 的命令调用 图形 API 指令



**UE4.22 及之后**
UE4.23 支持**移动端**的动态实例化渲染

1. 遍历场景的所有经过了可见性测试的 FPrimitiveSceneProxy 对象
2. 通过 FPrimitiveSceneProxy 收集不同的 FMeshBatch
3. **通过不同的渲染 Pass 在 FMeshPassProcessor 中遍历 FMeshBatch 生成 FMeshDrawCommand**
   在这里一个在这一步缓存静态网格的绘制命令
4. 通过不同的渲染 Pass 中生成的 FMeshDrawCommand 转换成对应的 RHICommandList 命令
5. 根据 RHICommandList 的命令调用 图形 API 指令



### 1.1 动态绘制路径

> **动 / 静态绘制并不冲突，一个 AActor 可能既有动态元素，又有静态元素**

方法：在运行阶段合并多个 Actor 作为一个资源

应用场景：一个建筑如果是一个模型的话

- 是一个 drawcall，如果分成十个模型拼组成建筑就是十个 drawcall
- 只看到这个建筑的一个像素，引擎也会把整个房子渲染一遍



UE并没有像 Unity 那样的动态合批功能，只有编辑器阶段手动合网格

- 合并的网格不能撤销，需要**谨慎操作**
- 合并的网格可以被导出到其他三维软件再次编辑

![](./images/path_dynamic.png)



### 1.2 静态绘制路径

方法：在开发阶段合并多个 Actor 作为一个资源

应用场景：

- **UInstancedStaticMeshComponent(ISM)**：静态的模型 Instance
  只有位置相异而 mesh 和材质均完全相同的物体可以合并成一个 Actor，在理想情况下只提交一次DrawCall
  ISM 好处是 DrawCall 少，坏处是 LOD 计算，裁剪和 OC 等等都是按一个对象来做，往往 ISM 的 Drawcall 减少了，但提交渲染的三角形却更多了
- **UHierarchicalInstancedStaticMeshComponent(HISM)：**基于分层实现
  一个 Mesh 有大量实例时分区域进行裁剪、计算 LOD，UE中 的植被就是 HISM 的子类
  HISM 的实现有两部分
  一部分是构建分层（自动化的，每次修改它的 instance 个数、位置等都会触发）并保存到文件中
  另一部分则是基于分层的可见性计算、LOD 和渲染组织



静态绘制路径通常可以被缓存，所以也叫缓存绘制路径，适用的对象可以是静态模型

![](./images/path_static.png)





## 2. 渲染中各个 Pass 绘制流程

1. **PrePass / Depth Only Pass**（Early Z Pass）
   使用 FDepthDrawingPolicy 策略进行绘制，只绘制 depth 到 Depth-Buffer，这个有利于减少后面的 Base pass 中的 pixel 填充，节省 pixel-shader 的执行
2. **Base pass**
   绘制不透明体和 masked material 属性的几何体，输入材质属性到 G-Buffer
   计算 Lightmap 和 sky lighting 的贡献量到 scene color buffer
3. **Issue Occlusion Queries / BeginOcclusionTests**
   执行遮挡查询（通过绘制几何体的包围盒进行 z-depth 测试）
   在绘制下一帧时，InitView 会使用这些信息进行可见性判断
4. **ShadowMap**
   针对每个光源渲染相应的 Shadowmap，光源也被累积到 translucency lighting volumes 中
5. **Lighting**（光照计算）
   分为如下子阶段:
   Pre-lighting composition lighting stage 预处理组合型光照(eg. deferred decals, SSAO)
   Render lights
6. **Draw atmosphere**
   对非透明表面绘制大气效果
7. **Draw Fog**
   针对非透明表面逐像素计算雾
8. **Draw translucency**
   在离屏渲染 RT 上（也是绘制雾的 RT）绘制半透明几何体（在单个 pass 中计算光照的最终透明度来混合）
9. **Post Processing**
   绘制 BoxBlur 等这样的后处理效果





# 四、UE4 渲染技巧

## 1. 光源



## 2. 阴影





# 五、UE4 Shader 系统

## 1. 文件库存储

UE 的内置 Shader 文件在 Engine\Shaders 目录下

- `.ush` Shader 的声明文件（头文件，可被 include，例 `#include "Common.ush"`）
- `.usf` Shader 的定义文件（不可被 include）
- `.tps` Shader 的功能介绍，证书说明文件，本质是 xml 文件



常用 USH 文件介绍

- **Platform.ush**
  定义了跟图形API（DirectX、OpenGL、Vulkan、Metal）和 FEATURE_LEVEL 相关的宏、变量及工具类接口
- **Common.ush**
  包含了图形 API 或 Feature Level 相关的宏、类型、局部变量、静态变量、基础工具接口等
- **Definitions.ush**
  预先定义了一些常见的宏，防止其它模块引用时出现语法错误
- **ShadingCommon.ush**
  定义了材质所有着色模型，并提供了少量相关的工具类接口
  其中 UE 默认的 ShadingModel ID 只占用 4bit，最多 16 个，而目前 UE 内置着色模型已占用了 13 个，意味着自定义的 ShadingModel 最多只能 3 个了
- **BasePassCommon.ush**
  定义了 BasePass 的一些变量、宏定义、插值结构体和工具类接口
- **VertexFactoryCommon.ush**
  定义了顶点变换相关的辅助接口
- **LocalVertexFactoryCommon.ush**
  顶点工厂的数据插值结构体及部分辅助接口
- **BRDF.ush**
  双向反射分布函数模块，提供了很多基础光照算法及相关辅助接口
- **ShadingModels.ush**
  着色模型以及光照计算相关的类型和辅助接口



## 2. 运行对象

**Shader Parameter**：一组由 CPU 的 C++ 层传入 GPU Shader 并存储于 GPU 寄存器或显存的数据（FRHITexture、UAV、Uniform buffer）
没有统一的父类，一般只有一层类
Parameter 类型和到 GPU 的数据类型一一对应，一个 Parameter 类型对应一种 GPU 数据

```c++
// 在 UE4.22 及以上版本
// LAYOUT_FIELD 是可以声明指定着色器参数的类型、名字、初始值、位域、写入函数等数据的宏
LAYOUT_FIELD(FShaderParameter, ShaderParam); // 等价于: FShaderParameter ShaderParam;
```



**Shader Permutation**：UE Shader 的自定义数据类型，方便将用户的自定义类型转化为 UE 的自定义类型填充到 HLSL，编译出对应的着色器代码
可以让用户定义的数据 和 Permutation 值是 1 对 多 的映射关系

```c++
// UE Shader 自己的 bool 类型
struct FShaderPermutationBool { /** static functions */ }

// 对应的宏定义声明 
// 注意宏定义里 FShaderPermutationBool 前面的 public
#define SHADER_PERMUTATION_BOOL(InDefineName) \
public FShaderPermutationBool { \
	public:\
        static constexpr const TCHAR* DefinedName = TEXT(InDefineName); \
}

// 使用时
class FDeferredLightPS : public FGlobalShader {
    // 相当于继承了全都是静态函数的 FShaderPermutationBool 类
    // class FSourceTextureDim : public FShaderPermutationBool { ... };
    class FSourceTextureDim : SHADER_PERMUTATION_BOOL("USE_SOURCE_TEXTURE");
}
```



**Uniform Buffer**：最底层的是 RHI 层的 FRHIUniformBuffer，封装了各种图形 API 的统一缓冲区（也叫 Constant Buffer）
继承自 FRenderResource 在 UE4.27 的版本相比 4.21 版本减少了许多对象

**Vertex Factory**：涉及各方面的数据和类型
继承自 FRenderResource，

- 顶点着色器
  顶点着色器的输入输出需要顶点工厂来表明数据的布局
- 顶点工厂的参数和 RHI 资源
  这些数据将从 C++ 层传入到顶点着色器中进行处理
- 顶点缓冲和顶点布局
  通过顶点布局，我们可以自定义和扩展顶点缓冲的输入，从而实现定制化 Shader 代码
- 几何预处理
  顶点缓冲、网格资源、材质参数等等都可以在真正渲染前预处理它们



**FShader**：已经编译好的着色器代码和它的参数绑定的类型
存储着 Shader 关联的绑定参数、顶点工厂、编译后的各类资源等数据，并提供了编译器修改和检测接口，其子类有

- **FGlobalShader**
  全局着色器，只有唯一的实例，常用于屏幕绘制、后处理、光照、工具类、可视化、地形、虚拟纹理等方面

- **FMaterialShader**
  材质着色器，由 FMaterialShaderType 指定的材质引用的着色器，是材质蓝图在实例化后的一个 shader 子集

- 自定义的 Shader

  ```c++
  // Shader 声明和实现宏
  // 声明指定类型（FShader子类）的 Shader, 可以是 Global, Material, MeshMaterial, ...
  #define DECLARE_SHADER_TYPE(ShaderClass,ShaderMetaTypeShortcut,...)
  // 实现指定类型的 Shader, 可以是 Global, Material, MeshMaterial, ...
  #define IMPLEMENT_SHADER_TYPE(TemplatePrefix,ShaderClass,SourceFilename,FunctionName,Frequency)
  
  // 声明 FGlobalShader 及其子类.
  #define DECLARE_GLOBAL_SHADER(ShaderClass)
  // 实现 FGlobalShader 及其子类.
  #define IMPLEMENT_GLOBAL_SHADER(ShaderClass,SourceFilename,FunctionName,Frequency)
  
  // 实现Material着色器.
  #define IMPLEMENT_MATERIAL_SHADER_TYPE(TemplatePrefix,ShaderClass,SourceFilename,FunctionName,Frequency)
  
  // 其它不常见的宏
  [......]
  
  // 例
  class FDeferredLightPS : public FGlobalShader
  {
      // 1. 在 FDeferredLightPS 类内声明全局着色器
      DECLARE_SHADER_TYPE(FDeferredLightPS, Global)
      [......]
  };
  
  // 2. 实现 FDeferredLightPS 着色器, 让它和代码文件, 主入口及着色频率关联起来.
  IMPLEMENT_GLOBAL_SHADER(FDeferredLightPS, "/Engine/Private/DeferredLightPixelShaders.usf", "DeferredLightPixelMain", SF_Pixel);
  ```

  

## 3. 缓存对象

![](./images/ShaderContainer.png)

**Shader Map**：存储编译后的 shader 代码

- FGlobalShaderMap：保存并管理着所有编译好的 FGlobalShader 代码，在 `FEngineLoop::PreInitPreStartupScreen` 初始化完成
- FMaterialShaderMap：存储和管理着一组 FMaterialShader 实例的对象
- FMeshMaterialShaderMap：存储和管理一组 FMeshMaterialShader 实例的对象





## 4. 编译流程





## 5. 开发流程

### 5.1 调试

### 5.2 优化

### 5.3 使用举例





# 引用

- [《Exploring in UE4》多线程机制详解 原理分析 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/38881269)
- [剖析虚幻渲染体系（02）- 多线程渲染 - 0向往0 - 博客园 (cnblogs.com)](https://www.cnblogs.com/timlly/p/14327537.html)
- [A new, community-hosted Unreal Engine Wiki - Announcements and Releases - Unreal Engine Forums](https://forums.unrealengine.com/t/a-new-community-hosted-unreal-engine-wiki/141494)
- [Unreal Engine 4 Materials Tutorial](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.raywenderlich.com%2F504-unreal-engine-4-materials-tutorial)
- [UE4 Instance 使用 – Cheney Shen](https://cheneyshen.com/ue4-instance-使用/)
- [UE4 Mesh Drawing pipeline official](https://docs.unrealengine.com/zh-CN/Programming/Rendering/MeshDrawingPipeline/index.html)
- [游戏程序员的自我修养-房燕梁 ](https://neil3d.github.io/unreal/mcpp-fork-join.html)

