---
name: write-game-doc
description: Generate HTML + Markdown documentation for a game system (Unity or Unreal), covering code, client assets, config tables, and persistent data. Use when user wants to document a game system, write system docs, or create a navigation index for game code/prefabs/blueprints/configs.
---

# Write Game System Doc

Supports both **Unity** and **Unreal Engine**. Auto-detect: `*.uproject` + `Source/` → UE; `Assets/` + `ProjectSettings/` → Unity. Ask if ambiguous.

## Document Philosophy

This document is a **code navigation index**, not a reference manual.

- 目标读者：新人扫一眼就知道"系统由哪几块组成、每块在哪里"，然后 Ctrl-Click 跳进源码自学
- 一个系统 = **代码** + **客户端资源** + **配置** + **持久化数据**（可有可无）
- 写"哪里有什么"，**不**写"它怎么实现"
- 宁缺毋滥：没有的部分整节删掉，不要留空架子
- 所有类名 / 路径 / 表名 / 字段名必须用 `<code>` 包裹，方便 IDE 全局搜索定位

## Process

1. **Scan** the codebase for the named system:

   | | Unity | Unreal |
   |---|---|---|
   | Entry code | `MonoBehaviour` / static manager / SO | `UObject` / `AActor` / `UGameInstanceSubsystem` / Blueprint class |
   | Config | `ScriptableObject`, CSV/Excel (via custom loader), `[SerializeField]` | `UDataAsset`, `UDataTable` (CSV/Excel imported via editor), `UDeveloperSettings`, `UPROPERTY(EditAnywhere)` |
   | UI asset | Prefab via `Resources.Load` / Addressables / direct ref | `UUserWidget` Blueprint (`WBP_*.uasset`), referenced in `.cpp` or other BP |
   | Persistence | `PlayerPrefs` / `JsonUtility` save file / SQLite / 服务端协议 | `USaveGame` / `SaveSlot` / SQLite / 服务端 RPC |

2. **Ask** — at most 3 questions for things code alone can't tell you (scope boundary, where designer-facing configs live, non-obvious gotchas, whether DB lives client- or server-side).

3. **Generate two files** side by side:
   - `[SystemName].html` — copy `template.html` from this skill folder; replace all `<!-- ... -->` comments and placeholder text with real content; **delete unused sections / rows entirely**; update `<title>` and all instances of `SYSTEM NAME`; keep `<span class="step-num">N</span><span class="step-text">...</span>` structure in flow list items
   - `[SystemName].md` — same sections in plain GFM Markdown

## Document Sections

Five sections, **any of them removable**. Order is fixed.

| # | Section | What goes here |
|---|---|---|
| 1 | 系统概述 | 1–2 句话：做什么、谁用、所属模块 |
| 2 | 代码入口 | 入口类路径 + 主流程 3–5 步 + 关键类表（UE 标注 C++ / BP / C+++BP） |
| 3 | 配置项 | 见下方"配置项写法" |
| 4 | 客户端资源 | 见下方"资源写法" |
| 5 | 持久化数据 | 见下方"数据写法"。**没有就整节删** |

### 配置项写法（重点）

按"表 / 资产"分组。每组结构：

- **表名 + 路径** — `BuffTable.csv` @ `Assets/Config/Battle/`
- **读取类** — 哪个类负责加载这张表
- **行含义** — 一行代表什么（一个 buff？一个角色？）
- **字段表**：
  - **业务字段**（策划常改的）：列名 + 含义 + 取值范围 + 改了影响什么
  - **程序字段**（id / version / 内部 ref）：只列名，一句"内部使用"带过即可
- **改动场景示例**（可选，但推荐）：
  > 想新增一个 buff → 在 `BuffTable.csv` 末尾加一行；必填：`id` `name` `duration` `effectType`

ScriptableObject / DataAsset 同理：资产路径 + 字段表，业务字段详写、程序字段一笔带过。

### 资源写法

每个 Prefab / Widget 一行起步：**路径 → 主脚本 / 拥有类 → 一句话功能**。

当资源**触发事件、监听信号、跨系统引用**时，再加一行关联：

> `BtnBuy` (Prefab) → 触发 `OnPurchaseClicked` → `ShopManager.HandlePurchase`

纯展示资源（图标、背景）保持一行即可，不展开。

### 数据写法

- 存储介质：`PlayerPrefs` / `SaveGame` / SQLite / 服务端表
- 表名 / Key → 字段 → **谁写**（哪个类的哪个方法）/ **谁读**
- 不要贴完整 schema；只要"哪里存、谁动它"

## Writing Rules

- 用 `<div class="warn">⚠️ ...</div>`（HTML）或 `> ⚠️`（Markdown）标注非显而易见的注意点：GC、replication、tick 成本、资源循环引用、跨线程、存档版本兼容等
- UE 关键类表必须标 **C++** / **BP** / **C++ + BP child**
- 不引用外部 URL；不要留 `<!-- TODO: screenshot -->` 之类永远不会填的占位（用户主动提供截图路径才引用）
- 同步维护 HTML 与 MD：结构一致、内容一致、格式不同
- **不要**修改 `template.html` 本身——它是共享样式基线
- 删原则：宁缺毋滥
  - 整节没内容 → 删整节
  - 表里某行没内容 → 删那行
  - 不要为了"看起来完整"硬填

## Output Location

两份文件放在被记录代码旁边（入口类同目录，或同级 `Docs/` 文件夹）。不清楚时问用户。
