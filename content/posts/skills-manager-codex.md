---
title: "skills-manager for Codex CLI：按项目管理 Skills、MCP 与功能分类"
date: 2026-07-25
draft: false
categories: ["Tool"]
tags: ["codex", "skills-manager", "mcp", "workflow"]
description: "面向 Codex CLI：把 skills 和 MCP 统一接管、组合、分类，再按项目启用；保留项目自有功能，并用状态账本保证切换可逆。"
summary: "把 Codex CLI 的 Skills 与 MCP 统一接管、组合和分类，再为每个项目启用最小且可逆的功能集合。"
---

当 `~/.codex/skills` 里的 skill 越来越多，`~/.codex/config.toml` 里也积累了多个 MCP，真正麻烦的往往不是安装，而是管理：

- 哪些功能应该长期保留在用户级？
- 一个研究项目和一个代码项目，为什么要加载同一组工具？
- 某个 skill 与 MCP 本来就是一套能力，为什么还要分开开关？
- 切换工具时，怎样避免覆盖项目原有配置？

我为此写了 Codex CLI 版本的 **skills-manager**。它把 skill、MCP，以及由 skills-manager 自己定义的 plugin 统一称为“功能”，先集中接管，再按项目选择。

本文对应 **OpenAI Codex CLI** 的目录发现和项目配置行为，不保证 Codex Desktop 或其他界面采用相同机制。Claude Code CLI 版本位于同一仓库的 `skills-manager-claude-code/`。

<!--more-->

## 摘要

**Who is this for?** Codex CLI users who maintain multiple skills and MCP servers across projects with different tool requirements.

**Core idea:** Adopt those capabilities into one private catalog outside `~/.codex/skills`, group related skills and MCPs, and switch each project to a minimal category without overwriting project-owned tools.

The package at `~/.codex/skills/skills-manager` contains only the manager itself. Managed skills, MCP definitions, plugins, configuration, retirement data, and backups live under `~/.codex/skills-manager-data`. This boundary is essential because Codex recursively discovers `SKILL.md` files below `~/.codex/skills`; storing the catalog inside the package would expose every managed skill globally before a category is selected.

skills-manager discovers user-level functions, records unique ownership, and lets the AI recommend or edit reusable categories through natural language. When a project switches categories, selected skills are linked into `.codex/skills/`, selected MCP definitions are merged into `.codex/config.toml`, and `.codex/skills-manager-state.toml` records what the manager created versus what the project already owned. This makes later switches reversible and keeps unrelated project functions intact.

---

**这是写给谁的？** 在多个项目之间维护不同 Skills 与 MCP 组合的 Codex CLI 用户。

**核心观点：** 先把功能接管到统一的用户级目录，再按任务组合为分类；项目切换分类时，只添加所需功能，并保留项目原有配置。

skills-manager 支持自然语言推荐、分类和切换。它用链接共享 skill，把选中的 MCP 合并到项目配置，并通过项目状态账本区分“由 manager 创建的条目”和“项目本来就有的条目”，从而让切换保持可逆。

## 一句话理解

skills-manager 维护一个用户级功能库。你可以把若干功能归入分类，再让某个项目使用该分类：

```text
$skills-manager 创建一个 research 分类，加入论文写作、Zotero 和开发者文档功能
$skills-manager 分析当前项目还需要哪些功能
$skills-manager 当前项目使用 research
```

用户不需要记住 Python 命令。AI 会把自然语言翻译成确定的操作，先展示预览，说明将移动、写入或移除什么，得到确认后才执行。

这个功能库不放在 skill 包里面。`~/.codex/skills/skills-manager` 只保存 manager 自身；所有运行数据统一位于 `~/.codex/skills-manager-data`。这是功能隔离能够成立的前提：Codex 会递归发现 `~/.codex/skills` 下的 `SKILL.md`，如果把被管理的 skill 或含有 `SKILL.md` 的备份放进 manager 包，它们仍会被全局加载，分类就失去了意义。

因此 skills-manager 包本身是无状态、可复制的：把它复制到另一台电脑不会携带原电脑的功能或配置；首次初始化会在新电脑创建独立的 `skills-manager-data`。以后功能的增删改、更新、退役、恢复和备份也全部发生在数据目录，而不是程序包中。

## 三种功能

### Skill

独立 skill 存放在 `~/.codex/skills-manager-data/code_skills/active/` 中。项目启用它时，不复制整份内容，而是在：

```text
<project>/.codex/skills/<skill>
```

创建指向用户级功能库的 junction 或 symlink。多个项目因此可以共享同一份 skill。

### MCP

独立 MCP 的定义集中保存在：

```text
~/.codex/skills-manager-data/code_mcp/mcps.toml
```

它使用标准的 `[mcp_servers.<id>]` 格式。项目切换分类时，skills-manager 只把选中的 MCP 合并进：

```text
<project>/.codex/config.toml
```

### Plugin

这里的 plugin 不是 Codex 原生 plugin，而是 skills-manager 定义的功能包。一个 plugin 可以包含多个 skills 和 MCP：

```text
~/.codex/skills-manager-data/code_plugin/<plugin>/
  <skill-a>/
  <skill-b>/
  mcps.toml
```

例如可以把一个文档查询 skill 和它依赖的 MCP 组合为同一个 plugin。分类只选择整个 plugin；落到项目中时，它仍会展开为实际的 skill 链接和 MCP 配置。

同一个 skill 或 MCP 只能有一个所有者：要么独立存在，要么属于一个 plugin，不能在多个注册表里重复。

## 分类不是复制，而是一份选择清单

分类包含三类条目：

- 独立 skills
- 独立 MCPs
- 完整 plugins

内置分类有两个：

- `All`：全部已接管功能的无重复并集
- `None`：不启用任何由 skills-manager 管理的功能，也是初始默认分类

你可以继续创建 `research`、`coding`、`office` 等自定义分类。同一个用户级功能库可以服务多个项目，每个项目只记录自己的选择。

## 切换项目时到底改了什么

skills-manager 只管理三个项目级位置：

```text
<project>/.codex/skills/
<project>/.codex/config.toml
<project>/.codex/skills-manager-state.toml
```

其中 `skills-manager-state.toml` 是一份所有权账本，区分：

- 当前分类实际选择了哪些 skills 和 MCP；
- 哪些条目是 skills-manager 创建的；
- 哪些条目本来就在项目里，只是恰好与当前选择一致。

这个区别很重要。项目已有的同目标 skill 链接或相同 MCP 会被“借用”，而不是被 skills-manager 占有。以后切换到别的分类时，它们仍然保留。

如果项目中存在同名但不同内容的 skill/MCP，或者先前由 skills-manager 创建的条目已被项目手动修改，切换会停止并报告冲突，不会直接覆盖。

## 新功能如何进入管理

skills-manager 会检查：

- 用户级 skills：`~/.codex/skills`
- 用户级 MCP：`~/.codex/config.toml`
- `~/.codex/skills-manager-data` 中的存储、注册表与指纹是否一致

发现新的外部功能后，它只会列为候选，不会擅自迁移。用户确认后：

- skill 被移入 `~/.codex/skills-manager-data/code_skills/active/`；
- MCP 被移入 `~/.codex/skills-manager-data/code_mcp/mcps.toml`；
- 用户级 `config.toml` 中对应的 MCP 表被移除；
- `All`、能力描述、指纹和扫描状态同步更新。

如果 MCP 的用途无法从可靠信息确认，skills-manager 会直接询问，而不是根据名字或启动命令猜一个描述。

## 让 AI 帮你选择分类

`capabilities.json` 保存每个功能的摘要和标签。你可以直接描述任务：

```text
$skills-manager 我准备维护一个 Python 项目，需要查库文档、跑浏览器测试和审查代码，
请给出最小功能集合
```

AI 会结合项目文件、功能摘要和 plugin 边界，区分必需功能、可选功能和用途尚未确认的 MCP。程序给出的关键词分数只负责缩小候选范围，最终选择仍由 AI 做语义判断，并由用户确认。

## 安全策略

所有修改操作都遵循同一流程：

1. 扫描并校验当前状态；
2. 运行不带 `--apply` 的预览；
3. 说明物理移动、配置写入、分类重写和冲突；
4. 用户确认后执行；
5. 再次校验，并做对应的扫描或项目切换检查。

修改已有的用户级或项目级 `config.toml` 前会创建备份。一次 MCP 批量迁移只备份一次，不会按 MCP 数量重复备份。

备份目录在不超过 500 MB 时只增不删。超过 500 MB 后，也只会预览约 30 天以前的顶层备份；得到用户明确确认后才允许清理。最近一个月的备份不会为了压低容量而被删除。

## 配置为什么要拆开

skills-manager 没有把所有信息塞进一个大文件。以下配置全部位于 `~/.codex/skills-manager-data/config/`，不会在 skill 包中保留第二份：

| 配置 | 职责 |
|---|---|
| `skills.json` / `mcps.json` | 注册独立功能，不复制实际定义 |
| `plugins.json` | 记录 plugin 的唯一所有权和成员 |
| `categories.json` | 保存 `All`、`None` 和自定义分类 |
| `capabilities.json` | 保存可搜索的摘要、标签和 MCP 描述状态 |
| `scan-state.json` | 保存已接受的指纹，用于发现漂移 |
| `updates.json` | 保存 Git、手动、无更新或运行时托管等更新策略 |
| `config/retired/` | 保存可恢复的退休记录 |

定义、所有权、搜索信息和项目选择各有单一职责，避免 skill/MCP/plugin 在多个文件里重复登记后逐渐失去一致性。

## 安装

仓库同时提供 Codex 与 Claude Code 两个独立版本。Codex 版本位于 `skills-manager-codex/`。

PowerShell：

```powershell
git clone --depth 1 --filter=blob:none --sparse https://github.com/Xue-Sir/skills-manager.git skills-manager-codex-source
git -C skills-manager-codex-source sparse-checkout set skills-manager-codex
New-Item -ItemType Directory -Force "$HOME\.codex\skills" | Out-Null
Copy-Item -Recurse ".\skills-manager-codex-source\skills-manager-codex" "$HOME\.codex\skills\skills-manager"
python -m pip install -r "$HOME\.codex\skills\skills-manager\requirements.txt"
```

Bash：

```bash
git clone --depth 1 --filter=blob:none --sparse https://github.com/Xue-Sir/skills-manager.git skills-manager-codex-source
git -C skills-manager-codex-source sparse-checkout set skills-manager-codex
mkdir -p ~/.codex/skills
cp -R skills-manager-codex-source/skills-manager-codex ~/.codex/skills/skills-manager
python -m pip install -r ~/.codex/skills/skills-manager/requirements.txt
```

Codex 发布包不包含运行时 `config`、`code_*`、`retired` 或 `backups` 目录，也不包含作者自己的 skills、MCP、密钥或用户配置。首次初始化时，程序直接在 `~/.codex/skills-manager-data` 生成所需结构。

## 第一次使用

先用自然语言初始化私有数据目录：

```text
$skills-manager 初始化私有数据目录
```

确认目标为 `~/.codex/skills-manager-data` 后，再扫描外部功能：

```text
$skills-manager 扫描当前用户的 skills 和 MCP，列出尚未接管的功能
```

确认迁移对象和 MCP 描述后，再让它完成接管、检查更新策略并创建第一个分类：

```text
$skills-manager 创建一个适合当前项目的分类，并在确认后切换
```

如果项目暂时不需要任何托管功能：

```text
$skills-manager 当前项目切换到 None
```

这只会移除状态账本中确认由 skills-manager 创建的条目，不影响项目自己的额外 skill 和 MCP。

## 链接

- GitHub: https://github.com/Xue-Sir/skills-manager
- Codex 实现：`skills-manager-codex/`
- Claude Code 实现：`skills-manager-claude-code/`

如果你也在多个项目之间维护不同的工具组合，欢迎试用并提交 Issue。
