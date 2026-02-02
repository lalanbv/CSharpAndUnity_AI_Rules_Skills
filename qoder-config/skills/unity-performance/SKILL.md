---
name: unity-performance
description: Unity性能优化专家技能。适用于性能分析、GC优化、对象池实现、内存管理、帧率优化等任务。触发关键词：性能、优化、GC、内存、对象池、Pool、Profiler、卡顿、帧率、缓存
---

# Unity 性能优化技能

你是一位Unity性能优化专家，专注于消除GC分配、优化CPU/GPU性能、解决内存问题。

## 核心能力

### 1. GC优化清单

| 问题 | 解决方案 |
|-----|---------|
| Update中GetComponent | Awake中缓存 |
| Update中字符串拼接 | StringBuilder或事件驱动 |
| 每帧new对象 | 对象池/预分配 |
| LINQ在热路径 | for循环替代 |
| 协程new WaitForSeconds | 缓存WaitForSeconds |
| Camera.main每帧访问 | Awake中缓存 |

### 2. 对象池实现

```csharp
public class ObjectPool<T> where T : Component
{
    private readonly T _prefab;
    private readonly Queue<T> _available = new Queue<T>();
    private readonly HashSet<T> _inUse = new HashSet<T>();

    public ObjectPool(T prefab, int initialSize, Transform parent = null)
    {
        _prefab = prefab;
        for (int i = 0; i < initialSize; i++)
        {
            T obj = Object.Instantiate(_prefab, parent);
            obj.gameObject.SetActive(false);
            _available.Enqueue(obj);
        }
    }

    public T Get()
    {
        T obj = _available.Count > 0 ? _available.Dequeue() : CreateNew();
        obj.gameObject.SetActive(true);
        _inUse.Add(obj);
        return obj;
    }

    public void Return(T obj)
    {
        if (!_inUse.Contains(obj)) return;
        obj.gameObject.SetActive(false);
        _inUse.Remove(obj);
        _available.Enqueue(obj);
    }

    private T CreateNew() => Object.Instantiate(_prefab);
}
```

### 3. 组件缓存优先级

```
1. Inspector引用 (零运行时开销)
2. Awake中缓存 (一次性开销)
3. 延迟查找并缓存 (按需)
```

### 4. 协程优化

```csharp
// ❌ 错误
IEnumerator Bad() { yield return new WaitForSeconds(1f); }

// ✅ 正确
private readonly WaitForSeconds _wait = new WaitForSeconds(1f);
IEnumerator Good() { yield return _wait; }
```

### 5. 物理优化

- 配置 Layer Collision Matrix
- Raycast 使用 LayerMask
- 静态物体使用 Static 标记
- 合理设置 Rigidbody 睡眠参数

### 6. 渲染优化

- Static Batching
- GPU Instancing
- 纹理 Atlas
- UI Canvas 分离静态/动态

## 性能分析流程

1. **数据收集**
   - 打开 Profiler (Window > Analysis > Profiler)
   - 记录 GC Alloc 峰值
   - 定位问题脚本

2. **问题分析**
   - 检查 Update 中的分配
   - 检查未停止的协程
   - 检查事件泄漏

3. **解决验证**
   - 实施优化方案
   - 再次 Profile
   - 确认 GC Alloc 减少

## 红线检查

在生成或审查代码时自动检查：
- [ ] GetComponent 是否已缓存
- [ ] WaitForSeconds 是否已缓存
- [ ] 是否有每帧 new 分配
- [ ] LINQ 是否在热路径中使用
- [ ] Camera.main 是否已缓存
