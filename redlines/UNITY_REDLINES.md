# Unity 开发红线清单

> 版本: 1.0.0 | 严重级别: P0(致命) > P1(严重) > P2(重要) > P3(建议)
> 红线 = 绝对禁止的做法，违反将导致性能问题、内存泄漏、崩溃或难以调试的Bug

---

## 一、性能红线 (Performance)

### P0-001: Update中调用GetComponent

```csharp
// ❌ 红线违反 - 每帧调用GetComponent
private void Update()
{
    GetComponent<Rigidbody>().velocity = Vector3.forward;  // 严重性能问题!
}

// ✅ 正确做法 - Awake中缓存
private Rigidbody _rb;
private void Awake()
{
    _rb = GetComponent<Rigidbody>();
}
private void Update()
{
    _rb.velocity = Vector3.forward;
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P0 (致命) |
| 性能影响 | GetComponent每帧调用比缓存引用慢10-100倍 |
| 检测方式 | Profiler → Scripts → GetComponent |
| 修复方案 | 在Awake/Start中缓存所有组件引用 |

---

### P0-002: Update中使用Find方法

```csharp
// ❌ 红线违反 - 每帧遍历整个场景
private void Update()
{
    GameObject player = GameObject.Find("Player");  // 极慢!
    Transform target = GameObject.FindWithTag("Enemy").transform;  // 极慢!
    var manager = FindObjectOfType<GameManager>();  // 极慢!
}

// ✅ 正确做法 - Start中缓存或使用事件/引用
private GameObject _player;
private Transform _target;
private GameManager _manager;

private void Start()
{
    _player = GameObject.Find("Player");
    _target = GameObject.FindWithTag("Enemy")?.transform;
    _manager = FindObjectOfType<GameManager>();
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P0 (致命) |
| 性能影响 | Find方法遍历整个场景层级，复杂度O(n) |
| 检测方式 | Profiler → Scripts → GameObject.Find |
| 修复方案 | Inspector引用 / 单例模式 / 事件系统 |

---

### P0-003: Update中字符串拼接

```csharp
// ❌ 红线违反 - 每帧创建新字符串，触发GC
private void Update()
{
    _scoreText.text = "Score: " + _score;  // GC!
    _healthText.text = $"HP: {_health}/{_maxHealth}";  // GC!
}

// ✅ 正确做法 - 缓存StringBuilder或只在值变化时更新
private StringBuilder _sb = new StringBuilder(32);

private void UpdateScoreDisplay()  // 只在分数变化时调用
{
    _sb.Clear();
    _sb.Append("Score: ").Append(_score);
    _scoreText.text = _sb.ToString();
}

// 或使用事件驱动
private void OnScoreChanged(int newScore)
{
    _scoreText.text = newScore.ToString();
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P0 (致命) |
| 性能影响 | 字符串不可变，每次拼接创建新对象，导致GC Spike |
| 检测方式 | Profiler → GC Alloc列 |
| 修复方案 | StringBuilder / 只在值变化时更新 / 缓存格式化字符串 |

---

### P0-004: Update中new分配

```csharp
// ❌ 红线违反 - 每帧堆分配
private void Update()
{
    List<Enemy> enemies = new List<Enemy>();  // GC!
    Vector3[] path = new Vector3[100];  // GC!
    var info = new DamageInfo();  // struct在某些情况下也会装箱
}

// ✅ 正确做法 - 预分配并复用
private List<Enemy> _enemyCache = new List<Enemy>(64);
private Vector3[] _pathCache = new Vector3[100];

private void Update()
{
    _enemyCache.Clear();  // 复用List
    FindEnemies(_enemyCache);
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P0 (致命) |
| 性能影响 | 每帧分配导致GC频繁触发，造成卡顿 |
| 检测方式 | Profiler → GC Alloc > 0B in Update |
| 修复方案 | 预分配集合 / 对象池 / 使用struct |

---

### P0-005: 空的Update/FixedUpdate方法

```csharp
// ❌ 红线违反 - Unity仍会调用空方法
public class EmptyUpdate : MonoBehaviour
{
    private void Update() { }  // 有开销!
    private void FixedUpdate() { }  // 有开销!
    private void LateUpdate() { }  // 有开销!
}

// ✅ 正确做法 - 删除不需要的方法
public class NoUpdate : MonoBehaviour
{
    private void Awake()
    {
        // 只保留需要的生命周期方法
    }
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P1 (严重) |
| 性能影响 | 每个空Update约0.5μs开销，1000个脚本=每帧0.5ms |
| 检测方式 | 代码审查 / 自动化脚本检测 |
| 修复方案 | 删除空方法 / 使用enabled控制 |

---

### P0-006: LINQ在热路径中使用

```csharp
// ❌ 红线违反 - LINQ在Update中创建大量GC
private void Update()
{
    var activeEnemies = _enemies.Where(e => e.IsActive).ToList();  // GC!
    var closest = _enemies.OrderBy(e => e.Distance).First();  // GC!
    int total = _items.Sum(i => i.Value);  // 可能GC
}

// ✅ 正确做法 - 使用传统循环
private void Update()
{
    _activeEnemies.Clear();
    for (int i = 0; i < _enemies.Count; i++)
    {
        if (_enemies[i].IsActive)
            _activeEnemies.Add(_enemies[i]);
    }
    
    Enemy closest = null;
    float minDist = float.MaxValue;
    for (int i = 0; i < _enemies.Count; i++)
    {
        if (_enemies[i].Distance < minDist)
        {
            minDist = _enemies[i].Distance;
            closest = _enemies[i];
        }
    }
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P1 (严重) |
| 性能影响 | LINQ创建迭代器和委托，产生GC |
| 检测方式 | Profiler → GC Alloc / 代码审查搜索LINQ |
| 修复方案 | 使用for循环 / 在非热路径中使用LINQ |

---

### P0-007: 每帧访问Camera.main

```csharp
// ❌ 红线违反 - Camera.main每次调用都查找
private void Update()
{
    Vector3 screenPos = Camera.main.WorldToScreenPoint(transform.position);  // 查找!
}

// ✅ 正确做法 - 缓存Camera引用
private Camera _mainCamera;
private void Awake()
{
    _mainCamera = Camera.main;
}
private void Update()
{
    Vector3 screenPos = _mainCamera.WorldToScreenPoint(transform.position);
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P1 (严重) |
| 性能影响 | Camera.main使用FindGameObjectsWithTag，每次调用都查找 |
| 检测方式 | Profiler → Camera.main |
| 修复方案 | 在Awake/Start中缓存Camera.main |

---

### P0-008: WaitForSeconds未缓存

```csharp
// ❌ 红线违反 - 每次创建新WaitForSeconds
private IEnumerator SpawnRoutine()
{
    while (true)
    {
        Spawn();
        yield return new WaitForSeconds(1f);  // 每次GC!
    }
}

// ✅ 正确做法 - 缓存WaitForSeconds
private readonly WaitForSeconds _waitOneSecond = new WaitForSeconds(1f);
private IEnumerator SpawnRoutine()
{
    while (true)
    {
        Spawn();
        yield return _waitOneSecond;  // 复用
    }
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P1 (严重) |
| 性能影响 | 每次yield return new都分配内存 |
| 检测方式 | Profiler → GC Alloc in coroutine |
| 修复方案 | 缓存常用的WaitForSeconds实例 |

---

### P0-009: Material泄漏

```csharp
// ❌ 红线违反 - 访问.material创建实例，从不销毁
private void Update()
{
    GetComponent<Renderer>().material.color = Color.red;  // 每次创建新Material!
}

// ✅ 正确做法 - 缓存并在OnDestroy中销毁
private Material _material;
private void Awake()
{
    _material = GetComponent<Renderer>().material;  // 创建实例
}
private void Update()
{
    _material.color = Color.red;
}
private void OnDestroy()
{
    if (_material != null)
        Destroy(_material);  // 必须销毁!
}

// 或使用sharedMaterial(不创建实例，影响所有使用者)
private void UpdateColorShared()
{
    GetComponent<Renderer>().sharedMaterial.color = Color.red;  // 影响所有实例
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P0 (致命) |
| 性能影响 | 内存泄漏，Material永不释放 |
| 检测方式 | Memory Profiler → Materials |
| 修复方案 | 缓存material并在OnDestroy中Destroy |

---

## 二、内存红线 (Memory)

### P0-010: 事件订阅未取消

```csharp
// ❌ 红线违反 - 只订阅不取消，导致内存泄漏
public class LeakyComponent : MonoBehaviour
{
    private void OnEnable()
    {
        GameEvents.OnPlayerDied += HandlePlayerDeath;  // 订阅
    }
    // 缺少OnDisable取消订阅!
}

// ✅ 正确做法 - 始终配对
public class SafeComponent : MonoBehaviour
{
    private void OnEnable()
    {
        GameEvents.OnPlayerDied += HandlePlayerDeath;
    }
    private void OnDisable()
    {
        GameEvents.OnPlayerDied -= HandlePlayerDeath;  // 必须取消!
    }
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P0 (致命) |
| 性能影响 | 对象无法被GC回收，内存持续增长 |
| 检测方式 | Memory Profiler → 检查已销毁对象仍被引用 |
| 修复方案 | OnEnable订阅 → OnDisable取消 |

---

### P0-011: 协程未在OnDisable停止

```csharp
// ❌ 红线违反 - 对象禁用后协程继续运行
public class LeakyCoroutine : MonoBehaviour
{
    private void Start()
    {
        StartCoroutine(EndlessRoutine());
    }
    // OnDisable中未停止协程!
}

// ✅ 正确做法 - 跟踪并停止协程
public class SafeCoroutine : MonoBehaviour
{
    private Coroutine _routine;
    
    private void OnEnable()
    {
        _routine = StartCoroutine(MyRoutine());
    }
    
    private void OnDisable()
    {
        if (_routine != null)
        {
            StopCoroutine(_routine);
            _routine = null;
        }
        // 或直接 StopAllCoroutines();
    }
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P1 (严重) |
| 性能影响 | 协程在对象禁用后继续运行，可能访问已销毁对象 |
| 检测方式 | 代码审查 / 运行时错误日志 |
| 修复方案 | OnDisable中StopCoroutine或StopAllCoroutines |

---

### P0-012: 静态集合持有场景对象引用

```csharp
// ❌ 红线违反 - 静态集合持有MonoBehaviour引用
public static class EnemyRegistry
{
    public static List<Enemy> AllEnemies = new List<Enemy>();
}

public class Enemy : MonoBehaviour
{
    private void Awake()
    {
        EnemyRegistry.AllEnemies.Add(this);
    }
    // OnDestroy中未移除!
}

// ✅ 正确做法 - 必须在OnDestroy中移除
public class Enemy : MonoBehaviour
{
    private void Awake()
    {
        EnemyRegistry.AllEnemies.Add(this);
    }
    private void OnDestroy()
    {
        EnemyRegistry.AllEnemies.Remove(this);  // 必须!
    }
}

// 更好做法 - 使用RuntimeSet ScriptableObject
[CreateAssetMenu]
public class EnemySet : RuntimeSet<Enemy> { }
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P0 (致命) |
| 性能影响 | 场景切换后旧对象无法GC，内存泄漏 |
| 检测方式 | Memory Profiler → 检查跨场景引用 |
| 修复方案 | OnDestroy中移除 / 使用ScriptableObject RuntimeSet |

---

### P0-013: 异步回调访问已销毁对象

```csharp
// ❌ 红线违反 - 异步完成时对象可能已销毁
public class AsyncDanger : MonoBehaviour
{
    private async void Start()
    {
        var data = await LoadDataAsync();
        _text.text = data;  // 对象可能已销毁!
    }
}

// ✅ 正确做法 - 使用CancellationToken
public class AsyncSafe : MonoBehaviour
{
    private CancellationTokenSource _cts;
    
    private void Awake()
    {
        _cts = new CancellationTokenSource();
    }
    
    private async void Start()
    {
        try
        {
            var data = await LoadDataAsync(_cts.Token);
            _text.text = data;
        }
        catch (OperationCanceledException)
        {
            // 正常取消
        }
    }
    
    private void OnDestroy()
    {
        _cts?.Cancel();
        _cts?.Dispose();
    }
}

// UniTask方式 - 更简洁
public class AsyncUniTask : MonoBehaviour
{
    private async UniTaskVoid Start()
    {
        var data = await LoadDataAsync(this.GetCancellationTokenOnDestroy());
        _text.text = data;  // 安全，对象销毁时自动取消
    }
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P0 (致命) |
| 性能影响 | NullReferenceException / MissingReferenceException |
| 检测方式 | 运行时错误日志 |
| 修复方案 | CancellationToken / UniTask的GetCancellationTokenOnDestroy |

---

## 三、架构红线 (Architecture)

### P0-014: 单例滥用

```csharp
// ❌ 红线违反 - 所有东西都是单例
public class PlayerSingleton : Singleton<PlayerSingleton> { }  // 玩家不应该是单例
public class BulletSingleton : Singleton<BulletSingleton> { }  // 子弹不应该是单例
public class UISingleton : Singleton<UISingleton> { }  // 每个UI都是单例

// ✅ 正确做法 - 只有真正全局唯一的系统使用单例
// 适合单例: GameManager, AudioManager, InputManager, SaveManager
// 不适合单例: Player, Enemy, UI Panel, Weapon
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P2 (重要) |
| 性能影响 | 代码耦合、难以测试、难以扩展 |
| 检测方式 | 代码审查 |
| 修复方案 | 依赖注入 / 事件系统 / ScriptableObject |

---

### P0-015: 深层嵌套回调

```csharp
// ❌ 红线违反 - 回调地狱
public void LoadGame()
{
    LoadPlayerData((playerData) => {
        LoadInventory(playerData.id, (inventory) => {
            LoadQuests(playerData.id, (quests) => {
                LoadAchievements(playerData.id, (achievements) => {
                    InitializeGame(playerData, inventory, quests, achievements);
                });
            });
        });
    });
}

// ✅ 正确做法 - 使用async/await
public async UniTask LoadGame()
{
    var playerData = await LoadPlayerDataAsync();
    var inventory = await LoadInventoryAsync(playerData.id);
    var quests = await LoadQuestsAsync(playerData.id);
    var achievements = await LoadAchievementsAsync(playerData.id);
    InitializeGame(playerData, inventory, quests, achievements);
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P2 (重要) |
| 性能影响 | 代码可读性差、难以调试、难以维护 |
| 检测方式 | 代码审查 |
| 修复方案 | async/await / UniTask / Promise模式 |

---

### P0-016: 组件间循环依赖

```csharp
// ❌ 红线违反 - A依赖B，B依赖A
public class ComponentA : MonoBehaviour
{
    [SerializeField] private ComponentB _b;
    private void Awake()
    {
        _b.Initialize(this);  // A传递给B
    }
}
public class ComponentB : MonoBehaviour
{
    private ComponentA _a;
    public void Initialize(ComponentA a)
    {
        _a = a;
        _a.DoSomething();  // B调用A
    }
}

// ✅ 正确做法 - 使用事件解耦
public class ComponentA : MonoBehaviour
{
    public event Action OnSomethingHappened;
    public void DoSomething()
    {
        OnSomethingHappened?.Invoke();
    }
}
public class ComponentB : MonoBehaviour
{
    [SerializeField] private ComponentA _a;
    private void OnEnable()
    {
        _a.OnSomethingHappened += HandleEvent;
    }
    private void OnDisable()
    {
        _a.OnSomethingHappened -= HandleEvent;
    }
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P2 (重要) |
| 性能影响 | 初始化顺序问题、难以测试、难以理解 |
| 检测方式 | 代码审查 / 依赖关系图 |
| 修复方案 | 事件系统 / 中介者模式 / 依赖注入 |

---

## 四、生命周期红线 (Lifecycle)

### P0-017: 在Awake中引用其他对象

```csharp
// ❌ 红线违反 - Awake时其他对象可能未初始化
private void Awake()
{
    _gameManager = FindObjectOfType<GameManager>();  // 可能为null!
    _player = GameObject.FindWithTag("Player");  // 可能为null!
    _gameManager.RegisterEnemy(this);  // NullReferenceException!
}

// ✅ 正确做法 - Awake初始化自身，Start引用他人
private void Awake()
{
    // 只初始化自身组件
    _rb = GetComponent<Rigidbody>();
    _health = _maxHealth;
}
private void Start()
{
    // 现在安全地引用其他对象
    _gameManager = GameManager.Instance;
    _player = GameObject.FindWithTag("Player");
    _gameManager?.RegisterEnemy(this);
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P1 (严重) |
| 性能影响 | 随机NullReferenceException，依赖脚本执行顺序 |
| 检测方式 | 运行时错误 / 代码审查 |
| 修复方案 | Awake只初始化自身，Start引用他人 |

---

### P0-018: 物理操作在Update中

```csharp
// ❌ 红线违反 - 物理操作在Update中
private void Update()
{
    _rb.AddForce(transform.forward * _force);  // 物理不稳定!
    _rb.MovePosition(transform.position + _velocity * Time.deltaTime);
}

// ✅ 正确做法 - 物理操作在FixedUpdate中
private void FixedUpdate()
{
    _rb.AddForce(transform.forward * _force);
    _rb.MovePosition(transform.position + _velocity * Time.fixedDeltaTime);
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P1 (严重) |
| 性能影响 | 物理模拟不稳定、穿透、抖动 |
| 检测方式 | 代码审查 |
| 修复方案 | Rigidbody操作移到FixedUpdate |

---

### P0-019: 输入检测在FixedUpdate中

```csharp
// ❌ 红线违反 - 输入在FixedUpdate中检测
private void FixedUpdate()
{
    if (Input.GetKeyDown(KeyCode.Space))  // 可能丢失输入!
    {
        Jump();
    }
}

// ✅ 正确做法 - 输入在Update中检测
private bool _jumpPressed;
private void Update()
{
    if (Input.GetKeyDown(KeyCode.Space))
    {
        _jumpPressed = true;
    }
}
private void FixedUpdate()
{
    if (_jumpPressed)
    {
        Jump();
        _jumpPressed = false;
    }
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P1 (严重) |
| 性能影响 | 输入丢失，玩家操作不响应 |
| 检测方式 | 代码审查 / 玩家反馈 |
| 修复方案 | Update检测输入，FixedUpdate处理物理 |

---

## 五、序列化红线 (Serialization)

### P0-020: 公共字段暴露

```csharp
// ❌ 红线违反 - 使用public字段
public class BadSerialization : MonoBehaviour
{
    public int health = 100;  // 破坏封装!
    public float speed = 5f;
    public GameObject prefab;
}

// ✅ 正确做法 - [SerializeField] private
public class GoodSerialization : MonoBehaviour
{
    [SerializeField] private int _health = 100;
    [SerializeField] private float _speed = 5f;
    [SerializeField] private GameObject _prefab;
    
    public int Health => _health;  // 只读属性暴露
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P3 (建议) |
| 性能影响 | 无直接性能影响，但破坏封装性 |
| 检测方式 | 代码审查 |
| 修复方案 | 使用[SerializeField] private + 只读属性 |

---

### P0-021: 属性期望被序列化

```csharp
// ❌ 红线违反 - 期望属性在Inspector中显示
public class PropertyMistake : MonoBehaviour
{
    public int Health { get; set; } = 100;  // 不会序列化!
}

// ✅ 正确做法
public class PropertyCorrect : MonoBehaviour
{
    // 方式1: 传统字段+属性
    [SerializeField] private int _health = 100;
    public int Health => _health;
    
    // 方式2: field属性 (Unity 2019.3+)
    [field: SerializeField] public int Health { get; private set; } = 100;
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P2 (重要) |
| 性能影响 | 值不保存，每次运行重置 |
| 检测方式 | Inspector检查 / 运行时验证 |
| 修复方案 | 使用字段或[field: SerializeField] |

---

## 六、编辑器红线 (Editor)

### P0-022: 编辑器代码未用条件编译

```csharp
// ❌ 红线违反 - 编辑器代码在运行时编译
public class MixedCode : MonoBehaviour
{
    private void OnValidate()  // 仅Editor调用，但会编译
    {
        // ...
    }
    
    public void DoSomething()
    {
        UnityEditor.Selection.activeObject = gameObject;  // 构建失败!
    }
}

// ✅ 正确做法 - 条件编译
public class SafeCode : MonoBehaviour
{
    #if UNITY_EDITOR
    private void OnValidate()
    {
        // 安全
    }
    #endif
    
    public void DoSomething()
    {
        #if UNITY_EDITOR
        UnityEditor.Selection.activeObject = gameObject;
        #endif
    }
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P0 (致命) |
| 性能影响 | 构建失败 |
| 检测方式 | 构建测试 |
| 修复方案 | #if UNITY_EDITOR 包裹所有编辑器代码 |

---

### P0-023: Editor脚本不在Editor文件夹

```csharp
// ❌ 红线违反 - CustomEditor在普通文件夹
// Assets/Scripts/MyEditor.cs
[CustomEditor(typeof(MyScript))]
public class MyEditor : Editor { }  // 构建失败!

// ✅ 正确做法 - 放在Editor文件夹
// Assets/Scripts/Editor/MyEditor.cs
[CustomEditor(typeof(MyScript))]
public class MyEditor : Editor { }  // 正确，自动排除构建
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P0 (致命) |
| 性能影响 | 构建失败 |
| 检测方式 | 构建测试 |
| 修复方案 | 移动到Editor文件夹 |

---

## 七、物理红线 (Physics)

### P0-024: Raycast无LayerMask

```csharp
// ❌ 红线违反 - Raycast检测所有层
if (Physics.Raycast(origin, direction, out hit))  // 检测所有物体!
{
    // ...
}

// ✅ 正确做法 - 使用LayerMask
private int _enemyLayer;
private void Awake()
{
    _enemyLayer = 1 << LayerMask.NameToLayer("Enemy");
}
private void Update()
{
    if (Physics.Raycast(origin, direction, out hit, maxDistance, _enemyLayer))
    {
        // 只检测Enemy层
    }
}
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P1 (严重) |
| 性能影响 | 不必要的碰撞检测，性能浪费 |
| 检测方式 | Profiler → Physics |
| 修复方案 | 使用LayerMask限制检测层 |

---

### P0-025: 碰撞矩阵未配置

```csharp
// ❌ 红线违反 - 所有层互相碰撞
// Edit > Project Settings > Physics > Layer Collision Matrix 全部勾选

// ✅ 正确做法 - 只启用必要的碰撞
// Player vs Enemy: ✓
// Player vs Pickup: ✓
// Enemy vs Enemy: ✗
// Bullet vs Bullet: ✗
// UI vs 所有: ✗
```

| 属性 | 值 |
|------|-----|
| 严重级别 | P1 (严重) |
| 性能影响 | 大量不必要的碰撞检测 |
| 检测方式 | Project Settings检查 |
| 修复方案 | 配置Layer Collision Matrix |

---

## 八、快速参考表

### 按严重级别排序

| 级别 | 红线ID | 简述 |
|------|--------|------|
| P0 | 001 | Update中GetComponent |
| P0 | 002 | Update中Find方法 |
| P0 | 003 | Update中字符串拼接 |
| P0 | 004 | Update中new分配 |
| P0 | 009 | Material泄漏 |
| P0 | 010 | 事件订阅未取消 |
| P0 | 012 | 静态集合持有场景对象 |
| P0 | 013 | 异步回调访问已销毁对象 |
| P0 | 022 | 编辑器代码未条件编译 |
| P0 | 023 | Editor脚本位置错误 |
| P1 | 005 | 空Update方法 |
| P1 | 006 | LINQ在热路径 |
| P1 | 007 | Camera.main未缓存 |
| P1 | 008 | WaitForSeconds未缓存 |
| P1 | 011 | 协程未停止 |
| P1 | 017 | Awake中引用他人 |
| P1 | 018 | 物理在Update |
| P1 | 019 | 输入在FixedUpdate |
| P1 | 024 | Raycast无LayerMask |
| P1 | 025 | 碰撞矩阵未配置 |
| P2 | 014 | 单例滥用 |
| P2 | 015 | 回调地狱 |
| P2 | 016 | 循环依赖 |
| P2 | 021 | 属性序列化误解 |
| P3 | 020 | 公共字段暴露 |

### 检查命令速记

```bash
# Profiler检查点
1. GC Alloc列 > 0 in Update → 检查红线001-004, 006, 008
2. Scripts → GetComponent → 检查红线001
3. Scripts → GameObject.Find → 检查红线002
4. Memory → Materials增长 → 检查红线009

# 代码审查检查点
1. 搜索 "void Update" 检查内容
2. 搜索 "GetComponent" 检查是否在热路径
3. 搜索 "OnEnable" 确认有对应OnDisable
4. 搜索 "StartCoroutine" 确认有停止逻辑
5. 搜索 "async" 确认有CancellationToken
6. 搜索 "#if UNITY_EDITOR" 确认编辑器代码隔离
```

---

## 九、自动化检测建议

### EditorWindow检测脚本示例

```csharp
#if UNITY_EDITOR
using UnityEditor;
using UnityEngine;
using System.Linq;
using System.Reflection;

public class RedlineChecker : EditorWindow
{
    [MenuItem("Tools/Redline Checker")]
    static void ShowWindow() => GetWindow<RedlineChecker>("Redline Checker");

    private void OnGUI()
    {
        if (GUILayout.Button("Check for Empty Update Methods"))
        {
            CheckEmptyUpdates();
        }
        
        if (GUILayout.Button("Check for Public Fields"))
        {
            CheckPublicFields();
        }
    }

    private void CheckEmptyUpdates()
    {
        var scripts = Resources.FindObjectsOfTypeAll<MonoScript>();
        foreach (var script in scripts)
        {
            var type = script.GetClass();
            if (type == null) continue;
            
            var updateMethod = type.GetMethod("Update", 
                BindingFlags.Instance | BindingFlags.NonPublic | BindingFlags.Public);
            if (updateMethod != null)
            {
                // 检查方法体是否为空
                Debug.Log($"Found Update in: {type.Name}");
            }
        }
    }

    private void CheckPublicFields()
    {
        var scripts = Resources.FindObjectsOfTypeAll<MonoScript>();
        foreach (var script in scripts)
        {
            var type = script.GetClass();
            if (type == null || !type.IsSubclassOf(typeof(MonoBehaviour))) continue;
            
            var publicFields = type.GetFields(BindingFlags.Instance | BindingFlags.Public)
                .Where(f => !f.IsInitOnly && f.GetCustomAttribute<HideInInspector>() == null);
            
            foreach (var field in publicFields)
            {
                Debug.LogWarning($"Public field '{field.Name}' in {type.Name} - consider using [SerializeField] private");
            }
        }
    }
}
#endif
```
