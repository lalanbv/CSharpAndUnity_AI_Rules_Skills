# Unity 开发规则与最佳实践完全指南

> 基于 GitHub 优秀项目(Unity Open Project、Mirror、UniTask、DOTween等)的最佳实践整理

---

## 一、命名规范

### 1.1 类型命名

```csharp
// 类/结构体: PascalCase
public class PlayerController { }
public struct DamageInfo { }

// 接口: I前缀 + PascalCase
public interface IDamageable { }
public interface IInteractable { }

// 枚举: PascalCase, 成员也是PascalCase
public enum GameState { Playing, Paused, GameOver }

// 泛型参数: T前缀
public class ObjectPool<TObject> where TObject : Component { }
```

### 1.2 成员命名

```csharp
public class Example : MonoBehaviour
{
    // 常量: PascalCase 或 UPPER_SNAKE_CASE (团队统一)
    public const int MaxHealth = 100;
    private const float GRAVITY = -9.81f;
    
    // 静态只读: PascalCase
    public static readonly Vector3 DefaultPosition = Vector3.zero;
    
    // 私有字段: _camelCase (推荐) 或 camelCase
    private int _currentHealth;
    private float _moveSpeed;
    
    // 序列化字段: _camelCase + [SerializeField]
    [SerializeField] private int _maxHealth = 100;
    [SerializeField] private GameObject _projectilePrefab;
    
    // 属性: PascalCase
    public int CurrentHealth => _currentHealth;
    public bool IsAlive => _currentHealth > 0;
    
    // 方法: PascalCase
    public void TakeDamage(int amount) { }
    private void UpdateHealthBar() { }
    
    // 事件: On前缀 + PascalCase
    public event Action<int> OnHealthChanged;
    public event Action OnDeath;
    
    // 参数: camelCase
    public void Initialize(int initialHealth, float initialSpeed) { }
}
```

### 1.3 Unity特定命名

```csharp
// MonoBehaviour生命周期: Unity标准命名
private void Awake() { }
private void Start() { }
private void Update() { }
private void FixedUpdate() { }
private void LateUpdate() { }
private void OnEnable() { }
private void OnDisable() { }
private void OnDestroy() { }

// 消息方法: On前缀
private void OnTriggerEnter(Collider other) { }
private void OnCollisionEnter(Collision collision) { }
private void OnApplicationPause(bool pause) { }

// 协程: 动词 + Coroutine 或 Co后缀
private IEnumerator FadeOutCoroutine() { }
private IEnumerator SpawnEnemiesCo() { }

// 缓存字段: 与组件类型相关
private Rigidbody _rigidbody;
private Animator _animator;
private Transform _transform;
```

---

## 二、代码组织规范

### 2.1 文件结构

```csharp
// 1. using语句 (按组排列)
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
// 第三方库
using DG.Tweening;
using Cysharp.Threading.Tasks;
// 项目命名空间
using MyGame.Core;
using MyGame.UI;

// 2. 命名空间
namespace MyGame.Gameplay
{
    // 3. 类定义
    public class PlayerController : MonoBehaviour, IDamageable
    {
        // 4. 成员排列顺序 (推荐)
        #region Constants
        private const float MoveSpeed = 5f;
        #endregion

        #region Serialized Fields
        [Header("References")]
        [SerializeField] private Transform _firePoint;
        [SerializeField] private GameObject _bulletPrefab;
        
        [Header("Settings")]
        [SerializeField] private int _maxHealth = 100;
        [SerializeField] private float _fireRate = 0.5f;
        #endregion

        #region Private Fields
        private int _currentHealth;
        private float _lastFireTime;
        private Rigidbody _rigidbody;
        #endregion

        #region Events
        public event Action<int> OnHealthChanged;
        public event Action OnDeath;
        #endregion

        #region Properties
        public int CurrentHealth => _currentHealth;
        public bool IsAlive => _currentHealth > 0;
        #endregion

        #region Unity Lifecycle
        private void Awake() { }
        private void Start() { }
        private void Update() { }
        private void OnDestroy() { }
        #endregion

        #region Public Methods
        public void TakeDamage(int amount) { }
        public void Heal(int amount) { }
        #endregion

        #region Private Methods
        private void HandleMovement() { }
        private void HandleFiring() { }
        #endregion
    }
}
```

### 2.2 文件夹结构 (推荐)

```
Assets/
├── _Project/                    # 项目主文件夹 (下划线确保排在最前)
│   ├── Scripts/
│   │   ├── Core/               # 核心系统 (单例、事件、工具)
│   │   │   ├── Singleton.cs
│   │   │   ├── EventBus.cs
│   │   │   └── GameManager.cs
│   │   ├── Gameplay/           # 游戏玩法
│   │   │   ├── Player/
│   │   │   ├── Enemy/
│   │   │   └── Weapons/
│   │   ├── UI/                 # UI相关
│   │   ├── Data/               # ScriptableObject定义
│   │   ├── Editor/             # 编辑器脚本
│   │   └── Utils/              # 工具类
│   ├── Prefabs/
│   │   ├── Characters/
│   │   ├── UI/
│   │   └── VFX/
│   ├── ScriptableObjects/       # SO资产实例
│   ├── Scenes/
│   ├── Materials/
│   ├── Textures/
│   ├── Audio/
│   └── Animations/
├── Plugins/                     # 第三方插件
└── Resources/                   # 仅放必须动态加载的资源
```

---

## 三、MonoBehaviour 最佳实践

### 3.1 生命周期使用规则

```csharp
public class LifecycleExample : MonoBehaviour
{
    private Rigidbody _rigidbody;
    private PlayerInput _input;
    private GameManager _gameManager;
    
    // Awake: 初始化自身组件引用和内部状态
    // 执行顺序: 最先执行，脚本被加载时调用
    private void Awake()
    {
        // ✅ 正确: 缓存自身组件
        _rigidbody = GetComponent<Rigidbody>();
        
        // ✅ 正确: 初始化内部状态
        _currentHealth = _maxHealth;
        
        // ❌ 错误: 不要在Awake中查找其他对象
        // _gameManager = FindObjectOfType<GameManager>(); // 可能还未初始化
    }
    
    // OnEnable: 订阅事件、注册回调
    private void OnEnable()
    {
        // ✅ 正确: 订阅事件
        GameEvents.OnGamePaused += HandleGamePaused;
        _input.OnFirePressed += HandleFire;
    }
    
    // Start: 初始化外部引用、与其他对象交互
    // 执行顺序: 所有Awake执行完后，第一帧Update前
    private void Start()
    {
        // ✅ 正确: 查找其他对象
        _gameManager = GameManager.Instance;
        
        // ✅ 正确: 与其他组件交互
        _gameManager.RegisterPlayer(this);
    }
    
    // Update: 每帧逻辑 (输入、非物理移动、计时器)
    private void Update()
    {
        // ✅ 正确: 输入检测
        HandleInput();
        
        // ✅ 正确: UI更新
        UpdateUI();
        
        // ❌ 错误: 物理操作应放在FixedUpdate
        // _rigidbody.AddForce(Vector3.forward); 
    }
    
    // FixedUpdate: 物理计算 (固定时间步长，默认0.02s)
    private void FixedUpdate()
    {
        // ✅ 正确: 物理操作
        _rigidbody.AddForce(_moveDirection * _moveForce);
        
        // ❌ 错误: 不要在这里检测输入
        // if (Input.GetKeyDown(KeyCode.Space)) // 可能丢失输入
    }
    
    // LateUpdate: 跟随逻辑、最终调整 (在所有Update后执行)
    private void LateUpdate()
    {
        // ✅ 正确: 相机跟随
        // ✅ 正确: 骨骼IK调整
        // ✅ 正确: 最终位置修正
    }
    
    // OnDisable: 取消订阅事件 (必须与OnEnable配对!)
    private void OnDisable()
    {
        // ✅ 必须: 取消所有在OnEnable中的订阅
        GameEvents.OnGamePaused -= HandleGamePaused;
        _input.OnFirePressed -= HandleFire;
    }
    
    // OnDestroy: 最终清理
    private void OnDestroy()
    {
        // ✅ 正确: 释放资源
        if (_instantiatedMaterial != null)
            Destroy(_instantiatedMaterial);
            
        // ✅ 正确: 从管理器注销
        _gameManager?.UnregisterPlayer(this);
    }
}
```

### 3.2 空方法处理

```csharp
// ❌ 错误: 空的Unity回调方法仍有开销
public class BadExample : MonoBehaviour
{
    private void Update() { }  // Unity仍会调用，浪费性能
    private void FixedUpdate() { }
}

// ✅ 正确: 删除不需要的回调方法
public class GoodExample : MonoBehaviour
{
    // 只保留实际使用的方法
    private void Awake()
    {
        // 实际初始化代码
    }
}

// ✅ 正确: 动态启用/禁用Update
public class OptimizedExample : MonoBehaviour
{
    private bool _needsUpdate = false;
    
    private void Update()
    {
        if (!_needsUpdate) return;
        // 实际逻辑
    }
    
    // 或者使用enabled控制
    public void StartUpdating()
    {
        enabled = true;
    }
    
    public void StopUpdating()
    {
        enabled = false;
    }
}
```

---

## 四、序列化规范

### 4.1 字段序列化

```csharp
public class SerializationExample : MonoBehaviour
{
    // ✅ 推荐: [SerializeField] + private
    [SerializeField] private int _health = 100;
    [SerializeField] private float _speed = 5f;
    [SerializeField] private GameObject _prefab;
    
    // ⚠️ 可用但不推荐: public字段
    // public int health = 100;  // 暴露了内部实现
    
    // ✅ 推荐: 使用属性暴露只读访问
    public int Health => _health;
    
    // ❌ 错误: 属性不会被序列化
    // public int Health { get; set; } = 100;  // Inspector不显示
    
    // ✅ 正确: 使用[field: SerializeField] (Unity 2019.3+)
    [field: SerializeField] public int MaxHealth { get; private set; } = 100;
}
```

### 4.2 Inspector 组织

```csharp
public class InspectorOrganization : MonoBehaviour
{
    [Header("Movement Settings")]
    [Tooltip("Maximum movement speed in units per second")]
    [SerializeField] private float _moveSpeed = 5f;
    
    [Tooltip("Rotation speed in degrees per second")]
    [SerializeField] private float _rotationSpeed = 180f;
    
    [Header("Combat Settings")]
    [SerializeField] private int _maxHealth = 100;
    
    [Range(0.1f, 2f)]
    [SerializeField] private float _attackCooldown = 0.5f;
    
    [Header("References")]
    [SerializeField] private Transform _firePoint;
    [SerializeField] private GameObject _bulletPrefab;
    
    [Header("Audio")]
    [SerializeField] private AudioClip _fireSound;
    [SerializeField] private AudioClip _hitSound;
    
    [Header("Debug")]
    [SerializeField] private bool _showDebugInfo = false;
    
    // 使用Space增加间距
    [Space(10)]
    [SerializeField] private bool _isInvincible = false;
}
```

### 4.3 自定义可序列化类型

```csharp
// ✅ 正确: 可序列化的自定义类
[System.Serializable]
public class WeaponStats
{
    public string weaponName;
    public int damage;
    public float fireRate;
    public Sprite icon;
}

// ✅ 正确: 可序列化的结构体
[System.Serializable]
public struct DamageInfo
{
    public int amount;
    public DamageType type;
    public Vector3 hitPoint;
}

// 在MonoBehaviour中使用
public class Weapon : MonoBehaviour
{
    [SerializeField] private WeaponStats _stats;
    [SerializeField] private List<WeaponStats> _availableWeapons;
}

// ❌ 注意: Dictionary不能直接序列化
// [SerializeField] private Dictionary<string, int> _items; // 不工作!

// ✅ 解决方案: 使用可序列化的包装类
[System.Serializable]
public class SerializableDictionary<TKey, TValue> : ISerializationCallbackReceiver
{
    [SerializeField] private List<TKey> _keys = new List<TKey>();
    [SerializeField] private List<TValue> _values = new List<TValue>();
    
    private Dictionary<TKey, TValue> _dictionary = new Dictionary<TKey, TValue>();
    
    public TValue this[TKey key]
    {
        get => _dictionary[key];
        set => _dictionary[key] = value;
    }
    
    public void OnBeforeSerialize()
    {
        _keys.Clear();
        _values.Clear();
        foreach (var kvp in _dictionary)
        {
            _keys.Add(kvp.Key);
            _values.Add(kvp.Value);
        }
    }
    
    public void OnAfterDeserialize()
    {
        _dictionary.Clear();
        for (int i = 0; i < Math.Min(_keys.Count, _values.Count); i++)
        {
            _dictionary[_keys[i]] = _values[i];
        }
    }
}
```

---

## 五、组件引用规范

### 5.1 GetComponent 缓存

```csharp
public class ComponentCaching : MonoBehaviour
{
    // 方式1: Inspector引用 (推荐，避免运行时查找)
    [SerializeField] private Rigidbody _rigidbody;
    [SerializeField] private Animator _animator;
    
    // 方式2: Awake缓存 (当无法Inspector赋值时)
    private Camera _mainCamera;
    private AudioSource _audioSource;
    
    private void Awake()
    {
        // 自身组件
        _audioSource = GetComponent<AudioSource>();
        
        // 子物体组件
        _animator = GetComponentInChildren<Animator>();
        
        // 场景组件
        _mainCamera = Camera.main; // 缓存，避免每帧访问
    }
    
    // ❌ 错误示范
    private void Update()
    {
        // 每帧GetComponent - 严重性能问题!
        GetComponent<Rigidbody>().velocity = Vector3.forward;
        
        // 每帧访问Camera.main - 性能问题!
        Camera.main.ScreenToWorldPoint(Input.mousePosition);
    }
    
    // ✅ 正确示范
    private void Update_Correct()
    {
        _rigidbody.velocity = Vector3.forward;
        _mainCamera.ScreenToWorldPoint(Input.mousePosition);
    }
}
```

### 5.2 TryGetComponent 模式

```csharp
public class TryGetComponentPattern : MonoBehaviour
{
    // ❌ 旧模式: GetComponent + null检查
    private void OnTriggerEnter_Old(Collider other)
    {
        IDamageable damageable = other.GetComponent<IDamageable>();
        if (damageable != null)
        {
            damageable.TakeDamage(10);
        }
    }
    
    // ✅ 新模式: TryGetComponent (Unity 2019.2+)
    private void OnTriggerEnter(Collider other)
    {
        if (other.TryGetComponent<IDamageable>(out var damageable))
        {
            damageable.TakeDamage(10);
        }
    }
    
    // ✅ 多组件检查
    private void ProcessCollision(Collider other)
    {
        if (other.TryGetComponent<Health>(out var health))
        {
            health.TakeDamage(10);
        }
        
        if (other.TryGetComponent<Rigidbody>(out var rb))
        {
            rb.AddForce(transform.forward * 100f);
        }
    }
}
```

### 5.3 RequireComponent

```csharp
// ✅ 声明组件依赖
[RequireComponent(typeof(Rigidbody))]
[RequireComponent(typeof(Collider))]
public class PhysicsObject : MonoBehaviour
{
    private Rigidbody _rigidbody;
    
    private void Awake()
    {
        // RequireComponent确保组件存在，无需null检查
        _rigidbody = GetComponent<Rigidbody>();
    }
}

// 多个RequireComponent
[RequireComponent(typeof(AudioSource))]
[RequireComponent(typeof(Animator))]
[RequireComponent(typeof(CharacterController))]
public class PlayerCharacter : MonoBehaviour
{
    // ...
}
```

---

## 六、事件与通信规范

### 6.1 C# 事件模式

```csharp
public class EventPatterns : MonoBehaviour
{
    // 简单事件
    public event Action OnDeath;
    
    // 带参数事件
    public event Action<int> OnHealthChanged;
    public event Action<int, int> OnHealthChanged2; // (current, max)
    
    // 自定义EventArgs (复杂数据)
    public event EventHandler<DamageEventArgs> OnDamaged;
    
    private int _health = 100;
    
    public void TakeDamage(int amount, GameObject source)
    {
        int previousHealth = _health;
        _health -= amount;
        
        // 触发事件
        OnHealthChanged?.Invoke(_health);
        
        OnDamaged?.Invoke(this, new DamageEventArgs
        {
            Amount = amount,
            Source = source,
            PreviousHealth = previousHealth,
            CurrentHealth = _health
        });
        
        if (_health <= 0)
        {
            OnDeath?.Invoke();
        }
    }
}

public class DamageEventArgs : EventArgs
{
    public int Amount { get; set; }
    public GameObject Source { get; set; }
    public int PreviousHealth { get; set; }
    public int CurrentHealth { get; set; }
}

// 订阅者
public class HealthUI : MonoBehaviour
{
    [SerializeField] private EventPatterns _player;
    
    private void OnEnable()
    {
        // ✅ 订阅
        _player.OnHealthChanged += UpdateHealthBar;
        _player.OnDeath += ShowDeathScreen;
    }
    
    private void OnDisable()
    {
        // ✅ 必须取消订阅!
        _player.OnHealthChanged -= UpdateHealthBar;
        _player.OnDeath -= ShowDeathScreen;
    }
    
    private void UpdateHealthBar(int health) { }
    private void ShowDeathScreen() { }
}
```

### 6.2 全局事件总线

```csharp
// 类型安全的事件总线
public static class EventBus<T> where T : struct
{
    private static event Action<T> OnEvent;
    
    public static void Subscribe(Action<T> handler)
    {
        OnEvent += handler;
    }
    
    public static void Unsubscribe(Action<T> handler)
    {
        OnEvent -= handler;
    }
    
    public static void Publish(T eventData)
    {
        OnEvent?.Invoke(eventData);
    }
    
    public static void Clear()
    {
        OnEvent = null;
    }
}

// 事件定义 (使用struct避免GC)
public struct PlayerDiedEvent
{
    public Vector3 Position;
    public string KilledBy;
    public int Score;
}

public struct EnemySpawnedEvent
{
    public GameObject Enemy;
    public EnemyType Type;
}

// 发布事件
public class Player : MonoBehaviour
{
    private void Die()
    {
        EventBus<PlayerDiedEvent>.Publish(new PlayerDiedEvent
        {
            Position = transform.position,
            KilledBy = "Enemy",
            Score = _score
        });
    }
}

// 订阅事件
public class GameOverUI : MonoBehaviour
{
    private void OnEnable()
    {
        EventBus<PlayerDiedEvent>.Subscribe(OnPlayerDied);
    }
    
    private void OnDisable()
    {
        EventBus<PlayerDiedEvent>.Unsubscribe(OnPlayerDied);
    }
    
    private void OnPlayerDied(PlayerDiedEvent e)
    {
        ShowGameOver(e.Score);
    }
}
```

### 6.3 ScriptableObject 事件

```csharp
// 可在Inspector中配置的事件通道
[CreateAssetMenu(fileName = "GameEvent", menuName = "Events/Game Event")]
public class GameEventSO : ScriptableObject
{
    private readonly HashSet<GameEventListener> _listeners = new HashSet<GameEventListener>();
    
    public void Raise()
    {
        foreach (var listener in _listeners)
        {
            listener.OnEventRaised();
        }
    }
    
    public void RegisterListener(GameEventListener listener)
    {
        _listeners.Add(listener);
    }
    
    public void UnregisterListener(GameEventListener listener)
    {
        _listeners.Remove(listener);
    }
}

// 带参数版本
[CreateAssetMenu(fileName = "IntEvent", menuName = "Events/Int Event")]
public class IntEventSO : ScriptableObject
{
    private readonly HashSet<IntEventListener> _listeners = new HashSet<IntEventListener>();
    
    public void Raise(int value)
    {
        foreach (var listener in _listeners)
        {
            listener.OnEventRaised(value);
        }
    }
    
    // ... Register/Unregister
}

// 监听器组件
public class GameEventListener : MonoBehaviour
{
    [SerializeField] private GameEventSO _event;
    [SerializeField] private UnityEvent _response;
    
    private void OnEnable() => _event.RegisterListener(this);
    private void OnDisable() => _event.UnregisterListener(this);
    
    public void OnEventRaised() => _response.Invoke();
}
```

---

## 七、协程规范

### 7.1 协程基础

```csharp
public class CoroutinePatterns : MonoBehaviour
{
    private Coroutine _currentCoroutine;
    
    // ✅ 缓存WaitForSeconds
    private readonly WaitForSeconds _waitOneSecond = new WaitForSeconds(1f);
    private readonly WaitForFixedUpdate _waitFixed = new WaitForFixedUpdate();
    private readonly WaitForEndOfFrame _waitEndOfFrame = new WaitForEndOfFrame();
    
    // 基础协程
    private IEnumerator FadeOut(float duration)
    {
        float elapsed = 0f;
        Color startColor = _renderer.material.color;
        
        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            float t = elapsed / duration;
            
            Color newColor = startColor;
            newColor.a = Mathf.Lerp(1f, 0f, t);
            _renderer.material.color = newColor;
            
            yield return null; // 等待下一帧
        }
    }
    
    // 使用缓存的等待对象
    private IEnumerator SpawnWaves()
    {
        while (true)
        {
            SpawnWave();
            yield return _waitOneSecond; // ✅ 使用缓存，无GC
            // yield return new WaitForSeconds(1f); // ❌ 每次创建新对象
        }
    }
    
    // 安全启动协程 (停止之前的)
    public void StartFade(float duration)
    {
        if (_currentCoroutine != null)
        {
            StopCoroutine(_currentCoroutine);
        }
        _currentCoroutine = StartCoroutine(FadeOut(duration));
    }
    
    // 对象销毁时停止协程
    private void OnDisable()
    {
        StopAllCoroutines();
    }
}
```

### 7.2 协程进阶模式

```csharp
public class AdvancedCoroutines : MonoBehaviour
{
    // 可取消的协程
    private CancellationTokenSource _cts;
    
    // 等待条件
    private IEnumerator WaitForCondition()
    {
        // 等待直到条件为真
        yield return new WaitUntil(() => _isReady);
        
        // 等待直到条件为假
        yield return new WaitWhile(() => _isLoading);
    }
    
    // 嵌套协程
    private IEnumerator ComplexSequence()
    {
        Debug.Log("Phase 1");
        yield return StartCoroutine(Phase1());
        
        Debug.Log("Phase 2");
        yield return StartCoroutine(Phase2());
        
        Debug.Log("Complete");
    }
    
    // 并行协程
    private IEnumerator ParallelOperations()
    {
        Coroutine op1 = StartCoroutine(Operation1());
        Coroutine op2 = StartCoroutine(Operation2());
        Coroutine op3 = StartCoroutine(Operation3());
        
        // 等待所有完成
        yield return op1;
        yield return op2;
        yield return op3;
    }
    
    // 带超时的协程
    private IEnumerator WithTimeout(IEnumerator routine, float timeout)
    {
        float startTime = Time.time;
        
        while (routine.MoveNext())
        {
            if (Time.time - startTime > timeout)
            {
                Debug.LogWarning("Coroutine timed out!");
                yield break;
            }
            yield return routine.Current;
        }
    }
}
```

---

## 八、异步编程规范 (UniTask)

### 8.1 UniTask 基础

```csharp
using Cysharp.Threading.Tasks;

public class UniTaskPatterns : MonoBehaviour
{
    private CancellationTokenSource _cts;
    
    private void Awake()
    {
        _cts = new CancellationTokenSource();
    }
    
    private void OnDestroy()
    {
        // ✅ 必须: 取消所有异步操作
        _cts?.Cancel();
        _cts?.Dispose();
    }
    
    // 基础异步方法
    private async UniTaskVoid Start()
    {
        await LoadGameAsync();
        Debug.Log("Game loaded!");
    }
    
    private async UniTask LoadGameAsync()
    {
        // 等待帧
        await UniTask.Yield();
        
        // 等待时间
        await UniTask.Delay(TimeSpan.FromSeconds(1f));
        
        // 等待条件
        await UniTask.WaitUntil(() => _isReady);
        
        // 带取消令牌
        await UniTask.Delay(1000, cancellationToken: _cts.Token);
    }
    
    // 带返回值的异步
    private async UniTask<int> CalculateScoreAsync()
    {
        await UniTask.Delay(100);
        return 1000;
    }
    
    // 多个异步并行
    private async UniTask LoadAllAsync()
    {
        var (result1, result2, result3) = await UniTask.WhenAll(
            LoadDataAsync(),
            LoadAudioAsync(),
            LoadTexturesAsync()
        );
    }
    
    // 竞速 (取第一个完成的)
    private async UniTask<int> RaceAsync()
    {
        return await UniTask.WhenAny(
            LoadFromServer(),
            LoadFromCache()
        );
    }
}
```

### 8.2 组件销毁安全

```csharp
public class SafeAsyncComponent : MonoBehaviour
{
    private CancellationTokenSource _destroyCts;
    
    private void Awake()
    {
        _destroyCts = new CancellationTokenSource();
    }
    
    private void OnDestroy()
    {
        _destroyCts.Cancel();
        _destroyCts.Dispose();
    }
    
    // 使用GetCancellationTokenOnDestroy扩展方法
    private async UniTaskVoid SafeOperation()
    {
        try
        {
            // 自动在组件销毁时取消
            await UniTask.Delay(5000, cancellationToken: this.GetCancellationTokenOnDestroy());
            
            // 只有未被取消才会执行到这里
            Debug.Log("Operation complete!");
        }
        catch (OperationCanceledException)
        {
            // 正常取消，不需要处理
        }
    }
    
    // 带超时的操作
    private async UniTask<bool> LoadWithTimeout()
    {
        var timeoutCts = new CancellationTokenSource();
        timeoutCts.CancelAfter(TimeSpan.FromSeconds(10));
        
        var linkedCts = CancellationTokenSource.CreateLinkedTokenSource(
            _destroyCts.Token, 
            timeoutCts.Token
        );
        
        try
        {
            await LoadDataAsync().AttachExternalCancellation(linkedCts.Token);
            return true;
        }
        catch (OperationCanceledException)
        {
            return false;
        }
        finally
        {
            timeoutCts.Dispose();
            linkedCts.Dispose();
        }
    }
}
```

---

## 九、对象池规范

### 9.1 通用对象池

```csharp
public class ObjectPool<T> where T : Component
{
    private readonly T _prefab;
    private readonly Transform _parent;
    private readonly Queue<T> _pool = new Queue<T>();
    private readonly HashSet<T> _activeObjects = new HashSet<T>();
    
    public ObjectPool(T prefab, int initialSize, Transform parent = null)
    {
        _prefab = prefab;
        _parent = parent;
        
        // 预热
        for (int i = 0; i < initialSize; i++)
        {
            T obj = CreateNew();
            obj.gameObject.SetActive(false);
            _pool.Enqueue(obj);
        }
    }
    
    public T Get()
    {
        T obj = _pool.Count > 0 ? _pool.Dequeue() : CreateNew();
        obj.gameObject.SetActive(true);
        _activeObjects.Add(obj);
        return obj;
    }
    
    public void Return(T obj)
    {
        if (!_activeObjects.Contains(obj)) return;
        
        obj.gameObject.SetActive(false);
        _activeObjects.Remove(obj);
        _pool.Enqueue(obj);
    }
    
    public void ReturnAll()
    {
        foreach (var obj in _activeObjects.ToArray())
        {
            Return(obj);
        }
    }
    
    private T CreateNew()
    {
        return Object.Instantiate(_prefab, _parent);
    }
    
    public int ActiveCount => _activeObjects.Count;
    public int PooledCount => _pool.Count;
}

// 使用示例
public class BulletSpawner : MonoBehaviour
{
    [SerializeField] private Bullet _bulletPrefab;
    [SerializeField] private int _poolSize = 50;
    
    private ObjectPool<Bullet> _bulletPool;
    
    private void Awake()
    {
        _bulletPool = new ObjectPool<Bullet>(_bulletPrefab, _poolSize, transform);
    }
    
    public void Fire(Vector3 position, Vector3 direction)
    {
        Bullet bullet = _bulletPool.Get();
        bullet.Initialize(position, direction, () => _bulletPool.Return(bullet));
    }
}
```

### 9.2 可池化接口

```csharp
// 池化对象接口
public interface IPoolable
{
    void OnSpawn();     // 从池中取出时调用
    void OnDespawn();   // 返回池中时调用
}

public class PoolableBullet : MonoBehaviour, IPoolable
{
    private TrailRenderer _trail;
    
    public void OnSpawn()
    {
        // 重置状态
        _trail.Clear();
        // 启用组件
        enabled = true;
    }
    
    public void OnDespawn()
    {
        // 清理状态
        _trail.Clear();
        // 禁用组件
        enabled = false;
    }
}

// 支持IPoolable的对象池
public class SmartObjectPool<T> where T : Component, IPoolable
{
    private readonly Queue<T> _pool = new Queue<T>();
    // ... 其他代码
    
    public T Get()
    {
        T obj = _pool.Count > 0 ? _pool.Dequeue() : CreateNew();
        obj.gameObject.SetActive(true);
        obj.OnSpawn(); // 调用初始化
        return obj;
    }
    
    public void Return(T obj)
    {
        obj.OnDespawn(); // 调用清理
        obj.gameObject.SetActive(false);
        _pool.Enqueue(obj);
    }
}
```

---

## 十、ScriptableObject 规范

### 10.1 数据容器

```csharp
// 游戏配置数据
[CreateAssetMenu(fileName = "GameConfig", menuName = "Config/Game Config")]
public class GameConfig : ScriptableObject
{
    [Header("Player Settings")]
    public int startingHealth = 100;
    public float moveSpeed = 5f;
    public float jumpForce = 10f;
    
    [Header("Game Rules")]
    public int maxLives = 3;
    public float respawnTime = 3f;
    
    [Header("Difficulty")]
    public AnimationCurve difficultyCurve;
    
    // 验证数据
    private void OnValidate()
    {
        startingHealth = Mathf.Max(1, startingHealth);
        moveSpeed = Mathf.Max(0.1f, moveSpeed);
    }
}

// 武器数据
[CreateAssetMenu(fileName = "WeaponData", menuName = "Data/Weapon")]
public class WeaponData : ScriptableObject
{
    public string weaponName;
    public Sprite icon;
    public GameObject prefab;
    
    [Header("Stats")]
    public int damage;
    public float fireRate;
    public int magazineSize;
    public float reloadTime;
    
    [Header("Audio")]
    public AudioClip fireSound;
    public AudioClip reloadSound;
}
```

### 10.2 运行时数据

```csharp
// 运行时共享变量
[CreateAssetMenu(fileName = "IntVariable", menuName = "Variables/Int")]
public class IntVariable : ScriptableObject
{
    [SerializeField] private int _initialValue;
    [System.NonSerialized] private int _runtimeValue;
    
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
    
    private void OnEnable()
    {
        _runtimeValue = _initialValue;
    }
    
    // Editor中重置
    public void Reset()
    {
        _runtimeValue = _initialValue;
    }
}

// 使用
public class Player : MonoBehaviour
{
    [SerializeField] private IntVariable _health;
    
    public void TakeDamage(int amount)
    {
        _health.Value -= amount;
    }
}

public class HealthUI : MonoBehaviour
{
    [SerializeField] private IntVariable _health;
    [SerializeField] private Text _healthText;
    
    private void OnEnable()
    {
        _health.OnValueChanged += UpdateDisplay;
        UpdateDisplay(_health.Value);
    }
    
    private void OnDisable()
    {
        _health.OnValueChanged -= UpdateDisplay;
    }
    
    private void UpdateDisplay(int value)
    {
        _healthText.text = $"Health: {value}";
    }
}
```

### 10.3 运行时集合

```csharp
// 运行时对象集合
[CreateAssetMenu(fileName = "RuntimeSet", menuName = "Sets/Runtime Set")]
public class RuntimeSet<T> : ScriptableObject
{
    private readonly List<T> _items = new List<T>();
    
    public IReadOnlyList<T> Items => _items;
    public int Count => _items.Count;
    
    public event Action<T> OnItemAdded;
    public event Action<T> OnItemRemoved;
    
    public void Add(T item)
    {
        if (!_items.Contains(item))
        {
            _items.Add(item);
            OnItemAdded?.Invoke(item);
        }
    }
    
    public void Remove(T item)
    {
        if (_items.Remove(item))
        {
            OnItemRemoved?.Invoke(item);
        }
    }
    
    public void Clear()
    {
        _items.Clear();
    }
    
    private void OnDisable()
    {
        _items.Clear();
    }
}

// 具体实现
[CreateAssetMenu(fileName = "EnemySet", menuName = "Sets/Enemy Set")]
public class EnemyRuntimeSet : RuntimeSet<Enemy> { }

// 使用
public class Enemy : MonoBehaviour
{
    [SerializeField] private EnemyRuntimeSet _enemySet;
    
    private void OnEnable() => _enemySet.Add(this);
    private void OnDisable() => _enemySet.Remove(this);
}

public class EnemyManager : MonoBehaviour
{
    [SerializeField] private EnemyRuntimeSet _enemySet;
    
    public void DamageAllEnemies(int damage)
    {
        foreach (var enemy in _enemySet.Items)
        {
            enemy.TakeDamage(damage);
        }
    }
    
    public int AliveEnemyCount => _enemySet.Count;
}
```

---

## 十一、调试规范

### 11.1 条件编译

```csharp
public class DebuggingPatterns : MonoBehaviour
{
    // 只在Editor中执行
    [System.Diagnostics.Conditional("UNITY_EDITOR")]
    private void EditorOnlyDebug()
    {
        Debug.Log("This only runs in Editor");
    }
    
    // 自定义编译符号
    [System.Diagnostics.Conditional("ENABLE_CHEATS")]
    public void CheatAddHealth()
    {
        _health += 100;
    }
    
    // 使用#if
    private void Update()
    {
        #if UNITY_EDITOR
        if (Input.GetKeyDown(KeyCode.F1))
        {
            Debug.Log("Debug info: " + _debugInfo);
        }
        #endif
        
        // 正常游戏逻辑
        HandleInput();
    }
    
    // Debug绘制
    private void OnDrawGizmos()
    {
        Gizmos.color = Color.yellow;
        Gizmos.DrawWireSphere(transform.position, _detectionRadius);
    }
    
    private void OnDrawGizmosSelected()
    {
        // 只在选中时绘制
        Gizmos.color = Color.red;
        Gizmos.DrawLine(transform.position, _targetPosition);
    }
}
```

### 11.2 自定义日志系统

```csharp
public static class GameLog
{
    public enum LogLevel { Debug, Info, Warning, Error }
    
    public static LogLevel MinLevel = LogLevel.Debug;
    
    [System.Diagnostics.Conditional("UNITY_EDITOR")]
    [System.Diagnostics.Conditional("DEVELOPMENT_BUILD")]
    public static void Debug(string message, UnityEngine.Object context = null)
    {
        if (MinLevel <= LogLevel.Debug)
            UnityEngine.Debug.Log($"[DEBUG] {message}", context);
    }
    
    public static void Info(string message, UnityEngine.Object context = null)
    {
        if (MinLevel <= LogLevel.Info)
            UnityEngine.Debug.Log($"[INFO] {message}", context);
    }
    
    public static void Warning(string message, UnityEngine.Object context = null)
    {
        if (MinLevel <= LogLevel.Warning)
            UnityEngine.Debug.LogWarning($"[WARN] {message}", context);
    }
    
    public static void Error(string message, UnityEngine.Object context = null)
    {
        UnityEngine.Debug.LogError($"[ERROR] {message}", context);
    }
    
    // 带分类的日志
    public static void Log(string category, string message)
    {
        UnityEngine.Debug.Log($"[{category}] {message}");
    }
}

// 使用
public class Player : MonoBehaviour
{
    public void TakeDamage(int amount)
    {
        GameLog.Debug($"Player took {amount} damage", this);
        _health -= amount;
        
        if (_health <= 0)
        {
            GameLog.Info("Player died");
        }
    }
}
```

---

## 十二、附录：代码审查检查清单

### 每次提交前检查

- [ ] 所有GetComponent调用已缓存
- [ ] 所有事件订阅都有对应的取消订阅
- [ ] 没有在Update中使用Find方法
- [ ] 没有在Update中进行字符串拼接
- [ ] 协程的WaitForSeconds已缓存
- [ ] 编辑器代码已用#if UNITY_EDITOR包裹
- [ ] 序列化字段使用[SerializeField] private
- [ ] 高频创建的对象使用对象池
- [ ] 没有空的Update/FixedUpdate方法
- [ ] 使用TryGetComponent而非GetComponent+null检查

### 性能敏感代码检查

- [ ] 避免LINQ在热路径中使用
- [ ] 避免装箱拆箱操作
- [ ] 使用StringBuilder处理字符串
- [ ] 物理操作在FixedUpdate中
- [ ] Camera.main已缓存
- [ ] 合理使用Physics LayerMask
