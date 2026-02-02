# Unity 客户端开发 AI 辅助规则库

> 为 Unity 客户端开发工程师定制的 AI 编程辅助规则、红线与技能体系

---

## 目录结构

```
C#AndUnity/
├── README.md                           # 本文件 - 快速索引
├── rules/
│   └── UNITY_DEVELOPMENT_RULES.md      # Unity 开发规则与最佳实践
├── redlines/
│   └── UNITY_REDLINES.md               # 性能红线与禁止事项清单
├── skills/
│   └── UNITY_SKILLS.md                 # AI 技能体系与使用指南
├── templates/
│   └── UNITY_CODE_TEMPLATES.md         # 代码模板库
└── checklists/
    └── UNITY_CHECKLISTS.md             # 开发检查清单
```

---

## 快速导航

### 我要写代码
→ [代码模板](templates/UNITY_CODE_TEMPLATES.md)
- MonoBehaviour 标准模板
- 单例管理器模板
- ScriptableObject 模板
- 对象池模板
- 事件系统模板
- 编辑器工具模板

### 我要检查代码
→ [开发规则](rules/UNITY_DEVELOPMENT_RULES.md)
- 命名规范
- 代码组织
- 生命周期规范
- 序列化规范
- 组件引用规范

### 我要避免踩坑
→ [红线清单](redlines/UNITY_REDLINES.md)
- P0 致命红线 (必须避免)
- P1 严重红线 (应当避免)
- P2 重要红线 (建议避免)
- P3 建议红线 (最佳实践)

### 我要使用 AI 技能
→ [技能体系](skills/UNITY_SKILLS.md)
- 技能选择决策树
- 触发关键词速查
- 场景-技能映射表

### 我要提交/发布
→ [检查清单](checklists/UNITY_CHECKLISTS.md)
- 代码提交检查清单
- 性能优化检查清单
- 发布前检查清单

---

## 核心红线速记 (Top 10)

| 序号 | 红线 | 严重级别 |
|-----|------|---------|
| 1 | Update 中 GetComponent | P0 |
| 2 | Update 中 Find 方法 | P0 |
| 3 | Update 中字符串拼接 | P0 |
| 4 | Update 中 new 分配 | P0 |
| 5 | Material 泄漏 (.material 未销毁) | P0 |
| 6 | 事件订阅未取消 | P0 |
| 7 | 静态集合持有场景对象 | P0 |
| 8 | 异步回调访问已销毁对象 | P0 |
| 9 | 编辑器代码未条件编译 | P0 |
| 10 | 空的 Update 方法 | P1 |

---

## AI 技能快速选择

```
问题类型               → 使用技能
────────────────────────────────
生命周期/序列化/组件    → unity-fundamentals
性能问题/GC/优化        → unity-performance  
架构设计/设计模式       → unity-architecture
编辑器工具/UI/输入      → unity-workflows
C# 高级特性            → csharp-advanced
渲染/Shader            → unity-rendering
网络/多人游戏          → unity-networking
────────────────────────────────
需要脚本模板           → /generate-script
需要编辑器工具模板     → /create-editor-tool
代码审查               → unity-code-reviewer
```

---

## 最佳实践速查

### 组件缓存
```csharp
// ✅ 正确
private Rigidbody _rb;
void Awake() { _rb = GetComponent<Rigidbody>(); }
void Update() { _rb.velocity = Vector3.forward; }

// ❌ 错误
void Update() { GetComponent<Rigidbody>().velocity = Vector3.forward; }
```

### 事件订阅
```csharp
// ✅ 正确
void OnEnable() { GameEvents.OnDied += HandleDeath; }
void OnDisable() { GameEvents.OnDied -= HandleDeath; }

// ❌ 错误 (缺少 OnDisable)
void OnEnable() { GameEvents.OnDied += HandleDeath; }
```

### 序列化
```csharp
// ✅ 正确
[SerializeField] private int _health = 100;
public int Health => _health;

// ❌ 错误
public int health = 100;
```

### 协程
```csharp
// ✅ 正确
private readonly WaitForSeconds _wait = new WaitForSeconds(1f);
IEnumerator Routine() { yield return _wait; }

// ❌ 错误
IEnumerator Routine() { yield return new WaitForSeconds(1f); }
```

---

## 文件版本

| 文件 | 版本 | 更新日期 |
|-----|------|---------|
| UNITY_DEVELOPMENT_RULES.md | 1.0.0 | 2024 |
| UNITY_REDLINES.md | 1.0.0 | 2024 |
| UNITY_SKILLS.md | 1.0.0 | 2024 |
| UNITY_CODE_TEMPLATES.md | 1.0.0 | 2024 |
| UNITY_CHECKLISTS.md | 1.0.0 | 2024 |

---

## 配合现有技能使用

本规则库设计为与以下现有 AI 技能配合使用:

1. **dotnet-architect** - C#/.NET 高级特性
2. **unity-developer** - Unity 综合开发
3. **Claude_Unity_dev_plugin** - Unity 开发插件
   - unity-fundamentals
   - unity-performance
   - unity-architecture
   - unity-workflows

建议工作流:
```
1. 需求分析 → unity-developer (宏观方案)
2. 架构设计 → unity-architecture (设计模式)
3. 代码实现 → unity-fundamentals + 本规则库
4. 性能优化 → unity-performance + 红线检查
5. 代码审查 → unity-code-reviewer + 检查清单
```

---

## 反馈与贡献

如发现规则遗漏或有优化建议，请更新对应文档。
