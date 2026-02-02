# Unity AI 技能体系

> 版本: 1.0.0
> 为 Unity 客户端开发工程师定制的 AI 技能使用指南

---

## 一、技能体系概览

```
┌─────────────────────────────────────────────────────────────┐
│                    Unity 开发 AI 技能体系                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Level 1   │  │   Level 2   │  │   Level 3   │         │
│  │   基础层    │  │   架构层    │  │   专业层    │         │
│  ├─────────────┤  ├─────────────┤  ├─────────────┤         │
│  │ fundamentals│  │ architecture│  │  rendering  │         │
│  │ performance │  │  workflows  │  │  networking │         │
│  │  csharp     │  │   editor    │  │    dots     │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                    工具层 (自动触发)                  │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │  code-reviewer  │  generate-script  │  editor-tool  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 二、Level 1: 基础技能 (高频使用)

### 2.1 unity-fundamentals (Unity 基础)

**触发关键词**: 生命周期、Awake、Start、Update、序列化、SerializeField、Prefab、组件

**核心能力**:
- MonoBehaviour 生命周期方法使用规范
- 序列化系统与 Inspector 集成
- 组件架构与 GetComponent 模式
- Prefab 工作流与变体系统

**典型场景**:
```
Q: "Awake 和 Start 应该怎么用?"
A: 使用 unity-fundamentals 技能
   - Awake: 缓存自身组件、初始化内部状态
   - Start: 引用其他对象、注册到管理器

Q: "为什么我的私有字段在 Inspector 不显示?"
A: 使用 unity-fundamentals 技能
   - 需要添加 [SerializeField] 特性
   - 属性默认不序列化
```

**关键知识点**:
```csharp
// 生命周期顺序
Awake() → OnEnable() → Start() → FixedUpdate() → Update() → LateUpdate() → OnDisable() → OnDestroy()

// 序列化规则
[SerializeField] private int _health;  // ✅ Inspector显示
public int health;                      // ✅ Inspector显示(不推荐)
public int Health { get; set; }        // ❌ 不显示
private int _internal;                  // ❌ 不显示

// 组件缓存
private Rigidbody _rb;
void Awake() { _rb = GetComponent<Rigidbody>(); }
```

---

### 2.2 unity-performance (性能优化)

**触发关键词**: 性能、优化、GC、内存、对象池、Profiler、卡顿、帧率

**核心能力**:
- 引用缓存与查找优化
- 对象池实现与使用
- GC 分配减少策略
- Profiler 使用与瓶颈分析

**典型场景**:
```
Q: "游戏运行一段时间后卡顿"
A: 使用 unity-performance 技能
   - 打开 Profiler 检查 GC Alloc
   - 检查 Update 中是否有 new 分配
   - 检查是否有未停止的协程

Q: "如何实现子弹对象池?"
A: 使用 unity-performance 技能
   - 提供 ObjectPool<T> 泛型实现
   - 包含预热、获取、归还逻辑
```

**关键知识点**:
```csharp
// 缓存优先级 (从高到低)
1. Inspector引用 > 2. Awake缓存 > 3. 延迟查找

// 对象池核心API
var bullet = _pool.Get();       // 获取
_pool.Return(bullet);           // 归还
_pool.ReturnAll();              // 全部归还

// GC避免清单
- 缓存 WaitForSeconds
- 复用 List/Array
- 避免 Update 中字符串拼接
- 避免 LINQ 在热路径
```

---

### 2.3 csharp-advanced (C# 高级特性)

**触发关键词**: C#、async、await、LINQ、泛型、委托、表达式、模式匹配

**核心能力**:
- 现代 C# 语法 (C# 9/10/11)
- async/await 异步编程
- LINQ 查询优化
- 泛型与约束

**典型场景**:
```
Q: "如何在 Unity 中正确使用 async/await?"
A: 使用 csharp-advanced 技能
   - 推荐使用 UniTask 而非原生 Task
   - 必须处理 CancellationToken
   - 组件销毁时取消异步操作

Q: "Record 类型在 Unity 中能用吗?"
A: 使用 csharp-advanced 技能
   - Unity 2021+ 支持 C# 9 Record
   - 注意: Record 不可序列化到 Inspector
```

**关键知识点**:
```csharp
// 模式匹配
if (obj is Enemy { Health: > 0 } enemy) { }

// 空值处理
var name = player?.GetName() ?? "Unknown";

// UniTask 模式
async UniTaskVoid Start()
{
    await LoadAsync(this.GetCancellationTokenOnDestroy());
}
```

---

## 三、Level 2: 架构技能 (中频使用)

### 3.1 unity-architecture (游戏架构)

**触发关键词**: 架构、设计模式、单例、事件系统、ScriptableObject、Manager、MVC

**核心能力**:
- Manager 模式与单例实现
- ScriptableObject 数据架构
- 事件系统设计 (C# Event / UnityEvent / EventBus)
- 组件通信与解耦

**典型场景**:
```
Q: "如何设计游戏管理器系统?"
A: 使用 unity-architecture 技能
   - 提供泛型单例基类
   - Manager初始化顺序控制
   - 服务定位器模式作为替代

Q: "如何实现全局事件系统?"
A: 使用 unity-architecture 技能
   - 提供类型安全的 EventBus<T>
   - ScriptableObject 事件方案
   - 订阅/取消订阅最佳实践
```

**关键知识点**:
```csharp
// 单例基类
public class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    public static T Instance { get; private set; }
    protected virtual void Awake()
    {
        if (Instance != null) { Destroy(gameObject); return; }
        Instance = this as T;
        DontDestroyOnLoad(gameObject);
    }
}

// 事件总线
EventBus<PlayerDiedEvent>.Subscribe(OnPlayerDied);
EventBus<PlayerDiedEvent>.Unsubscribe(OnPlayerDied);
EventBus<PlayerDiedEvent>.Publish(new PlayerDiedEvent { });

// ScriptableObject 数据
[CreateAssetMenu] public class WeaponData : ScriptableObject { }
```

---

### 3.2 unity-workflows (开发工作流)

**触发关键词**: 编辑器、EditorWindow、CustomEditor、Inspector、Input System、UI Toolkit

**核心能力**:
- 编辑器脚本开发
- 自定义 Inspector
- 新版 Input System
- UI 系统 (uGUI / UI Toolkit)

**典型场景**:
```
Q: "如何创建自定义编辑器窗口?"
A: 使用 unity-workflows 技能
   - EditorWindow 模板
   - MenuItem 注册
   - GUI/GUILayout 使用

Q: "新版 Input System 怎么用?"
A: 使用 unity-workflows 技能
   - Input Actions 配置
   - PlayerInput 组件
   - 代码读取输入值
```

**关键知识点**:
```csharp
// EditorWindow
public class MyWindow : EditorWindow
{
    [MenuItem("Tools/My Window")]
    static void ShowWindow() => GetWindow<MyWindow>();
    void OnGUI() { /* GUI绘制 */ }
}

// CustomEditor
[CustomEditor(typeof(MyScript))]
public class MyScriptEditor : Editor
{
    public override void OnInspectorGUI() { }
}

// Input System
var moveInput = _inputActions.Player.Move.ReadValue<Vector2>();
```

---

### 3.3 unity-editor (编辑器扩展)

**触发关键词**: 编辑器工具、自动化、批处理、资源处理、AssetDatabase、Undo

**核心能力**:
- PropertyDrawer 自定义
- 资源批处理工具
- 场景工具 (Handles/Gizmos)
- 构建管线扩展

**典型场景**:
```
Q: "如何批量修改所有预制体?"
A: 使用 unity-editor 技能
   - AssetDatabase.FindAssets
   - PrefabUtility API
   - Undo 支持

Q: "如何在 Scene 视图绘制辅助线?"
A: 使用 unity-editor 技能
   - OnDrawGizmos
   - Handles API
   - SceneView.duringSceneGui
```

---

## 四、Level 3: 专业技能 (按需使用)

### 4.1 unity-rendering (渲染专业)

**触发关键词**: 渲染、Shader、ShaderGraph、URP、HDRP、材质、后处理

**核心能力**:
- URP/HDRP 管线配置
- Shader Graph 节点
- 自定义渲染特性
- 后处理效果

---

### 4.2 unity-networking (网络专业)

**触发关键词**: 网络、多人、Netcode、同步、服务器、客户端

**核心能力**:
- Netcode for GameObjects
- 状态同步与预测
- RPC 通信
- 大厅与匹配

---

### 4.3 unity-dots (DOTS专业)

**触发关键词**: DOTS、ECS、Job System、Burst、Entity

**核心能力**:
- Entity Component System
- Job System 多线程
- Burst 编译器优化
- 混合 DOTS 模式

---

## 五、工具层技能

### 5.1 code-reviewer (代码审查代理)

**自动触发**: 完成代码编写后

**审查内容**:
- 生命周期方法使用
- 性能反模式检测
- 序列化最佳实践
- 事件订阅配对检查

**输出格式**:
```
## Unity Code Review: PlayerController.cs

### ✅ Good Practices
- 使用 [SerializeField] private
- 组件引用已缓存

### ⚠️ Issues Found
**Performance - Critical**
- Line 23: GetComponent in Update
  - Fix: Cache in Awake

### Priority
- 🔴 Critical: 1 issue
- 🟡 Important: 0 issues
```

---

### 5.2 generate-script (脚本生成)

**使用方式**: `/generate-script [类型] [名称]`

**支持类型**:
- MonoBehaviour - 标准组件
- ScriptableObject - 数据资产
- EditorWindow - 编辑器窗口
- CustomEditor - 自定义 Inspector
- PropertyDrawer - 属性绘制器

**示例**:
```
/generate-script MonoBehaviour PlayerController
/generate-script ScriptableObject WeaponData
/generate-script EditorWindow LevelEditor
```

---

### 5.3 create-editor-tool (编辑器工具)

**使用方式**: `/create-editor-tool [类型] [名称]`

**支持类型**:
- EditorWindow - 独立窗口
- CustomEditor - Inspector 扩展
- PropertyDrawer - 属性显示
- MenuItem - 菜单命令
- SceneGUI - 场景叠加层

---

## 六、技能选择决策树

```
开始
  │
  ▼
┌─────────────────────────────┐
│ 这是什么类型的问题?          │
└─────────────────────────────┘
  │
  ├─[代码基础/生命周期/序列化]
  │     └──▶ unity-fundamentals
  │
  ├─[性能问题/卡顿/GC]
  │     └──▶ unity-performance
  │
  ├─[C#语法/异步/泛型]
  │     └──▶ csharp-advanced
  │
  ├─[架构设计/设计模式]
  │     └──▶ unity-architecture
  │
  ├─[编辑器工具/Inspector]
  │     └──▶ unity-workflows / unity-editor
  │
  ├─[渲染/Shader/光照]
  │     └──▶ unity-rendering
  │
  ├─[网络/多人游戏]
  │     └──▶ unity-networking
  │
  └─[高性能计算/ECS]
        └──▶ unity-dots
```

---

## 七、场景-技能映射表

| 开发场景 | 首选技能 | 辅助技能 | 工具 |
|---------|---------|---------|------|
| 新建 MonoBehaviour | fundamentals | - | generate-script |
| 新建 ScriptableObject | architecture | - | generate-script |
| 新建编辑器工具 | workflows | editor | create-editor-tool |
| 性能分析优化 | performance | - | code-reviewer |
| 架构重构 | architecture | fundamentals | - |
| 对象池实现 | performance | architecture | - |
| 事件系统设计 | architecture | - | - |
| UI 开发 | workflows | performance | - |
| 输入系统 | workflows | - | - |
| 代码审查 | - | - | code-reviewer |

---

## 八、技能触发词速查

### Level 1 触发词

```
unity-fundamentals:
  Awake, Start, Update, OnEnable, OnDisable, 生命周期
  SerializeField, 序列化, Inspector, Header
  GetComponent, 组件, RequireComponent
  Prefab, 预制体, Instantiate

unity-performance:
  性能, 优化, GC, 内存, 卡顿, 帧率
  对象池, Pool, 缓存, Cache
  Profiler, 分析, 瓶颈
  Update优化, 协程优化

csharp-advanced:
  async, await, Task, UniTask
  LINQ, Where, Select, OrderBy
  泛型, Generic, 约束
  委托, Action, Func, 事件
  模式匹配, switch表达式, Record
```

### Level 2 触发词

```
unity-architecture:
  架构, 设计模式, Pattern
  单例, Singleton, Manager
  ScriptableObject, SO, 数据驱动
  事件, Event, EventBus, 解耦
  MVC, MVP, 状态机

unity-workflows:
  编辑器, Editor, EditorWindow
  CustomEditor, Inspector, PropertyDrawer
  Input System, 输入, 按键
  UI, Canvas, UI Toolkit, UGUI

unity-editor:
  MenuItem, 菜单, 快捷键
  AssetDatabase, 资源处理
  Handles, Gizmos, Scene视图
  Undo, 撤销, 编辑器工具
```

### Level 3 触发词

```
unity-rendering:
  渲染, Shader, Material
  URP, HDRP, 渲染管线
  ShaderGraph, 节点, 效果
  后处理, Post Processing

unity-networking:
  网络, Netcode, 多人
  同步, RPC, 服务器
  客户端, 预测, 补偿

unity-dots:
  DOTS, ECS, Entity
  Job System, 多线程
  Burst, 编译, 优化
```

---

## 九、最佳实践示例

### 示例1: 新功能开发流程

```
需求: 实现玩家受伤系统

1. 架构设计 (unity-architecture)
   - 设计 IDamageable 接口
   - 设计 Health 组件
   - 设计伤害事件

2. 代码实现 (unity-fundamentals)
   - 创建 Health.cs MonoBehaviour
   - 实现生命周期方法
   - 添加序列化字段

3. 性能考虑 (unity-performance)
   - 事件使用 struct 避免 GC
   - 伤害数字使用对象池

4. 代码审查 (code-reviewer)
   - 检查事件订阅配对
   - 检查组件缓存
```

### 示例2: 性能问题排查流程

```
问题: 游戏运行一段时间后掉帧

1. 数据收集 (unity-performance)
   - 打开 Profiler
   - 记录 GC Alloc 峰值
   - 定位问题脚本

2. 问题分析 (unity-performance)
   - 检查 Update 中的分配
   - 检查未停止的协程
   - 检查事件泄漏

3. 解决方案 (unity-performance + unity-fundamentals)
   - 实现对象池
   - 缓存 WaitForSeconds
   - 修复事件取消订阅

4. 验证 (unity-performance)
   - 再次 Profile
   - 确认 GC Alloc 减少
```

---

## 十、技能扩展建议

### 建议新增技能

1. **unity-testing** (测试技能)
   - Unity Test Framework
   - PlayMode / EditMode 测试
   - Mock 模式

2. **unity-cicd** (CI/CD 技能)
   - Unity Cloud Build
   - 自动化构建脚本
   - 多平台发布

3. **unity-mobile** (移动平台技能)
   - iOS/Android 优化
   - 触控输入处理
   - 平台特定 API

4. **unity-audio** (音频技能)
   - Audio Mixer
   - 3D 空间音频
   - 音频优化
