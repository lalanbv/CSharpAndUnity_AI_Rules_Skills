# Unity 客户端开发全局规则

> 此文件为全局配置，对所有 Qoder 会话生效

## 角色定位

你正在协助一位 **Unity 客户端开发工程师**，主要工作包括:
- C# 游戏业务逻辑开发
- Unity 编辑器工具开发
- Unity 插件开发
- 性能优化与架构设计

## 核心红线 (绝对禁止)

### P0 - 致命级别
1. **禁止** 在 Update/FixedUpdate/LateUpdate 中调用 GetComponent
2. **禁止** 在 Update 中使用 Find/FindObjectOfType 方法
3. **禁止** 在 Update 中进行字符串拼接或 new 分配
4. **禁止** 访问 .material 后不在 OnDestroy 中销毁
5. **禁止** OnEnable 中订阅事件但 OnDisable 中不取消
6. **禁止** 静态集合持有场景对象引用而不在 OnDestroy 中移除
7. **禁止** 编辑器代码不使用 #if UNITY_EDITOR 包裹

### P1 - 严重级别
1. **避免** 空的 Update/FixedUpdate/LateUpdate 方法
2. **避免** 在热路径中使用 LINQ
3. **避免** 每帧访问 Camera.main (应缓存)
4. **避免** 协程中每次 new WaitForSeconds (应缓存)

## 代码规范强制要求

### 命名
- 类/方法/属性: PascalCase
- 私有字段: _camelCase (下划线前缀)
- 参数/局部变量: camelCase

### 序列化
```csharp
// 必须使用
[SerializeField] private int _health = 100;
public int Health => _health;

// 禁止使用
public int health = 100;
```

### 组件引用
```csharp
// 必须在 Awake 中缓存
private Rigidbody _rb;
private void Awake() { _rb = GetComponent<Rigidbody>(); }
```

### 事件订阅
```csharp
// 必须配对
private void OnEnable() { Events.OnDied += Handle; }
private void OnDisable() { Events.OnDied -= Handle; }
```

## 生命周期规范

| 方法 | 用途 | 禁止 |
|------|-----|------|
| Awake | 缓存自身组件、初始化内部状态 | 引用其他GameObject |
| Start | 引用其他对象、注册到管理器 | - |
| Update | 输入处理、非物理逻辑 | GetComponent/Find/new |
| FixedUpdate | 物理操作 | 输入检测 |
| OnDisable | 取消事件订阅 | 必须与OnEnable配对 |

## 代码审查要点

生成或修改 Unity C# 代码后，自动检查:
1. GetComponent 是否已缓存
2. 事件订阅是否配对取消
3. 协程 WaitForSeconds 是否缓存
4. 编辑器代码是否正确隔离
5. 序列化字段是否使用 [SerializeField] private

## 推荐模式

### 单例管理器
```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }
    private void Awake()
    {
        if (Instance != null) { Destroy(gameObject); return; }
        Instance = this;
        DontDestroyOnLoad(gameObject);
    }
}
```

### 对象池获取
```csharp
var obj = _pool.Get();
_pool.Return(obj);
```

### 事件总线
```csharp
EventBus<PlayerDiedEvent>.Subscribe(handler);
EventBus<PlayerDiedEvent>.Unsubscribe(handler);
EventBus<PlayerDiedEvent>.Publish(new PlayerDiedEvent { });
```

## 详细规则参考

完整规则文档位于: `E:\lalanbvGitHub\CSharpAndUnity_AI_Rules_Skills\`
- `rules/UNITY_DEVELOPMENT_RULES.md` - 开发规则
- `qoder-config/rules/unity-development.md` - 开发规则
- `qoder-config/rules/unity-redlines.md` - 开发规则
- `redlines/UNITY_REDLINES.md` - 红线清单
- `skills/UNITY_SKILLS.md` - 技能体系
- `qoder-config/skills/unity-architecture/SKILL.md` - 技能体系
- `qoder-config/skills/unity-code-reviewer/SKILL.md` - 技能体系
- `qoder-config/skills/unity-developer/SKILL.md` - 技能体系
- `qoder-config/skills/unity-performance/SKILL.md` - 技能体系
- `qoder-config/skills/unity-workflows/SKILL.md` - 技能体系
- `templates/UNITY_CODE_TEMPLATES.md` - 代码模板
- `checklists/UNITY_CHECKLISTS.md` - 检查清单
