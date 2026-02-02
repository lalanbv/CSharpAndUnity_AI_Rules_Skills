# Qoder 配置安装指南

> Unity AI 辅助规则库的 Qoder 配置安装说明

## 目录结构说明

```
qoder-config/
├── rules/                              # Qoder 规则文件
│   ├── unity-redlines.md               # Unity 性能红线规则
│   └── unity-development.md            # Unity 开发规范规则
└── skills/                             # Qoder 技能配置
    ├── unity-developer/SKILL.md        # Unity 综合开发技能
    ├── unity-performance/SKILL.md      # 性能优化技能
    ├── unity-architecture/SKILL.md     # 架构设计技能
    ├── unity-code-reviewer/SKILL.md    # 代码审查技能
    └── unity-workflows/SKILL.md        # 开发工作流技能
```

## 安装步骤

### 方式一：手动复制 (推荐)

1. **安装规则文件**
   ```powershell
   # 复制规则到 Qoder 全局配置目录
   Copy-Item -Path ".\qoder-config\rules\*" -Destination "$env:USERPROFILE\.qoder\rules\" -Recurse -Force
   ```

2. **安装技能文件**
   ```powershell
   # 复制技能到 Qoder 全局配置目录
   Copy-Item -Path ".\qoder-config\skills\*" -Destination "$env:USERPROFILE\.qoder\skills\" -Recurse -Force
   ```

### 方式二：一键安装脚本

在 PowerShell 中运行：
```powershell
# 创建目录
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.qoder\rules"
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.qoder\skills"

# 复制文件
Copy-Item -Path ".\qoder-config\rules\*" -Destination "$env:USERPROFILE\.qoder\rules\" -Recurse -Force
Copy-Item -Path ".\qoder-config\skills\*" -Destination "$env:USERPROFILE\.qoder\skills\" -Recurse -Force

Write-Host "Qoder Unity 配置安装完成!" -ForegroundColor Green
```

## 验证安装

1. **检查规则文件**
   ```powershell
   dir "$env:USERPROFILE\.qoder\rules\"
   ```
   应显示:
   - unity-redlines.md
   - unity-development.md

2. **检查技能文件**
   ```powershell
   dir "$env:USERPROFILE\.qoder\skills\"
   ```
   应显示:
   - unity-developer
   - unity-performance
   - unity-architecture
   - unity-code-reviewer
   - unity-workflows

3. **在 Qoder 中验证**
   - 打开 Qoder
   - 输入 `/skills` 查看可用技能列表
   - 应能看到所有 unity-* 技能

## 使用方式

### 技能调用

| 场景 | 技能 | 调用方式 |
|-----|-----|---------|
| 编写 MonoBehaviour | unity-developer | 自动触发或输入关键词 |
| 性能问题排查 | unity-performance | 提及"性能"、"优化"、"GC" |
| 架构设计 | unity-architecture | 提及"架构"、"单例"、"事件系统" |
| 代码审查 | unity-code-reviewer | 提及"审查"、"review" |
| 编辑器工具 | unity-workflows | 提及"Editor"、"Inspector" |

### 规则生效

规则会在编辑 `.cs` 文件时自动生效，提供:
- P0-P3 级别的红线检查
- 代码规范提醒
- 最佳实践建议

## 卸载

```powershell
# 删除规则
Remove-Item "$env:USERPROFILE\.qoder\rules\unity-*.md" -Force

# 删除技能
Remove-Item "$env:USERPROFILE\.qoder\skills\unity-*" -Recurse -Force
```

## 更新

重新运行安装脚本即可覆盖更新。

## 常见问题

**Q: 技能没有出现在列表中？**
A: 确认 SKILL.md 文件存在于对应的技能文件夹中，且 YAML frontmatter 格式正确。

**Q: 规则没有生效？**
A: 检查规则文件的 `globs` 配置是否匹配当前编辑的文件类型。

**Q: 如何自定义规则？**
A: 直接编辑 `~/.qoder/rules/` 目录下的规则文件，修改后立即生效。
