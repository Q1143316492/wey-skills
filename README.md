# wey-skills

个人 AI Agent Skill 库，按来源分类整理。

## 📂 目录结构

```
wey-skills/
├── mattpocock/      # 7个 - 简洁快速的个人向 skills
├── superpowers/     # 14个 - 严谨完整的团队流程 skills  
└── custom/          # 3个 - 个人开发定制 skills
```

## 📚 来源

- **[mattpocock/skills](https://github.com/mattpocock/skills)** - 轻量级工作流，适合快速任务和个人项目
- **[obra/superpowers](https://github.com/obra/superpowers)** - 完整开发流程，适合团队协作和大型项目

## 🚀 使用方式

### 方式1：选择性拷贝（推荐）
```powershell
# 只拷贝需要的分类到项目
Copy-Item -Recurse "D:\Unity\Src\wey-skills\mattpocock" "<project>\.github\skills\"
Copy-Item -Recurse "D:\Unity\Src\wey-skills\custom" "<project>\.github\skills\"
```

### 方式2：Junction 全量链接
```powershell
# 链接整个仓库
cmd /c mklink /J "<project>\.github\skills" "D:\Unity\Src\wey-skills"
```

## 📋 快速参考

### mattpocock/ - 快速任务
- `grill-me` - 反复追问明确需求
- `diagnose` - 快速调试循环
- `tdd` - 轻量级红绿循环
- `caveman` - 超压缩沟通省token
- `handoff` - 会话交接文档
- `zoom-out` - 全局代码地图
- `write-a-skill` - 快速创建skill

### superpowers/ - 完整流程
- `brainstorming` → `writing-plans` → `subagent-driven-development` - 完整开发流程
- `test-driven-development` / `systematic-debugging` - 严格TDD和调试
- `requesting-code-review` / `receiving-code-review` - Code Review流程
- `using-git-worktrees` / `finishing-a-development-branch` - Git工作流
- `verification-before-completion` - 完成前验证
- `dispatching-parallel-agents` / `executing-plans` - 任务编排

### custom/ - Unity定制
- `write-game-doc` - 生成游戏系统文档（HTML + Markdown）
- `skill-evaluator` - Skill评估测试框架
- `testcase` - 测试用例示例

## ⚖️ 功能重复说明

| 功能 | mattpocock | superpowers | 最终使用 |
|------|-----------|-------------|---------|
| TDD | `tdd` | `test-driven-development` | ✅ superpowers |
| 调试 | `diagnose` | `systematic-debugging` | ✅ superpowers |
| 写Skill | `write-a-skill` | `writing-skills` | ✅ superpowers |

## 📦 生产环境最终列表

```
mattpocock/caveman
mattpocock/grill-me
mattpocock/handoff
mattpocock/zoom-out
superpowers/brainstorming
superpowers/dispatching-parallel-agents
superpowers/executing-plans
superpowers/finishing-a-development-branch
superpowers/receiving-code-review
superpowers/requesting-code-review
superpowers/subagent-driven-development
superpowers/systematic-debugging
superpowers/test-driven-development
superpowers/using-git-worktrees
superpowers/using-superpowers
superpowers/verification-before-completion
superpowers/writing-plans
superpowers/writing-skills
custom/skill-evaluator
custom/write-game-doc
```
