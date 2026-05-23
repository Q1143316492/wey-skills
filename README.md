# wey-skills

个人 AI Agent Skill 库。反 vibe coding，注重人在 AI 时代的主导价值。

Skills 是按需加载的工作流指令，Copilot/Claude 会根据对话内容自动识别并调用。

## Skills

| Skill | 触发方式 | 用途 |
|-------|---------|------|
| `grill-me` | "grill me" / 开始新功能前 | 让 AI 反复追问，逼你想清楚需求，再开始做 |
| `diagnose` | "diagnose" / "debug this" / 报告 bug | 结构化调试：复现→假设→验证→修复→回归 |
| `zoom-out` | "zoom out" / 看到陌生代码时 | 先看全局模块地图，再动代码 |
| `handoff` | "handoff" / 换 session 前 | 把当前对话压缩成交接文档，供下次继续 |
| `caveman` | "caveman" / "less tokens" | 超压缩沟通模式，省 ~75% token |
| `tdd` | "tdd" / "red-green-refactor" | 测试驱动开发，逐步红绿循环 |
| `write-a-skill` | "write a skill" / 新建 skill 时 | 帮你创建新 skill 的 meta skill |

## 目录结构

```
wey-skills/
├── README.md
├── grill-me/SKILL.md
├── diagnose/SKILL.md
├── zoom-out/SKILL.md
├── handoff/SKILL.md
├── caveman/SKILL.md
├── tdd/SKILL.md
└── write-a-skill/SKILL.md
```

## 在项目中使用

本目录通过 Junction 链接到各项目：

```
<project>/.github/skills/  →  D:\Unity\Src\wey-skills\
```

新项目接入：
```powershell
cmd /c mklink /J "<project>\.github\skills" "D:\Unity\Src\wey-skills"
```
