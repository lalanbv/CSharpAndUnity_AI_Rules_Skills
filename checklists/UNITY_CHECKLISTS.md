# Unity 开发检查清单

> 用于代码审查、发布前检查和日常开发的实用清单

---

## 一、代码提交检查清单

### 1.1 每次提交前 (Pre-Commit)

#### 性能检查
- [ ] 所有 `GetComponent` 调用已在 `Awake`/`Start` 中缓存
- [ ] 没有在 `Update`/`FixedUpdate`/`LateUpdate` 中调用 `Find` 方法
- [ ] 没有在热路径中进行字符串拼接
- [ ] 没有在热路径中使用 `new` 分配对象
- [ ] 协程中的 `WaitForSeconds` 已缓存
- [ ] `Camera.main` 已缓存
- [ ] 没有空的 `Update`/`FixedUpdate`/`LateUpdate` 方法

#### 生命周期检查
- [ ] `Awake` 中只初始化自身组件和状态
- [ ] `Start` 中引用其他对象
- [ ] 所有 `OnEnable` 中的事件订阅都有对应的 `OnDisable` 取消订阅
- [ ] 协程在 `OnDisable` 或 `OnDestroy` 中停止
- [ ] 物理操作在 `FixedUpdate` 中
- [ ] 输入检测在 `Update` 中

#### 序列化检查
- [ ] 使用 `[SerializeField] private` 而非 `public` 字段
- [ ] 复杂字段有 `[Header]` 和 `[Tooltip]` 特性
- [ ] 数值字段有合理的 `[Range]` 或 `[Min]` 限制
- [ ] 没有期望属性被序列化

#### 代码质量
- [ ] 所有类都在命名空间中
- [ ] 命名遵循约定 (PascalCase/camelCase)
- [ ] 没有魔法数字 (使用 const 或配置)
- [ ] 复杂逻辑有注释说明

#### 编辑器代码
- [ ] 编辑器脚本在 `Editor` 文件夹中
- [ ] 编辑器代码使用 `#if UNITY_EDITOR` 包裹
- [ ] `OnValidate` 等编辑器回调有条件编译

---

### 1.2 Pull Request 检查清单

#### 功能完整性
- [ ] 功能按需求完整实现
- [ ] 边界情况已处理
- [ ] 错误处理完善

#### 代码审查
- [ ] 通过所有 Pre-Commit 检查
- [ ] 没有引入新的技术债务
- [ ] 代码可读性良好
- [ ] 没有重复代码

#### 测试
- [ ] 在编辑器中测试通过
- [ ] 在目标平台测试通过 (如适用)
- [ ] 性能在可接受范围内

#### 文档
- [ ] 公共 API 有 XML 注释
- [ ] 复杂系统有使用说明
- [ ] 配置项有说明

---

## 二、性能优化检查清单

### 2.1 CPU 优化

#### Update 循环
- [ ] 移除不必要的 Update 方法
- [ ] 使用 `enabled = false` 禁用不活跃的组件
- [ ] 非每帧逻辑使用协程或 InvokeRepeating
- [ ] 批量处理分散到多帧

#### 组件查找
- [ ] 所有组件引用已缓存
- [ ] 使用 `TryGetComponent` 而非 `GetComponent + null`
- [ ] 频繁碰撞检测使用组件缓存字典
- [ ] 使用 `CompareTag` 而非 `tag ==`

#### 物理
- [ ] 配置 Layer Collision Matrix
- [ ] Raycast 使用 LayerMask
- [ ] 静态物体使用 Static 标记
- [ ] Rigidbody 睡眠设置合理

### 2.2 内存优化

#### GC 分配
- [ ] 字符串操作使用 StringBuilder
- [ ] 集合预分配大小
- [ ] 避免 LINQ 在热路径
- [ ] 使用 struct 而非 class (适当时)
- [ ] 避免装箱拆箱

#### 对象池
- [ ] 高频创建的对象使用对象池
- [ ] 池对象正确重置状态
- [ ] 池有合理的初始大小

#### 资源管理
- [ ] 动态创建的 Material 在 OnDestroy 中销毁
- [ ] 大资源使用 Addressables 异步加载
- [ ] 不再使用的资源及时卸载

### 2.3 渲染优化

#### Draw Call
- [ ] 使用 Static Batching
- [ ] 使用 GPU Instancing
- [ ] 纹理使用 Atlas
- [ ] UI 元素分离静态/动态 Canvas

#### 其他
- [ ] 配置合理的 LOD
- [ ] 使用 Occlusion Culling
- [ ] 关闭不必要的阴影
- [ ] 移动端降低纹理分辨率

---

## 三、发布前检查清单

### 3.1 功能测试

- [ ] 所有功能正常工作
- [ ] 所有 UI 交互正常
- [ ] 所有输入响应正常
- [ ] 音频正常播放
- [ ] 存档/读档功能正常
- [ ] 网络功能正常 (如适用)

### 3.2 性能测试

- [ ] 目标平台帧率稳定
- [ ] 没有明显卡顿
- [ ] 内存使用在合理范围
- [ ] 启动时间可接受
- [ ] 加载时间可接受

### 3.3 兼容性测试

- [ ] 所有目标平台测试通过
- [ ] 不同分辨率测试通过
- [ ] 不同屏幕比例测试通过
- [ ] 低端设备测试通过

### 3.4 构建检查

- [ ] 构建无错误无警告
- [ ] 构建大小在预期范围
- [ ] 必要的平台 SDK 配置正确
- [ ] 签名/证书配置正确
- [ ] 版本号更新

### 3.5 资源检查

- [ ] 没有未使用的资源
- [ ] 资源大小优化
- [ ] 纹理压缩格式正确
- [ ] 音频压缩格式正确

### 3.6 安全检查

- [ ] 没有敏感信息在代码中
- [ ] 日志级别设置正确 (Release 关闭 Debug 日志)
- [ ] 反作弊措施到位 (如需要)
- [ ] 数据验证完善

---

## 四、新功能开发检查清单

### 4.1 开发前

- [ ] 需求明确理解
- [ ] 技术方案评审
- [ ] 依赖项识别
- [ ] 工作量评估
- [ ] 风险识别

### 4.2 开发中

- [ ] 遵循代码规范
- [ ] 及时提交小改动
- [ ] 保持与主分支同步
- [ ] 记录技术决策
- [ ] 及时沟通阻塞问题

### 4.3 开发后

- [ ] 代码自审
- [ ] 单元测试 (如适用)
- [ ] 集成测试
- [ ] 性能测试
- [ ] 文档更新

---

## 五、Bug 修复检查清单

### 5.1 定位阶段

- [ ] 能稳定复现问题
- [ ] 确定问题范围
- [ ] 查看相关日志
- [ ] 定位到问题代码

### 5.2 修复阶段

- [ ] 理解问题根本原因
- [ ] 修复不引入新问题
- [ ] 修复范围最小化
- [ ] 相关代码一并检查

### 5.3 验证阶段

- [ ] 问题不再复现
- [ ] 相关功能正常
- [ ] 没有引入新 Bug
- [ ] 性能没有下降

---

## 六、MonoBehaviour 检查清单

### 6.1 新建 MonoBehaviour 时

```
□ 添加命名空间
□ 添加类注释
□ 评估是否需要 [RequireComponent]
□ 确定需要哪些生命周期方法
□ 规划序列化字段
□ 规划公共接口
```

### 6.2 生命周期方法检查

```
Awake:
□ 只缓存自身组件
□ 只初始化内部状态
□ 不引用其他 GameObject

OnEnable:
□ 订阅所有需要的事件
□ 启动需要的协程
□ 重置临时状态

Start:
□ 引用其他对象
□ 注册到管理器
□ 执行依赖外部的初始化

Update:
□ 检查是否真的需要
□ 是否可以使用事件驱动替代
□ 是否可以降低频率

FixedUpdate:
□ 只做物理相关操作
□ 不检测输入

LateUpdate:
□ 只做跟随/调整逻辑
□ 不做物理操作

OnDisable:
□ 取消所有 OnEnable 中的订阅
□ 停止协程
□ 清理临时状态

OnDestroy:
□ 从管理器注销
□ 销毁动态创建的资源
□ 最终清理
```

---

## 七、ScriptableObject 检查清单

### 7.1 新建 ScriptableObject 时

```
□ 添加 [CreateAssetMenu] 特性
□ 设置合理的 fileName 和 menuName
□ 只存储数据，不存储状态
□ 使用只读属性暴露数据
□ 考虑运行时数据与设计时数据分离
```

### 7.2 使用 ScriptableObject 时

```
□ 不在 ScriptableObject 中存储运行时状态
□ 使用 [NonSerialized] 标记运行时字段
□ OnEnable 中重置运行时状态
□ 考虑使用 ScriptableObject 事件解耦
```

---

## 八、编辑器工具检查清单

### 8.1 新建编辑器工具时

```
□ 放在 Editor 文件夹中
□ 使用 #if UNITY_EDITOR 包裹
□ 使用合适的基类 (Editor/EditorWindow/PropertyDrawer)
□ 添加 [MenuItem] 或 [CustomEditor] 等特性
□ 实现 Undo 支持
□ 添加输入验证
```

### 8.2 GUI 绘制检查

```
□ 使用 serializedObject.Update() 和 ApplyModifiedProperties()
□ 使用 SerializedProperty 而非直接访问字段
□ 使用 EditorGUI.BeginChangeCheck/EndChangeCheck
□ 使用 Undo.RecordObject 记录更改
□ 使用 EditorUtility.SetDirty 标记修改
```

---

## 九、快速参考

### 9.1 性能敏感度分级

| 级别 | 调用频率 | 优化要求 |
|------|---------|---------|
| 热路径 | 每帧多次 | 零 GC，极致优化 |
| 温路径 | 每帧 1 次 | 低 GC，注意缓存 |
| 冷路径 | 偶尔调用 | 可接受少量 GC |
| 初始化 | 只调用 1 次 | 无特殊要求 |

### 9.2 常见问题速查

| 问题现象 | 可能原因 | 检查点 |
|---------|---------|--------|
| 卡顿 | GC Spike | Profiler → GC Alloc |
| 帧率低 | CPU/GPU 瓶颈 | Profiler → CPU/GPU |
| 内存增长 | 资源泄漏 | Memory Profiler |
| 空引用 | 生命周期问题 | Awake/Start 顺序 |
| 事件不触发 | 未订阅/已取消 | OnEnable/OnDisable |

### 9.3 检查命令速记

```bash
# Profiler 检查
1. Window > Analysis > Profiler
2. 查看 GC Alloc 列
3. 查看 CPU Usage
4. 查看 Memory

# 内存检查
1. Window > Analysis > Memory Profiler
2. Take Snapshot
3. 分析 Unity Objects

# 代码搜索
Ctrl+Shift+F 搜索:
- "void Update"
- "GetComponent"
- "Find("
- "OnEnable"
- "new List"
- "new WaitForSeconds"
```

---

## 十、团队约定模板

### 10.1 提交信息格式

```
[类型] 简短描述

详细说明（可选）

相关 Issue: #123

类型包括:
- [Feature] 新功能
- [Fix] Bug 修复
- [Refactor] 重构
- [Perf] 性能优化
- [Style] 代码风格
- [Docs] 文档
- [Test] 测试
```

### 10.2 分支命名

```
feature/功能名称
fix/问题描述
refactor/重构内容
release/版本号
hotfix/紧急修复
```

### 10.3 代码审查标准

```
必须修复:
- P0 红线违反
- 明显 Bug
- 安全问题

建议修复:
- P1/P2 红线违反
- 代码风格问题
- 可读性问题

可选:
- P3 建议
- 个人偏好
```
