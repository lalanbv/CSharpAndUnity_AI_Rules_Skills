# Unity 代码模板

> 开箱即用的 Unity 代码模板，遵循最佳实践

---

## 一、MonoBehaviour 模板

### 1.1 标准 MonoBehaviour

```csharp
using UnityEngine;
using System;

namespace YourProject.Gameplay
{
    /// <summary>
    /// [类描述]
    /// </summary>
    [RequireComponent(typeof(Rigidbody))]  // 按需添加
    public class StandardComponent : MonoBehaviour
    {
        #region Serialized Fields
        [Header("Settings")]
        [Tooltip("描述此字段的用途")]
        [SerializeField] private float _moveSpeed = 5f;
        
        [Header("References")]
        [SerializeField] private Transform _targetPoint;
        #endregion

        #region Private Fields
        private Rigidbody _rigidbody;
        private bool _isInitialized;
        #endregion

        #region Events
        public event Action OnSomethingHappened;
        public event Action<int> OnValueChanged;
        #endregion

        #region Properties
        public float MoveSpeed => _moveSpeed;
        public bool IsInitialized => _isInitialized;
        #endregion

        #region Unity Lifecycle
        private void Awake()
        {
            CacheComponents();
        }

        private void OnEnable()
        {
            SubscribeEvents();
        }

        private void Start()
        {
            Initialize();
        }

        private void Update()
        {
            if (!_isInitialized) return;
            HandleUpdate();
        }

        private void OnDisable()
        {
            UnsubscribeEvents();
        }

        private void OnDestroy()
        {
            Cleanup();
        }
        #endregion

        #region Initialization
        private void CacheComponents()
        {
            _rigidbody = GetComponent<Rigidbody>();
        }

        private void Initialize()
        {
            _isInitialized = true;
        }

        private void SubscribeEvents()
        {
            // 订阅外部事件
        }

        private void UnsubscribeEvents()
        {
            // 取消订阅
        }

        private void Cleanup()
        {
            // 释放资源
        }
        #endregion

        #region Public Methods
        public void DoSomething()
        {
            OnSomethingHappened?.Invoke();
        }
        #endregion

        #region Private Methods
        private void HandleUpdate()
        {
            // Update逻辑
        }
        #endregion

        #region Editor
        #if UNITY_EDITOR
        private void OnValidate()
        {
            _moveSpeed = Mathf.Max(0f, _moveSpeed);
        }

        private void OnDrawGizmosSelected()
        {
            if (_targetPoint != null)
            {
                Gizmos.color = Color.yellow;
                Gizmos.DrawLine(transform.position, _targetPoint.position);
            }
        }
        #endif
        #endregion
    }
}
```

### 1.2 单例管理器模板

```csharp
using UnityEngine;

namespace YourProject.Core
{
    /// <summary>
    /// [管理器描述]
    /// </summary>
    public class GameManager : MonoBehaviour
    {
        #region Singleton
        public static GameManager Instance { get; private set; }

        private void InitializeSingleton()
        {
            if (Instance != null && Instance != this)
            {
                Destroy(gameObject);
                return;
            }
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }
        #endregion

        #region Serialized Fields
        [Header("Settings")]
        [SerializeField] private bool _debugMode = false;
        #endregion

        #region Private Fields
        private bool _isInitialized;
        #endregion

        #region Events
        public event System.Action OnGameStarted;
        public event System.Action OnGamePaused;
        public event System.Action OnGameResumed;
        public event System.Action OnGameEnded;
        #endregion

        #region Properties
        public bool IsDebugMode => _debugMode;
        public bool IsInitialized => _isInitialized;
        #endregion

        #region Unity Lifecycle
        private void Awake()
        {
            InitializeSingleton();
            if (Instance != this) return;
            
            Initialize();
        }

        private void OnDestroy()
        {
            if (Instance == this)
            {
                Instance = null;
            }
        }
        #endregion

        #region Initialization
        private void Initialize()
        {
            // 初始化管理器
            _isInitialized = true;
        }
        #endregion

        #region Public Methods
        public void StartGame()
        {
            OnGameStarted?.Invoke();
        }

        public void PauseGame()
        {
            Time.timeScale = 0f;
            OnGamePaused?.Invoke();
        }

        public void ResumeGame()
        {
            Time.timeScale = 1f;
            OnGameResumed?.Invoke();
        }

        public void EndGame()
        {
            OnGameEnded?.Invoke();
        }
        #endregion
    }
}
```

### 1.3 泛型单例基类

```csharp
using UnityEngine;

namespace YourProject.Core
{
    /// <summary>
    /// 泛型单例基类
    /// </summary>
    public abstract class Singleton<T> : MonoBehaviour where T : MonoBehaviour
    {
        private static T _instance;
        private static readonly object _lock = new object();
        private static bool _applicationIsQuitting = false;

        public static T Instance
        {
            get
            {
                if (_applicationIsQuitting)
                {
                    Debug.LogWarning($"[Singleton] Instance of {typeof(T)} already destroyed on application quit.");
                    return null;
                }

                lock (_lock)
                {
                    if (_instance == null)
                    {
                        _instance = FindObjectOfType<T>();

                        if (_instance == null)
                        {
                            var singleton = new GameObject($"[{typeof(T).Name}]");
                            _instance = singleton.AddComponent<T>();
                            DontDestroyOnLoad(singleton);
                        }
                    }
                    return _instance;
                }
            }
        }

        protected virtual void Awake()
        {
            if (_instance == null)
            {
                _instance = this as T;
                DontDestroyOnLoad(gameObject);
                OnAwake();
            }
            else if (_instance != this)
            {
                Destroy(gameObject);
            }
        }

        protected virtual void OnAwake() { }

        protected virtual void OnDestroy()
        {
            if (_instance == this)
            {
                _applicationIsQuitting = true;
            }
        }
    }
    
    // 使用示例
    public class AudioManager : Singleton<AudioManager>
    {
        protected override void OnAwake()
        {
            // 初始化音频系统
        }
    }
}
```

---

## 二、ScriptableObject 模板

### 2.1 数据容器

```csharp
using UnityEngine;

namespace YourProject.Data
{
    /// <summary>
    /// [数据描述]
    /// </summary>
    [CreateAssetMenu(fileName = "NewData", menuName = "YourProject/Data/DataName")]
    public class DataContainer : ScriptableObject
    {
        [Header("Basic Info")]
        [SerializeField] private string _itemName;
        [SerializeField] private Sprite _icon;
        [TextArea(2, 5)]
        [SerializeField] private string _description;

        [Header("Stats")]
        [SerializeField] private int _value = 100;
        [Range(0f, 1f)]
        [SerializeField] private float _rarity = 0.5f;

        // 只读属性
        public string ItemName => _itemName;
        public Sprite Icon => _icon;
        public string Description => _description;
        public int Value => _value;
        public float Rarity => _rarity;

        #if UNITY_EDITOR
        private void OnValidate()
        {
            if (string.IsNullOrEmpty(_itemName))
            {
                _itemName = name;
            }
        }
        #endif
    }
}
```

### 2.2 运行时变量

```csharp
using UnityEngine;
using System;

namespace YourProject.Variables
{
    /// <summary>
    /// 运行时整数变量
    /// </summary>
    [CreateAssetMenu(fileName = "IntVariable", menuName = "YourProject/Variables/Int")]
    public class IntVariable : ScriptableObject
    {
        [SerializeField] private int _initialValue;
        
        [NonSerialized] private int _runtimeValue;
        [NonSerialized] private bool _isInitialized;

        public int Value
        {
            get
            {
                EnsureInitialized();
                return _runtimeValue;
            }
            set
            {
                EnsureInitialized();
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
            _isInitialized = false;
        }

        private void EnsureInitialized()
        {
            if (!_isInitialized)
            {
                _runtimeValue = _initialValue;
                _isInitialized = true;
            }
        }

        public void Reset()
        {
            Value = _initialValue;
        }

        public void Add(int amount)
        {
            Value += amount;
        }

        public void Subtract(int amount)
        {
            Value -= amount;
        }
    }
}
```

### 2.3 ScriptableObject 事件

```csharp
using UnityEngine;
using UnityEngine.Events;
using System.Collections.Generic;

namespace YourProject.Events
{
    /// <summary>
    /// ScriptableObject 事件通道
    /// </summary>
    [CreateAssetMenu(fileName = "GameEvent", menuName = "YourProject/Events/Game Event")]
    public class GameEventSO : ScriptableObject
    {
        private readonly HashSet<GameEventListener> _listeners = new HashSet<GameEventListener>();

        #if UNITY_EDITOR
        [TextArea(2, 4)]
        [SerializeField] private string _description = "Event description";
        #endif

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

    /// <summary>
    /// 事件监听器组件
    /// </summary>
    public class GameEventListener : MonoBehaviour
    {
        [SerializeField] private GameEventSO _event;
        [SerializeField] private UnityEvent _response;

        private void OnEnable()
        {
            _event?.RegisterListener(this);
        }

        private void OnDisable()
        {
            _event?.UnregisterListener(this);
        }

        public void OnEventRaised()
        {
            _response?.Invoke();
        }
    }
}
```

---

## 三、对象池模板

### 3.1 泛型对象池

```csharp
using UnityEngine;
using System;
using System.Collections.Generic;

namespace YourProject.Pooling
{
    /// <summary>
    /// 泛型对象池
    /// </summary>
    public class ObjectPool<T> where T : Component
    {
        private readonly T _prefab;
        private readonly Transform _parent;
        private readonly Queue<T> _available = new Queue<T>();
        private readonly HashSet<T> _inUse = new HashSet<T>();
        private readonly Action<T> _onGet;
        private readonly Action<T> _onReturn;

        public int AvailableCount => _available.Count;
        public int InUseCount => _inUse.Count;
        public int TotalCount => AvailableCount + InUseCount;

        public ObjectPool(
            T prefab, 
            int initialSize = 10, 
            Transform parent = null,
            Action<T> onGet = null,
            Action<T> onReturn = null)
        {
            _prefab = prefab;
            _parent = parent;
            _onGet = onGet;
            _onReturn = onReturn;

            // 预热
            for (int i = 0; i < initialSize; i++)
            {
                T obj = CreateInstance();
                obj.gameObject.SetActive(false);
                _available.Enqueue(obj);
            }
        }

        public T Get()
        {
            T obj = _available.Count > 0 
                ? _available.Dequeue() 
                : CreateInstance();

            obj.gameObject.SetActive(true);
            _inUse.Add(obj);
            _onGet?.Invoke(obj);

            return obj;
        }

        public T Get(Vector3 position, Quaternion rotation)
        {
            T obj = Get();
            obj.transform.SetPositionAndRotation(position, rotation);
            return obj;
        }

        public void Return(T obj)
        {
            if (obj == null || !_inUse.Contains(obj))
                return;

            _onReturn?.Invoke(obj);
            obj.gameObject.SetActive(false);
            _inUse.Remove(obj);
            _available.Enqueue(obj);
        }

        public void ReturnAll()
        {
            foreach (var obj in new List<T>(_inUse))
            {
                Return(obj);
            }
        }

        public void Clear()
        {
            foreach (var obj in _available)
            {
                if (obj != null)
                    UnityEngine.Object.Destroy(obj.gameObject);
            }
            _available.Clear();

            foreach (var obj in _inUse)
            {
                if (obj != null)
                    UnityEngine.Object.Destroy(obj.gameObject);
            }
            _inUse.Clear();
        }

        private T CreateInstance()
        {
            return UnityEngine.Object.Instantiate(_prefab, _parent);
        }
    }

    // 使用示例
    public class BulletSpawner : MonoBehaviour
    {
        [SerializeField] private Bullet _bulletPrefab;
        [SerializeField] private int _poolSize = 50;

        private ObjectPool<Bullet> _bulletPool;

        private void Awake()
        {
            _bulletPool = new ObjectPool<Bullet>(
                _bulletPrefab, 
                _poolSize, 
                transform,
                onGet: bullet => bullet.ResetState(),
                onReturn: bullet => bullet.ClearEffects()
            );
        }

        public void Fire(Vector3 position, Vector3 direction)
        {
            var bullet = _bulletPool.Get(position, Quaternion.LookRotation(direction));
            bullet.Initialize(direction, () => _bulletPool.Return(bullet));
        }

        private void OnDestroy()
        {
            _bulletPool?.Clear();
        }
    }
}
```

### 3.2 可池化接口

```csharp
using UnityEngine;

namespace YourProject.Pooling
{
    /// <summary>
    /// 可池化对象接口
    /// </summary>
    public interface IPoolable
    {
        void OnSpawn();
        void OnDespawn();
    }

    /// <summary>
    /// 可池化组件基类
    /// </summary>
    public abstract class PoolableComponent : MonoBehaviour, IPoolable
    {
        public virtual void OnSpawn()
        {
            // 从池中取出时调用
        }

        public virtual void OnDespawn()
        {
            // 返回池中时调用
        }
    }

    /// <summary>
    /// 支持IPoolable的对象池
    /// </summary>
    public class SmartPool<T> where T : Component, IPoolable
    {
        private readonly ObjectPool<T> _pool;

        public SmartPool(T prefab, int initialSize = 10, Transform parent = null)
        {
            _pool = new ObjectPool<T>(
                prefab,
                initialSize,
                parent,
                onGet: obj => obj.OnSpawn(),
                onReturn: obj => obj.OnDespawn()
            );
        }

        public T Get() => _pool.Get();
        public T Get(Vector3 position, Quaternion rotation) => _pool.Get(position, rotation);
        public void Return(T obj) => _pool.Return(obj);
        public void ReturnAll() => _pool.ReturnAll();
        public void Clear() => _pool.Clear();
    }
}
```

---

## 四、事件系统模板

### 4.1 类型安全事件总线

```csharp
using System;
using System.Collections.Generic;

namespace YourProject.Events
{
    /// <summary>
    /// 类型安全的事件总线
    /// </summary>
    public static class EventBus<T> where T : struct
    {
        private static readonly HashSet<Action<T>> _handlers = new HashSet<Action<T>>();

        public static void Subscribe(Action<T> handler)
        {
            _handlers.Add(handler);
        }

        public static void Unsubscribe(Action<T> handler)
        {
            _handlers.Remove(handler);
        }

        public static void Publish(T eventData)
        {
            foreach (var handler in _handlers)
            {
                try
                {
                    handler?.Invoke(eventData);
                }
                catch (Exception e)
                {
                    UnityEngine.Debug.LogException(e);
                }
            }
        }

        public static void Clear()
        {
            _handlers.Clear();
        }
    }

    // 事件定义示例
    public struct PlayerDiedEvent
    {
        public UnityEngine.Vector3 Position;
        public string CauseOfDeath;
        public int FinalScore;
    }

    public struct EnemyKilledEvent
    {
        public UnityEngine.GameObject Enemy;
        public int ExperienceReward;
    }

    public struct ItemCollectedEvent
    {
        public string ItemId;
        public int Quantity;
    }

    // 使用示例
    /*
    // 订阅
    private void OnEnable()
    {
        EventBus<PlayerDiedEvent>.Subscribe(OnPlayerDied);
    }
    
    private void OnDisable()
    {
        EventBus<PlayerDiedEvent>.Unsubscribe(OnPlayerDied);
    }
    
    // 发布
    EventBus<PlayerDiedEvent>.Publish(new PlayerDiedEvent
    {
        Position = transform.position,
        CauseOfDeath = "Enemy",
        FinalScore = _score
    });
    */
}
```

---

## 五、编辑器模板

### 5.1 EditorWindow 模板

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;
using System.Collections.Generic;

namespace YourProject.Editor
{
    /// <summary>
    /// [窗口描述]
    /// </summary>
    public class CustomEditorWindow : EditorWindow
    {
        #region Fields
        private Vector2 _scrollPosition;
        private string _searchText = "";
        private bool _showSettings = true;
        #endregion

        #region Menu Item
        [MenuItem("Tools/YourProject/Custom Window")]
        private static void ShowWindow()
        {
            var window = GetWindow<CustomEditorWindow>();
            window.titleContent = new GUIContent("Custom Window", EditorGUIUtility.IconContent("d_Settings").image);
            window.minSize = new Vector2(400, 300);
            window.Show();
        }
        #endregion

        #region Unity Callbacks
        private void OnEnable()
        {
            // 初始化
        }

        private void OnDisable()
        {
            // 清理
        }

        private void OnGUI()
        {
            DrawToolbar();
            DrawMainContent();
            DrawFooter();
        }
        #endregion

        #region GUI Drawing
        private void DrawToolbar()
        {
            EditorGUILayout.BeginHorizontal(EditorStyles.toolbar);
            
            if (GUILayout.Button("Refresh", EditorStyles.toolbarButton, GUILayout.Width(60)))
            {
                Refresh();
            }

            GUILayout.FlexibleSpace();

            _searchText = EditorGUILayout.TextField(_searchText, EditorStyles.toolbarSearchField, GUILayout.Width(200));

            if (GUILayout.Button("", EditorStyles.toolbarButton))
            {
                _searchText = "";
                GUI.FocusControl(null);
            }

            EditorGUILayout.EndHorizontal();
        }

        private void DrawMainContent()
        {
            _scrollPosition = EditorGUILayout.BeginScrollView(_scrollPosition);

            // 设置折叠区域
            _showSettings = EditorGUILayout.Foldout(_showSettings, "Settings", true);
            if (_showSettings)
            {
                EditorGUI.indentLevel++;
                DrawSettings();
                EditorGUI.indentLevel--;
            }

            EditorGUILayout.Space(10);

            // 主要内容
            EditorGUILayout.LabelField("Content", EditorStyles.boldLabel);
            EditorGUILayout.HelpBox("Your content here", MessageType.Info);

            // 操作按钮
            EditorGUILayout.Space(10);
            if (GUILayout.Button("Execute Action", GUILayout.Height(30)))
            {
                ExecuteAction();
            }

            EditorGUILayout.EndScrollView();
        }

        private void DrawSettings()
        {
            // 绘制设置选项
        }

        private void DrawFooter()
        {
            EditorGUILayout.BeginHorizontal(EditorStyles.helpBox);
            EditorGUILayout.LabelField("Status: Ready", EditorStyles.miniLabel);
            EditorGUILayout.EndHorizontal();
        }
        #endregion

        #region Actions
        private void Refresh()
        {
            Debug.Log("Refreshing...");
            Repaint();
        }

        private void ExecuteAction()
        {
            if (EditorUtility.DisplayDialog("Confirm", "Execute action?", "Yes", "No"))
            {
                // 执行操作
                Debug.Log("Action executed!");
            }
        }
        #endregion
    }
}
#endif
```

### 5.2 CustomEditor 模板

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;

namespace YourProject.Editor
{
    /// <summary>
    /// [目标脚本] 的自定义Inspector
    /// </summary>
    [CustomEditor(typeof(TargetScript))]
    [CanEditMultipleObjects]
    public class TargetScriptEditor : UnityEditor.Editor
    {
        #region Serialized Properties
        private SerializedProperty _propHealth;
        private SerializedProperty _propSpeed;
        private SerializedProperty _propPrefab;
        #endregion

        #region State
        private bool _showAdvanced = false;
        #endregion

        #region Unity Callbacks
        private void OnEnable()
        {
            // 缓存序列化属性
            _propHealth = serializedObject.FindProperty("_health");
            _propSpeed = serializedObject.FindProperty("_speed");
            _propPrefab = serializedObject.FindProperty("_prefab");
        }

        public override void OnInspectorGUI()
        {
            serializedObject.Update();

            DrawHeader();
            DrawBasicProperties();
            DrawAdvancedSection();
            DrawActionButtons();

            serializedObject.ApplyModifiedProperties();
        }

        private void OnSceneGUI()
        {
            TargetScript script = (TargetScript)target;

            // 在场景视图绘制辅助图形
            Handles.color = Color.yellow;
            Handles.DrawWireDisc(script.transform.position, Vector3.up, 2f);

            // 可拖动控制点
            EditorGUI.BeginChangeCheck();
            Vector3 newPos = Handles.PositionHandle(script.transform.position, Quaternion.identity);
            if (EditorGUI.EndChangeCheck())
            {
                Undo.RecordObject(script.transform, "Move Target");
                script.transform.position = newPos;
            }
        }
        #endregion

        #region Drawing Methods
        private void DrawHeader()
        {
            EditorGUILayout.LabelField("Target Script Settings", EditorStyles.boldLabel);
            EditorGUILayout.Space();
        }

        private void DrawBasicProperties()
        {
            EditorGUILayout.PropertyField(_propHealth);
            EditorGUILayout.PropertyField(_propSpeed);
            EditorGUILayout.PropertyField(_propPrefab);
        }

        private void DrawAdvancedSection()
        {
            EditorGUILayout.Space();
            _showAdvanced = EditorGUILayout.Foldout(_showAdvanced, "Advanced Settings", true);
            
            if (_showAdvanced)
            {
                EditorGUI.indentLevel++;
                EditorGUILayout.HelpBox("Advanced configuration options", MessageType.None);
                // 高级设置...
                EditorGUI.indentLevel--;
            }
        }

        private void DrawActionButtons()
        {
            EditorGUILayout.Space();
            EditorGUILayout.BeginHorizontal();

            if (GUILayout.Button("Reset", GUILayout.Height(25)))
            {
                ResetValues();
            }

            if (GUILayout.Button("Test", GUILayout.Height(25)))
            {
                TestAction();
            }

            EditorGUILayout.EndHorizontal();
        }
        #endregion

        #region Actions
        private void ResetValues()
        {
            Undo.RecordObject(target, "Reset TargetScript");
            _propHealth.intValue = 100;
            _propSpeed.floatValue = 5f;
        }

        private void TestAction()
        {
            foreach (var t in targets)
            {
                var script = t as TargetScript;
                if (script != null)
                {
                    Debug.Log($"Testing {script.name}");
                }
            }
        }
        #endregion
    }
}
#endif
```

### 5.3 PropertyDrawer 模板

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;

namespace YourProject.Editor
{
    /// <summary>
    /// 自定义属性绘制器
    /// </summary>
    [CustomPropertyDrawer(typeof(RangeValue))]
    public class RangeValueDrawer : PropertyDrawer
    {
        private const float PADDING = 2f;

        public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
        {
            EditorGUI.BeginProperty(position, label, property);

            // 绘制标签
            position = EditorGUI.PrefixLabel(position, GUIUtility.GetControlID(FocusType.Passive), label);

            // 保存缩进级别
            int indent = EditorGUI.indentLevel;
            EditorGUI.indentLevel = 0;

            // 获取属性
            var minProp = property.FindPropertyRelative("min");
            var maxProp = property.FindPropertyRelative("max");

            // 计算矩形
            float halfWidth = (position.width - PADDING) / 2f;
            Rect minRect = new Rect(position.x, position.y, halfWidth, position.height);
            Rect maxRect = new Rect(position.x + halfWidth + PADDING, position.y, halfWidth, position.height);

            // 绘制字段
            EditorGUI.PropertyField(minRect, minProp, GUIContent.none);
            EditorGUI.PropertyField(maxRect, maxProp, GUIContent.none);

            // 验证 min <= max
            if (minProp.floatValue > maxProp.floatValue)
            {
                maxProp.floatValue = minProp.floatValue;
            }

            // 恢复缩进级别
            EditorGUI.indentLevel = indent;

            EditorGUI.EndProperty();
        }

        public override float GetPropertyHeight(SerializedProperty property, GUIContent label)
        {
            return EditorGUIUtility.singleLineHeight;
        }
    }

    /// <summary>
    /// 范围值类型
    /// </summary>
    [System.Serializable]
    public struct RangeValue
    {
        public float min;
        public float max;

        public RangeValue(float min, float max)
        {
            this.min = min;
            this.max = max;
        }

        public float Random => UnityEngine.Random.Range(min, max);
        public float Lerp(float t) => Mathf.Lerp(min, max, t);
    }
}
#endif
```

---

## 六、实用工具模板

### 6.1 扩展方法

```csharp
using UnityEngine;
using System.Collections.Generic;

namespace YourProject.Extensions
{
    /// <summary>
    /// Transform 扩展方法
    /// </summary>
    public static class TransformExtensions
    {
        public static void Reset(this Transform transform)
        {
            transform.localPosition = Vector3.zero;
            transform.localRotation = Quaternion.identity;
            transform.localScale = Vector3.one;
        }

        public static void DestroyAllChildren(this Transform transform)
        {
            for (int i = transform.childCount - 1; i >= 0; i--)
            {
                Object.Destroy(transform.GetChild(i).gameObject);
            }
        }

        public static T GetOrAddComponent<T>(this Transform transform) where T : Component
        {
            return transform.gameObject.GetOrAddComponent<T>();
        }
    }

    /// <summary>
    /// GameObject 扩展方法
    /// </summary>
    public static class GameObjectExtensions
    {
        public static T GetOrAddComponent<T>(this GameObject gameObject) where T : Component
        {
            var component = gameObject.GetComponent<T>();
            return component != null ? component : gameObject.AddComponent<T>();
        }

        public static void SetLayerRecursively(this GameObject gameObject, int layer)
        {
            gameObject.layer = layer;
            foreach (Transform child in gameObject.transform)
            {
                child.gameObject.SetLayerRecursively(layer);
            }
        }
    }

    /// <summary>
    /// Vector3 扩展方法
    /// </summary>
    public static class VectorExtensions
    {
        public static Vector3 WithX(this Vector3 v, float x) => new Vector3(x, v.y, v.z);
        public static Vector3 WithY(this Vector3 v, float y) => new Vector3(v.x, y, v.z);
        public static Vector3 WithZ(this Vector3 v, float z) => new Vector3(v.x, v.y, z);

        public static Vector3 Flat(this Vector3 v) => new Vector3(v.x, 0f, v.z);
        
        public static Vector2 ToVector2XZ(this Vector3 v) => new Vector2(v.x, v.z);
        public static Vector3 ToVector3XZ(this Vector2 v) => new Vector3(v.x, 0f, v.y);
    }

    /// <summary>
    /// Collection 扩展方法
    /// </summary>
    public static class CollectionExtensions
    {
        public static T GetRandom<T>(this IList<T> list)
        {
            if (list == null || list.Count == 0)
                return default;
            return list[Random.Range(0, list.Count)];
        }

        public static void Shuffle<T>(this IList<T> list)
        {
            for (int i = list.Count - 1; i > 0; i--)
            {
                int j = Random.Range(0, i + 1);
                (list[i], list[j]) = (list[j], list[i]);
            }
        }
    }
}
```

### 6.2 计时器工具

```csharp
using UnityEngine;
using System;

namespace YourProject.Utils
{
    /// <summary>
    /// 简单计时器
    /// </summary>
    public class Timer
    {
        public float Duration { get; private set; }
        public float Elapsed { get; private set; }
        public float Remaining => Mathf.Max(0f, Duration - Elapsed);
        public float Progress => Duration > 0f ? Mathf.Clamp01(Elapsed / Duration) : 1f;
        public bool IsRunning { get; private set; }
        public bool IsComplete => Elapsed >= Duration;

        public event Action OnComplete;

        public Timer(float duration)
        {
            Duration = duration;
        }

        public void Start()
        {
            Elapsed = 0f;
            IsRunning = true;
        }

        public void Stop()
        {
            IsRunning = false;
        }

        public void Reset()
        {
            Elapsed = 0f;
        }

        public void Tick(float deltaTime)
        {
            if (!IsRunning) return;

            Elapsed += deltaTime;
            
            if (Elapsed >= Duration)
            {
                IsRunning = false;
                OnComplete?.Invoke();
            }
        }
    }

    /// <summary>
    /// 冷却计时器
    /// </summary>
    public class CooldownTimer
    {
        private float _cooldownTime;
        private float _lastUseTime = float.NegativeInfinity;

        public float CooldownTime
        {
            get => _cooldownTime;
            set => _cooldownTime = Mathf.Max(0f, value);
        }

        public bool IsReady => Time.time >= _lastUseTime + _cooldownTime;
        public float RemainingTime => Mathf.Max(0f, (_lastUseTime + _cooldownTime) - Time.time);
        public float Progress => _cooldownTime > 0f ? 1f - (RemainingTime / _cooldownTime) : 1f;

        public CooldownTimer(float cooldownTime)
        {
            _cooldownTime = cooldownTime;
        }

        public bool TryUse()
        {
            if (!IsReady) return false;
            
            _lastUseTime = Time.time;
            return true;
        }

        public void Reset()
        {
            _lastUseTime = float.NegativeInfinity;
        }

        public void ForceReady()
        {
            _lastUseTime = Time.time - _cooldownTime;
        }
    }
}
```
