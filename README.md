# wey-skills

个人 AI Agent Skill 库。

Skills 是按需加载的工作流指令，Copilot/Claude 会根据对话内容自动识别并调用。

膜拜 https://github.com/mattpocock/skills/tree/main 并做按需迁移给自己


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
| `write-game-doc` | "生成文档" / "document this" / 写系统文档时 | 生成游戏系统的 HTML + Markdown 文档（Unity / UE 通用，含配置项、UI 资源、代码入口） |

## 第三方 Skill（项目内按需安装，不纳入存档）

> 通过各自的 CLI 工具安装到项目 `.github/prompts/` 目录，已加入 `.gitignore`。

| Skill | 安装命令 | 用途 |
|-------|---------|------|
| [`ui-ux-pro-max`](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | `uipro init --ai copilot` | UI/UX 设计智能：67 种风格、161 色板、57 字体搭配、99 条 UX 准则，支持 10+ 技术栈 |

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
├── write-a-skill/SKILL.md
└── write-game-doc/
    ├── SKILL.md
    └── template.html
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

## 感兴趣但是未试用

https://github.com/remotion-dev/skills