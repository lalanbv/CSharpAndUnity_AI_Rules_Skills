---
name: unity-architecture
description: Unity游戏架构设计技能。适用于设计模式应用、单例管理、事件系统设计、ScriptableObject架构、组件解耦等任务。触发关键词：架构、设计模式、单例、Singleton、Manager、事件系统、EventBus、ScriptableObject、解耦、MVC
---

# Unity 架构设计技能

你是一位游戏架构师，精通Unity架构设计和设计模式应用。

## 核心能力

### 1. 单例管理器

```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }
        Instance = this;
        DontDestroyOnLoad(gameObject);
    }

    private void OnDestroy()
    {
        if (Instance == this) Instance = null;
    }
}
```

#### 泛型单例基类
```csharp
public abstract class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T _instance;
    public static T Instance
    {
        get
        {
            if (_instance == null)
            {
                _instance = FindObjectOfType<T>();
                if (_instance == null)
                {
                    var go = new GameObject($"[{typeof(T).Name}]");
                    _instance = go.AddComponent<T>();
                    DontDestroyOnLoad(go);
                }
            }
            return _instance;
        }
    }

    protected virtual void Awake()
    {
        if (_instance == null)
        {
            _instance = this as T;
            DontDestroyOnLoad(gameObject);
        }
        else if (_instance != this)
        {
            Destroy(gameObject);
        }
    }
}
```

### 2. 类型安全事件总线

```csharp
public static class EventBus<T> where T : struct
{
    private static readonly HashSet<Action<T>> _handlers = new HashSet<Action<T>>();

    public static void Subscribe(Action<T> handler) => _handlers.Add(handler);
    public static void Unsubscribe(Action<T> handler) => _handlers.Remove(handler);
    
    public static void Publish(T eventData)
    {
        foreach (var handler in _handlers)
        {
            try { handler?.Invoke(eventData); }
            catch (Exception e) { Debug.LogException(e); }
        }
    }

    public static void Clear() => _handlers.Clear();
}

// 事件定义 (使用struct避免GC)
public struct PlayerDiedEvent
{
    public Vector3 Position;
    public string CauseOfDeath;
    public int FinalScore;
}

// 使用
EventBus<PlayerDiedEvent>.Subscribe(OnPlayerDied);
EventBus<PlayerDiedEvent>.Unsubscribe(OnPlayerDied);
EventBus<PlayerDiedEvent>.Publish(new PlayerDiedEvent { ... });
```

### 3. ScriptableObject架构

#### 数据容器
```csharp
[CreateAssetMenu(fileName = "WeaponData", menuName = "Data/Weapon")]
public class WeaponData : ScriptableObject
{
    [Header("Basic Info")]
    public string weaponName;
    public Sprite icon;
    
    [Header("Stats")]
    public int damage;
    public float fireRate;
}
```

#### 运行时变量
```csharp
[CreateAssetMenu(fileName = "IntVariable", menuName = "Variables/Int")]
public class IntVariable : ScriptableObject
{
    [SerializeField] private int _initialValue;
    [NonSerialized] private int _runtimeValue;
    
    public int Value
    {
        get => _runtimeValue;
        set
        {
            if (_runtimeValue != value)
            {
                _runtimeValue = value;
                OnValueChanged?.Invoke(value);
            }
        }
    }
    
    public event Action<int> OnValueChanged;
    
    private void OnEnable() => _runtimeValue = _initialValue;
}
```

#### ScriptableObject事件
```csharp
[CreateAssetMenu(fileName = "GameEvent", menuName = "Events/Game Event")]
public class GameEventSO : ScriptableObject
{
    private readonly HashSet<GameEventListener> _listeners = new HashSet<GameEventListener>();
    
    public void Raise()
    {
        foreach (var listener in _listeners)
            listener.OnEventRaised();
    }
    
    public void RegisterListener(GameEventListener l) => _listeners.Add(l);
    public void UnregisterListener(GameEventListener l) => _listeners.Remove(l);
}
```

### 4. 组件通信解耦

#### 避免循环依赖
```csharp
// ❌ 错误: A依赖B，B依赖A
// ✅ 正确: 使用事件解耦

public class ComponentA : MonoBehaviour
{
    public event Action OnSomethingHappened;
    public void DoSomething() => OnSomethingHappened?.Invoke();
}

public class ComponentB : MonoBehaviour
{
    [SerializeField] private ComponentA _a;
    
    private void OnEnable() => _a.OnSomethingHappened += HandleEvent;
    private void OnDisable() => _a.OnSomethingHappened -= HandleEvent;
    private void HandleEvent() { /* ... */ }
}
```

## 架构决策指南

| 场景 | 推荐方案 |
|-----|---------|
| 全局唯一系统 | 单例 (GameManager, AudioManager) |
| 游戏数据配置 | ScriptableObject |
| 跨系统通信 | 事件总线 / SO事件 |
| UI-逻辑分离 | MVC/MVP模式 |
| 状态管理 | 状态机模式 |

## 单例使用原则

适合单例:
- GameManager
- AudioManager
- InputManager
- SaveManager

不适合单例:
- Player
- Enemy
- UI Panel
- Weapon
