---
name: unity-workflows
description: Unity开发工作流技能。适用于编辑器工具开发、自定义Inspector、Input System配置、UI系统开发等任务。触发关键词：编辑器、Editor、EditorWindow、CustomEditor、Inspector、PropertyDrawer、Input System、UI、Canvas
---

# Unity 开发工作流技能

你是一位Unity工具开发专家，精通编辑器扩展和工作流优化。

## 核心能力

### 1. EditorWindow模板

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;

public class CustomEditorWindow : EditorWindow
{
    private Vector2 _scrollPosition;
    private string _searchText = "";

    [MenuItem("Tools/Custom Window")]
    private static void ShowWindow()
    {
        var window = GetWindow<CustomEditorWindow>();
        window.titleContent = new GUIContent("Custom Window");
        window.minSize = new Vector2(400, 300);
        window.Show();
    }

    private void OnGUI()
    {
        DrawToolbar();
        DrawMainContent();
    }

    private void DrawToolbar()
    {
        EditorGUILayout.BeginHorizontal(EditorStyles.toolbar);
        
        if (GUILayout.Button("Refresh", EditorStyles.toolbarButton, GUILayout.Width(60)))
            Refresh();

        GUILayout.FlexibleSpace();
        
        _searchText = EditorGUILayout.TextField(_searchText, 
            EditorStyles.toolbarSearchField, GUILayout.Width(200));

        EditorGUILayout.EndHorizontal();
    }

    private void DrawMainContent()
    {
        _scrollPosition = EditorGUILayout.BeginScrollView(_scrollPosition);
        // 内容绘制
        EditorGUILayout.EndScrollView();
    }

    private void Refresh() => Repaint();
}
#endif
```

### 2. CustomEditor模板

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;

[CustomEditor(typeof(TargetScript))]
[CanEditMultipleObjects]
public class TargetScriptEditor : Editor
{
    private SerializedProperty _propHealth;
    private bool _showAdvanced = false;

    private void OnEnable()
    {
        _propHealth = serializedObject.FindProperty("_health");
    }

    public override void OnInspectorGUI()
    {
        serializedObject.Update();

        EditorGUILayout.PropertyField(_propHealth);
        
        _showAdvanced = EditorGUILayout.Foldout(_showAdvanced, "Advanced", true);
        if (_showAdvanced)
        {
            EditorGUI.indentLevel++;
            // 高级设置
            EditorGUI.indentLevel--;
        }

        if (GUILayout.Button("Execute", GUILayout.Height(25)))
            ExecuteAction();

        serializedObject.ApplyModifiedProperties();
    }

    private void OnSceneGUI()
    {
        var script = (TargetScript)target;
        Handles.color = Color.yellow;
        Handles.DrawWireDisc(script.transform.position, Vector3.up, 2f);
    }

    private void ExecuteAction()
    {
        foreach (var t in targets)
        {
            var script = t as TargetScript;
            if (script != null) Debug.Log($"Executing on {script.name}");
        }
    }
}
#endif
```

### 3. PropertyDrawer模板

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;

[CustomPropertyDrawer(typeof(RangeValue))]
public class RangeValueDrawer : PropertyDrawer
{
    public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
    {
        EditorGUI.BeginProperty(position, label, property);

        position = EditorGUI.PrefixLabel(position, label);

        var minProp = property.FindPropertyRelative("min");
        var maxProp = property.FindPropertyRelative("max");

        float halfWidth = (position.width - 4) / 2f;
        var minRect = new Rect(position.x, position.y, halfWidth, position.height);
        var maxRect = new Rect(position.x + halfWidth + 4, position.y, halfWidth, position.height);

        EditorGUI.PropertyField(minRect, minProp, GUIContent.none);
        EditorGUI.PropertyField(maxRect, maxProp, GUIContent.none);

        EditorGUI.EndProperty();
    }
}

[System.Serializable]
public struct RangeValue
{
    public float min;
    public float max;
    public float Random => UnityEngine.Random.Range(min, max);
}
#endif
```

### 4. Input System配置

```csharp
using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerInput : MonoBehaviour
{
    private PlayerInputActions _inputActions;
    
    private void Awake()
    {
        _inputActions = new PlayerInputActions();
    }
    
    private void OnEnable()
    {
        _inputActions.Player.Enable();
        _inputActions.Player.Jump.performed += OnJump;
    }
    
    private void OnDisable()
    {
        _inputActions.Player.Jump.performed -= OnJump;
        _inputActions.Player.Disable();
    }
    
    private void Update()
    {
        Vector2 move = _inputActions.Player.Move.ReadValue<Vector2>();
        // 处理移动
    }
    
    private void OnJump(InputAction.CallbackContext context)
    {
        // 处理跳跃
    }
}
```

### 5. UI开发规范

```csharp
using UnityEngine;
using UnityEngine.UI;

public class UIPanel : MonoBehaviour
{
    [SerializeField] private Button _closeButton;
    [SerializeField] private Text _titleText;
    
    private void Awake()
    {
        _closeButton.onClick.AddListener(Close);
    }
    
    private void OnDestroy()
    {
        _closeButton.onClick.RemoveListener(Close);
    }
    
    public void Show()
    {
        gameObject.SetActive(true);
    }
    
    public void Close()
    {
        gameObject.SetActive(false);
    }
}
```

## 编辑器代码规范

1. **必须放在Editor文件夹**
   - Assets/Scripts/Editor/

2. **必须条件编译**
   ```csharp
   #if UNITY_EDITOR
   // 编辑器代码
   #endif
   ```

3. **支持Undo**
   ```csharp
   Undo.RecordObject(target, "Change Description");
   // 修改操作
   ```

4. **标记Dirty**
   ```csharp
   EditorUtility.SetDirty(target);
   ```
