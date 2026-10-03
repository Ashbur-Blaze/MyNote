
一、总览：C# 程序的执行链路
text
C# 源代码 (Program.cs)
    ↓ 编译（Roslyn 编译器）
IL 中间语言 + 元数据
    ↓ 打包
程序集 (.dll / .exe)
    ↓ 运行时加载
CLR（公共语言运行时）
    ↓ JIT 编译
本机机器码
    ↓
CPU 执行
二、IL（Intermediate Language，中间语言）
定义
IL 是 .NET 编译器输出的中间表示，与具体 CPU 架构无关，是一种基于栈的指令集。

特点
特点	说明
平台无关	不依赖具体 CPU，可在任何支持 CLR 的平台运行
基于栈	所有操作通过压栈/弹栈完成，没有寄存器概念
面向对象	支持类、继承、虚方法调用
强类型	每条指令都有明确的类型信息
可验证	可被 CLR 验证器检查类型安全
IL 示例
C# 代码：

csharp
int Add(int a, int b)
{
    return a + b;
}
对应 IL：

text
.method static int32 Add(int32 a, int32 b)
{
    ldarg.0        // 加载第一个参数 a 到栈
    ldarg.1        // 加载第二个参数 b 到栈
    add            // 弹出两个值，相加，结果压栈
    ret            // 返回栈顶值
}
常用 IL 指令
指令	含义
ldarg.N	加载第 N 个参数到栈
ldloc.N	加载第 N 个局部变量到栈
stloc.N	弹出栈顶，存入第 N 个局部变量
ldc.i4.N	加载常量整数 N 到栈
add / sub / mul / div	算术运算
call	调用方法
callvirt	虚方法调用
ret	返回
newobj	创建对象
box / unbox	装箱 / 拆箱
衍生概念：元数据（Metadata）
IL 不是孤立的，它和元数据一起打包在程序集中。元数据描述：

类型定义（类、接口、结构体、枚举）

成员定义（字段、方法、属性、事件）

引用（引用了哪些程序集、哪些类型）

特性（Attribute）

元数据是反射（Reflection）的基础。

衍生概念：Roslyn 编译器
Roslyn 是 C# 的官方编译器平台：

编译管线：语法分析 → 语义分析 → IL 生成

语法树（Syntax Tree）：源代码的树形表示

语义模型（Semantic Model）：类型信息、符号解析

开源：github.com/dotnet/roslyn

API 可用：可在代码中调用编译器，实现代码分析、重构工具

三、CLR（Common Language Runtime，公共语言运行时）
定义
CLR 是 .NET 的执行引擎，负责加载、编译、执行、管理托管代码。

六大职责
职责	英文	说明
程序集加载	Assembly Loading	定位、加载、验证程序集
JIT 编译	JIT Compilation	IL → 机器码
内存管理	Garbage Collection	自动回收堆对象
类型安全	Type Safety	验证 IL，防止非法访问
异常处理	Exception Handling	结构化异常处理
线程管理	Thread Management	托管线程调度
架构层次
text
.NET 应用程序
    ↓
基础类库（BCL）—— File、Console、SHA256、Dictionary
    ↓
CLR 执行引擎
├── 程序集加载器（Assembly Loader）
├── JIT 编译器（JIT Compiler）
├── 垃圾回收器（GC）
├── 类型验证器（Type Verifier）
├── 异常处理引擎（Exception Engine）
└── 线程调度器（Thread Scheduler）
    ↓
操作系统
    ↓
硬件
衍生概念：托管代码 vs 非托管代码
托管代码	非托管代码
运行环境	CLR	直接由 OS 执行
内存管理	GC	手动 malloc / free
类型安全	验证器保证	无保证
例子	C#、F#、VB.NET	C、C++、Rust
互操作	通过 P/Invoke、COM、FFI	通过导出函数
衍生概念：程序集（Assembly）
程序集是 .NET 的部署和版本控制单元，包含：

IL 代码

元数据

清单（Manifest）：程序集名称、版本、文化、公钥令牌、依赖列表

资源（可选）

类型：

类型	扩展名	说明
可执行程序集	.exe	有入口点
库程序集	.dll	无入口点，被引用
衍生概念：应用程序域（AppDomain）
.NET Framework 中的隔离单元，一个进程可有多个 AppDomain。

.NET Core / .NET 5+ 中已弱化，通常一个进程一个 AppDomain。

用途：隔离、卸载程序集（.NET Core 不再支持卸载单个程序集）。

四、JIT（Just-In-Time Compilation，即时编译）
定义
JIT 是 CLR 在运行时将 IL 编译为目标平台机器码的编译器。

编译流程
text
IL
    ↓ 导入（Import）
内部中间表示（IR）
    ↓ 优化（Optimize）
优化后的 IR
    ↓ 寄存器分配（Register Allocation）
    ↓ 代码生成（Code Generation）
机器码
关键机制
机制	说明
方法级编译	以方法为单元，首次调用时编译
延迟编译	用到了才编译，不预先全编译
编译缓存	编译过的机器码缓存，不重复编译
分层编译	Tier 0（快速编译）→ Tier 1（优化编译）
内联	小方法嵌入调用点，消除调用开销
去虚拟化	虚调用转直接调用
SIMD	向量化指令生成
循环优化	循环展开、不变量外提
分层编译（Tiered Compilation）
.NET Core 3.0+ 默认启用：

text
第 1 次调用方法
    ↓
Tier 0：快速编译，不优化，立即执行
    ↓
方法被多次调用（热点方法）
    ↓
Tier 1：重新编译，深度优化
    ↓
后续调用使用优化后的机器码
目的：启动快（Tier 0 编译快）+ 运行快（Tier 1 优化好）。

衍生概念：AOT（Ahead-Of-Time Compilation，提前编译）
JIT	AOT
编译时机	运行时	发布时
启动速度	较慢（需预热）	快
运行速度	优化后快	快
跨平台	IL 跨平台	每平台单独编译
反射支持	完整	受限
例子	默认模式	Native AOT、ReadyToRun
Native AOT：.NET 7+ 支持，直接把 C# 编译成本机可执行文件，不依赖 CLR 运行时。

衍生概念：ReadyToRun (R2R)
发布时预编译 IL 为机器码，减少 JIT 开销。

但保留 IL，运行时仍可 JIT（用于不支持预编译的场景）。

折中方案：启动比纯 JIT 快，比 Native AOT 灵活。

五、栈（Stack）
定义
栈是线程私有的内存区域，后进先出（LIFO），用于存储方法调用的上下文和局部变量。

特点
特点	说明
线程私有	每个线程有自己的栈
大小固定	通常 1 MB（Windows 默认）
分配速度快	移动栈指针即可
自动管理	方法退出时自动释放
后进先出	最后压入的最先弹出
存储内容	值类型局部变量、方法参数、返回地址、引用类型的引用
栈帧（Stack Frame）
每次方法调用，在栈上分配一个栈帧，包含：

方法参数

局部变量

返回地址

保存的寄存器状态

text
高地址
┌─────────────────┐
│  Main 的栈帧     │
├─────────────────┤
│  Add 的栈帧      │  ← 当前栈顶
│  ├── 参数 x, y  │
│  ├── 局部 sum   │
│  └── 返回地址   │
└─────────────────┘
低地址
栈溢出（Stack Overflow）
递归无终止条件 → 栈帧无限累积 → 超过栈大小 → StackOverflowException

栈溢出无法被 catch，进程直接终止。

大数组、大结构体作为局部变量也可能导致栈溢出。

衍生概念：调用约定（Calling Convention）
方法调用时，参数如何传递、返回值如何返回、栈由谁清理：

约定	说明
cdecl	调用方清理栈，C/C++ 默认
stdcall	被调用方清理栈，Win32 API
fastcall	部分参数用寄存器
thiscall	C++ 成员函数
CLR 约定	由 CLR 统一管理，开发者无需关心
六、堆（Heap）
定义
堆是进程共享的内存区域，用于存储引用类型的对象，由 GC 自动管理。

特点
特点	说明
进程共享	所有线程共享同一个堆
大小大	受物理内存和虚拟内存限制
分配速度慢	需查找空闲块
自动管理	GC 回收不再使用的对象
碎片化	长时间运行可能产生内存碎片
存储内容	引用类型对象、装箱后的值类型
托管堆的结构
text
托管堆
├── Gen 0（第 0 代）—— 新分配的小对象
├── Gen 1（第 1 代）—— 从 Gen 0 存活下来的
├── Gen 2（第 2 代）—— 长命对象
└── LOH（大对象堆）—— 大于 85KB 的对象
对象在堆上的布局
text
┌──────────────────┐
│  对象头（Header）  │  ← 同步块索引、方法表指针
├──────────────────┤
│  字段数据          │  ← 实例字段
├──────────────────┤
│  （对齐填充）      │
└──────────────────┘
对象头包含：

同步块索引（SyncBlock Index）：用于锁、哈希码缓存

方法表指针（Method Table Pointer）：指向类型的方法表，用于虚方法分派

衍生概念：GC（Garbage Collection，垃圾回收）
分代、标记-清除、压缩式回收器：

阶段	说明
标记（Mark）	从根（栈、静态字段、寄存器）出发，标记可达对象
清除（Sweep）	回收未标记对象
压缩（Compact）	移动存活对象，消除碎片
分代假设：

大部分对象很快死亡（Gen 0）

存活越久的对象越可能继续存活（Gen 2）

GC 触发条件：

Gen 0 满

显式调用 GC.Collect()

系统内存压力

衍生概念：大对象堆（LOH）
大于 85,000 字节的对象分配在 LOH。

LOH 不压缩（移动大对象代价高）。

LOH 碎片化是长期运行应用的常见问题。

.NET Core 支持 GCSettings.LargeObjectHeapCompactionMode 手动压缩。

衍生概念：写屏障（Write Barrier）
维护跨代引用：Gen 2 对象引用 Gen 0 对象。

GC 标记时，需检查跨代引用，写屏障在引用赋值时记录。

保证 GC 正确性。

衍生概念：终结器（Finalizer）
csharp
class MyClass
{
    ~MyClass()   // 终结器
    {
        // 释放非托管资源
    }
}
由 GC 在回收对象前调用。

调用时机不确定。

推荐用 IDisposable + using 替代。

衍生概念：IDisposable 与 using
csharp
using FileStream stream = File.OpenRead(path);
// 离开作用域时自动调用 stream.Dispose()
IDisposable 接口定义 Dispose() 方法。

using 确保 Dispose() 在离开作用域时被调用。

用于释放非托管资源（文件句柄、网络连接、加密上下文）。

衍生概念：装箱与拆箱（Boxing / Unboxing）
装箱：值类型 → 引用类型

csharp
int x = 42;
object obj = x;   // 装箱：在堆上分配，复制 x 的值
拆箱：引用类型 → 值类型

csharp
int y = (int)obj;   // 拆箱：从堆上复制值到栈
代价：

装箱：堆分配 + 内存复制

拆箱：类型检查 + 内存复制

避免装箱：使用泛型（List<int> 而不是 ArrayList）。

七、栈 vs 堆 对比
维度	栈	堆
归属	线程私有	进程共享
大小	1 MB（默认）	受物理内存限制
分配速度	极快（移动指针）	较慢（查找空闲块）
释放方式	自动（函数退出）	GC 回收
顺序	LIFO	无顺序
碎片	无	可能碎片化
存储	值类型局部变量、参数、返回地址	引用类型对象、装箱值
溢出	StackOverflowException	OutOfMemoryException
变量在栈/堆的分布
csharp
int count = 0;                              // count 在栈上
string path = args[0];                      // path 引用在栈上，字符串对象在堆上
Dictionary<string, string> dict = new ...;  // dict 引用在栈上，字典对象在堆上
图示：

text
栈                          堆
┌──────────┐
│ count: 0 │
├──────────┤
│ path: ●──┼──────────→ "C:\test"
├──────────┤
│ dict: ●──┼──────────→ { Dictionary 对象 }
└──────────┘
八、衍生知识点汇总
概念	一句话	关联
Roslyn	C# 编译器平台	生成 IL
元数据	描述类型的二进制数据	反射基础
程序集	.NET 部署单元	含 IL + 元数据
AppDomain	隔离单元	.NET Framework 遗留
托管代码	由 CLR 管理	对比非托管
AOT	提前编译	对比 JIT
ReadyToRun	预编译 IL	折中方案
分层编译	Tier 0 → Tier 1	JIT 优化策略
内联	小方法嵌入	JIT 优化
去虚拟化	虚调用转直接	JIT 优化
栈帧	方法调用上下文	栈的单位
调用约定	参数传递规则	底层 ABI
GC	垃圾回收	堆管理
分代	Gen 0/1/2	GC 模型
LOH	大对象堆	> 85KB
写屏障	维护跨代引用	GC 正确性
终结器	析构函数	非托管资源
IDisposable	释放接口	using
装箱/拆箱	值↔引用转换	性能陷阱
强名称	程序集签名	防篡改
延迟加载	用时才加载	程序集加载
九、Rust 对照
概念	C#	Rust
编译目标	IL → JIT → 机器码	直接机器码
运行时	CLR	无（或极小）
内存管理	GC	所有权 + 借用
栈	有	有
堆	有（GC 管理）	有（所有权管理）
装箱	值→引用	Box<T>
接口	interface	trait
空值	null	Option<T>
错误	异常	Result<T, E>
启动速度	较慢	极快
运行时开销	有	无