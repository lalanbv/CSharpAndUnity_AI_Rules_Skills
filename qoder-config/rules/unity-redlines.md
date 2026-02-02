---
description: Unity C#开发性能红线与禁止事项清单，在编写Unity C#代码时自动应用
alwaysApply: false
globs:
  - "*.cs"
---

# Unity 开发红线清单

> 严重级别: P0(致命) > P1(严重) > P2(重要) > P3(建议)

## P0 致命红线 (必须避免)

### P0-001: Update中调用GetComponent
```csharp
// ❌ 禁止
void Update() { GetComponent<Rigidbody>().velocity = Vector3.forward; }

// ✅ 正确 - Awake中缓存
private Rigidbody _rb;
void Awake() { _rb = GetComponent<Rigidbody>(); }
void Update() { _rb.velocity = Vector3.forward; }
```

### P0-002: Update中使用Find方法
```csharp
// ❌ 禁止
void Update() {
    GameObject player = GameObject.Find("Player");
    var manager = FindObjectOfType<GameManager>();
}

// ✅ 正确 - Start中缓存
private GameObject _player;
void Start() { _player = GameObject.Find("Player"); }
```

### P0-003: Update中字符串拼接
```csharp
// ❌ 禁止 - 每帧GC
void Update() { _scoreText.text = "Score: " + _score; }

// ✅ 正确 - 只在值变化时更新
void OnScoreChanged(int newScore) { _scoreText.text = newScore.ToString(); }
```

### P0-004: Update中new分配
```csharp
// ❌ 禁止
void Update() { List<Enemy> enemies = new List<Enemy>(); }

// ✅ 正确 - 预分配并复用
private List<Enemy> _enemyCache = new List<Enemy>(64);
void Update() { _enemyCache.Clear(); }
```

### P0-009: Material泄漏
```csharp
// ❌ 禁止 - 访问.material创建实例，从不销毁
void Update() { GetComponent<Renderer>().material.color = Color.red; }

// ✅ 正确 - 缓存并在OnDestroy中销毁
private Material _material;
void Awake() { _material = GetComponent<Renderer>().material; }
void OnDestroy() { if (_material != null) Destroy(_material); }
```

### P0-010: 事件订阅未取消
```csharp
// ❌ 禁止 - 只订阅不取消
void OnEnable() { GameEvents.OnPlayerDied += HandlePlayerDeath; }
// 缺少OnDisable!

// ✅ 正确 - 配对
void OnEnable() { GameEvents.OnPlayerDied += HandlePlayerDeath; }
void OnDisable() { GameEvents.OnPlayerDied -= HandlePlayerDeath; }
```

### P0-012: 静态集合持有场景对象引用
```csharp
// ❌ 禁止
public static List<Enemy> AllEnemies = new List<Enemy>();
void Awake() { AllEnemies.Add(this); } // OnDestroy未移除!

// ✅ 正确
void Awake() { AllEnemies.Add(this); }
void OnDestroy() { AllEnemies.Remove(this); }
```

### P0-013: 异步回调访问已销毁对象
```csharp
// ❌ 禁止
async void Start() {
    var data = await LoadDataAsync();
    _text.text = data;  // 对象可能已销毁!
}

// ✅ 正确 - 使用CancellationToken
private CancellationTokenSource _cts;
void OnDestroy() { _cts?.Cancel(); _cts?.Dispose(); }
```

### P0-022: 编辑器代码未用条件编译
```csharp
// ❌ 禁止 - 构建失败
public void DoSomething() {
    UnityEditor.Selection.activeObject = gameObject;
}

// ✅ 正确
#if UNITY_EDITOR
public void DoSomething() {
    UnityEditor.Selection.activeObject = gameObject;
}
#endif
```

## P1 严重红线 (应当避免)

### P1-005: 空的Update方法
```csharp
// ❌ 避免 - 仍有开销
void Update() { }
void FixedUpdate() { }

// ✅ 正确 - 删除不需要的方法
```

### P1-006: LINQ在热路径
```csharp
// ❌ 避免
void Update() { var active = _enemies.Where(e => e.IsActive).ToList(); }

// ✅ 正确 - 使用for循环
for (int i = 0; i < _enemies.Count; i++) { if (_enemies[i].IsActive) ... }
```

### P1-007: Camera.main未缓存
```csharp
// ❌ 避免
void Update() { Camera.main.WorldToScreenPoint(transform.position); }

// ✅ 正确
private Camera _mainCamera;
void Awake() { _mainCamera = Camera.main; }
```

### P1-008: WaitForSeconds未缓存
```csharp
// ❌ 避免
IEnumerator Routine() { yield return new WaitForSeconds(1f); }

// ✅ 正确
private readonly WaitForSeconds _wait = new WaitForSeconds(1f);
IEnumerator Routine() { yield return _wait; }
```

### P1-017: 在Awake中引用其他对象
```csharp
// ❌ 避免 - 其他对象可能未初始化
void Awake() { _gameManager = FindObjectOfType<GameManager>(); }

// ✅ 正确
void Awake() { _rb = GetComponent<Rigidbody>(); }
void Start() { _gameManager = GameManager.Instance; }
```

### P1-018: 物理操作在Update中
```csharp
// ❌ 避免
void Update() { _rb.AddForce(transform.forward * _force); }

// ✅ 正确
void FixedUpdate() { _rb.AddForce(transform.forward * _force); }
```

### P1-019: 输入检测在FixedUpdate中
```csharp
// ❌ 避免 - 可能丢失输入
void FixedUpdate() { if (Input.GetKeyDown(KeyCode.Space)) Jump(); }

// ✅ 正确
private bool _jumpPressed;
void Update() { if (Input.GetKeyDown(KeyCode.Space)) _jumpPressed = true; }
void FixedUpdate() { if (_jumpPressed) { Jump(); _jumpPressed = false; } }
```

## 代码审查自动检查点

生成或修改Unity C#代码后，自动检查:
1. GetComponent 是否已缓存
2. 事件订阅是否配对取消
3. 协程 WaitForSeconds 是否缓存
4. 编辑器代码是否正确隔离
5. 序列化字段是否使用 [SerializeField] private
6. 没有空的Update/FixedUpdate/LateUpdate方法
