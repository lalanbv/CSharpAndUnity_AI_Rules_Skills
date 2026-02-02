---
description: Unity C#开发规范与最佳实践，为Unity项目提供代码标准指导
alwaysApply: false
globs:
  - "*.cs"
---

# Unity 开发规范

## 命名规范

### 类型命名
```csharp
// 类/结构体: PascalCase
public class PlayerController { }
public struct DamageInfo { }

// 接口: I前缀 + PascalCase
public interface IDamageable { }

// 枚举: PascalCase
public enum GameState { Playing, Paused, GameOver }

// 泛型参数: T前缀
public class ObjectPool<TObject> where TObject : Component { }
```

### 成员命名
```csharp
// 私有字段: _camelCase (下划线前缀)
private int _currentHealth;

// 序列化字段: _camelCase + [SerializeField]
[SerializeField] private int _maxHealth = 100;

// 属性: PascalCase
public int CurrentHealth => _currentHealth;

// 方法: PascalCase
public void TakeDamage(int amount) { }

// 事件: On前缀 + PascalCase
public event Action<int> OnHealthChanged;

// 参数: camelCase
public void Initialize(int initialHealth) { }
```

## 生命周期规范

| 方法 | 用途 | 禁止 |
|------|-----|------|
| Awake | 缓存自身组件、初始化内部状态 | 引用其他GameObject |
| Start | 引用其他对象、注册到管理器 | - |
| Update | 输入处理、非物理逻辑 | GetComponent/Find/new |
| FixedUpdate | 物理操作 | 输入检测 |
| OnDisable | 取消事件订阅 | 必须与OnEnable配对 |

### 标准MonoBehaviour结构
```csharp
public class StandardComponent : MonoBehaviour
{
    #region Serialized Fields
    [Header("Settings")]
    [SerializeField] private float _moveSpeed = 5f;
    
    [Header("References")]
    [SerializeField] private Transform _targetPoint;
    #endregion

    #region Private Fields
    private Rigidbody _rigidbody;
    #endregion

    #region Events
    public event Action OnSomethingHappened;
    #endregion

    #region Properties
    public float MoveSpeed => _moveSpeed;
    #endregion

    #region Unity Lifecycle
    private void Awake() { _rigidbody = GetComponent<Rigidbody>(); }
    private void OnEnable() { SubscribeEvents(); }
    private void Start() { Initialize(); }
    private void Update() { HandleUpdate(); }
    private void OnDisable() { UnsubscribeEvents(); }
    private void OnDestroy() { Cleanup(); }
    #endregion
}
```

## 序列化规范

```csharp
// ✅ 推荐: [SerializeField] + private
[SerializeField] private int _health = 100;
public int Health => _health;

// ❌ 不推荐: public字段
public int health = 100;

// ❌ 错误: 属性不会被序列化
public int Health { get; set; } = 100;

// ✅ Unity 2019.3+
[field: SerializeField] public int MaxHealth { get; private set; } = 100;
```

## 组件引用规范

```csharp
// ✅ 方式1: Inspector引用 (推荐)
[SerializeField] private Rigidbody _rigidbody;

// ✅ 方式2: Awake缓存
private Camera _mainCamera;
private void Awake() {
    _mainCamera = Camera.main;
}

// ✅ 使用TryGetComponent (Unity 2019.2+)
if (other.TryGetComponent<IDamageable>(out var damageable)) {
    damageable.TakeDamage(10);
}

// ✅ 声明组件依赖
[RequireComponent(typeof(Rigidbody))]
public class PhysicsObject : MonoBehaviour { }
```

## 事件规范

```csharp
// 事件定义
public event Action OnDeath;
public event Action<int> OnHealthChanged;

// 必须配对
private void OnEnable() {
    GameEvents.OnDied += Handle;
}
private void OnDisable() {
    GameEvents.OnDied -= Handle;
}

// 事件总线模式
EventBus<PlayerDiedEvent>.Subscribe(handler);
EventBus<PlayerDiedEvent>.Unsubscribe(handler);
EventBus<PlayerDiedEvent>.Publish(new PlayerDiedEvent { });
```

## 协程规范

```csharp
// ✅ 缓存WaitForSeconds
private readonly WaitForSeconds _waitOneSecond = new WaitForSeconds(1f);
private IEnumerator SpawnWaves() {
    while (true) {
        SpawnWave();
        yield return _waitOneSecond;
    }
}

// ✅ 对象销毁时停止协程
private void OnDisable() {
    StopAllCoroutines();
}
```

## 对象池规范

```csharp
// 标准用法
var obj = _pool.Get();
_pool.Return(obj);

// 可池化接口
public interface IPoolable {
    void OnSpawn();
    void OnDespawn();
}
```

## 编辑器代码规范

```csharp
// 必须条件编译
#if UNITY_EDITOR
private void OnValidate() {
    _moveSpeed = Mathf.Max(0f, _moveSpeed);
}
#endif

// Editor脚本必须在Editor文件夹中
// Assets/Scripts/Editor/MyEditor.cs
```
