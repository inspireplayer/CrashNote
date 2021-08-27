[TOC]

# 一、粒子系统

UE4 有两种粒子系统

- Cascade 粒子系统
  通过 CPU 主线程来计算粒子属性值
  通过 Level Of Detail（LOD）让远处的粒子配置的简洁，近处的粒子配置的复杂的方法来降低粒子的功耗
- Niagara 粒子系统



## 1. 编辑器配置

### 1.1 普通粒子

普通的粒子指的多个粒子发射器的集合，其中一个粒子发射器有以下模块：

- **Required**（默认模块，无法删除）
  包含了一些属性，都是对粒子系统绝对需要用到的属性，比如粒子使用的材质，发射器发射粒子的时间，以及其他
- **Spawn**（默认模块，无法删除）
  这个模块控制粒子从发射器生成的速度（个/秒）= Rate * Rate Scale，它们是否以 Burst 生成（爆破形式），以及其他和粒子发生时机有关的属性
- **Lifetime**
  定义了每个粒子在生成后存在的时间（单位：秒），**决定了粒子的最终长度**
- **发射器类型数据模块**（只能有一个，不能重复添加）
  发射器默认都是面片发射器
  可以不使用该模块



### 1.2 Ribbon 条带粒子

> 条带粒子的发射源头可能不止一个，因此其发射配置不能使用默认的 Spawn 模块（由于默认模块无法删除，通过将发射速率设置 0 来关闭 Spawn 模块）

**Trail/Source 模块**：指定条带粒子发射器的位置配置

- PET2SRCM Default 粒子发射器位置**实时跟随**粒子组件 attach 的组件上
- PET2SRCM Particle 粒子发射器位置**实时跟随**另一个粒子发射器产生的粒子
- PET2SRCM Actor 粒子发射器位置**实时跟随** Actor 对象



**Spawn PerUnit 模块**：Source 模块为 **PET2SRCM Default** 模式下配置有效
用于物体**只有移动时**才会产生拖尾的效果

- Unit Scalar 每秒条带的长度
- Spawn Per Unit 每秒粒子发射数
- 每个条带每秒内每单位位移产生的粒子个数：Spawn Per Unit / Unit Scalar，**会使发射器移动速度影响条带的粗细**
  如果每个条带每秒内每单位位移产生粒子数过多，每个条带每秒产生的粒子总数是不变的，这会让粒子在**位移的时**产生的条带过细



**Ribbon Data 模块**：装配该数据模块的发射器在<u>移动</u>后会形成一个条带轨迹

- Sheets Per Trail 围绕条带延伸方向旋转进行渲染的面片数量
- Max Trail Count 每秒允许存活条带的数量
- Max Particle In Trail Count 所有条带总共包含的最大粒子数量，**决定每秒条带包含的固定粒子数**
  Max Particle In Trail Count = Active Particle In Trail Count + Spawn Per Unit / Unit Scalar
  会销毁过多的 Active 粒子来确保当前的 Max Particle In Trail Count 不变



## 2. 内存结构 Freelist

粒子系统采用 Freelist 的内存结构，用链表的方式存储粒子内存池的当前活动索引
避免了大量粒子产生和销毁的过程中频繁的进行内存开辟和释放

每个拖尾粒子对象都会有一个 int32 的 Flags 成员变量，它包含了在同一个尾迹内

1. 下一个拖尾粒子的索引 [13 - 0 bits] 
2. 上一个拖尾粒子的索引 [27 - 14 bits] 
3. 当前粒子的标志类型 [31 - 28 bits]



粒子对象和索引数组

- 运行时历史最长数组作为数组的最终长度
- 采用标记删除法，将删除的对象移动到数组最后，通过 ActiveParticles 来说明当前存储长度
- Start 索引标记：条带粒子的开始，将会一直指向新产生的粒子
- End 索引标记：条带粒子的结束，将会一直指向即将清除的粒子



# 二、动画系统







# 引用

- [粒子系统的关键概念 | 虚幻引擎文档 (unrealengine.com)](https://docs.unrealengine.com/4.26/zh-CN/RenderingAndGraphics/ParticleSystems/Overview/)
- [UE4：Niagara使用基础要点 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/338810919)
- [UE4官方直播学习记录\]Niagara基础1 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/90188073)

