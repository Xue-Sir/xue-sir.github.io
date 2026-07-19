---
title: "skills-manager: 一键切换你的 Claude Code 工具集"
date: 2026-07-19
draft: false
categories: ["Tool"]
tags: ["claude-code", "skills-manager", "workflow"]
description: "你是否也有这样的烦恼：装了一堆 skills、MCPs、plugins，不同项目需要不同的工具组合，每次都要手动开关？"
---

# skills-manager: 一键切换你的 Claude Code 工具集

> 你是否也有这样的烦恼：装了一堆 skills、MCPs、plugins，不同项目需要不同的工具组合，每次都要手动开关？

## 摘要

**Who is this for?** Claude Code users who install many skills, MCPs, and plugins across different projects.

**Core idea:** Organize your tools into categories and switch between them with one command — no more manual toggling.

skills-manager scans your installed tools, lets you group them into categories (e.g. research, coding), and activates only the ones you need per project. It writes project-level configs (`.mcp.json`, `settings.local.json`, `skills-manage-state.json`) so each project stays independent. It also fixes the MCP "global leak" issue by migrating configs from `~/.claude.json` to project-level control. Supports natural language input in both Chinese and English.

---

**这是写给谁的？** 在不同项目中安装了大量 skills、MCPs、plugins 的 Claude Code 用户。

**核心观点：** 把工具按分类管理，一条命令切换，不用再手动开关。

skills-manager 自动扫描已安装的工具，将它们分组为分类（如研究、编程），每个项目只激活需要的工具。切换分类时，它在项目中写入配置文件（`.mcp.json`、`settings.local.json`、`skills-manage-state.json`），各项目互不干扰。同时解决了 MCP 的「全局泄漏」问题，将配置从 `~/.claude.json` 迁移到项目级控制。支持中英文自然语言输入。

## 问题

作为 Claude Code 用户，我装了很多工具：
- 研究用的 skills
- 编程用的 MCPs
- 各种 plugins

但问题是：
- 研究项目不需要编程工具
- 编程项目不需要研究工具
- 手动管理太麻烦
- MCP 配置存在「全局泄漏」：`~/.claude.json` 中的 MCP 对所有项目生效，无法按项目控制

## 解决方案

我做了一个 Claude Code skill：**skills-manager**

它可以把工具组织成「分类」（categories），然后一键切换：

```
/skills-manager switch research    # 只有研究工具
/skills-manager switch coding      # 只有编程工具
/skills-manager switch All         # 全部开启
```

### 为什么装在用户级？

Skills 和 plugins 建议安装在用户级（全局），而不是项目级。因为项目多了之后，每个项目都装一遍会重复占用空间。安装在用户级，所有项目共享同一份工具。

### 切换分类的实现方式

切换分类时，skills-manager 不会移动 skills 和 plugins 的安装文件，而是在项目中写入三个配置文件：

- **`.mcp.json`** — 写入当前分类需要的 MCP 配置
- **`settings.local.json`** — 设置 skill 和 plugin 的 on/off 状态
- **`skills-manage-state.json`** — 记录当前分类名称、启用的功能列表、时间戳

这样每个项目只激活自己需要的工具，互不干扰。

## 特色功能

### 1. 自动扫描

大部分操作（如切换分类、添加工具、迁移 MCP）都会自动扫描用户已安装的 skills、MCPs 和 plugins，确保配置文件与实际安装状态同步。

### 2. 自然语言支持

不用记命令，直接说人话：

```
# 中文
"切换到research模式"
"把skill-a加入research"
"列出所有分类"

# 英文
"switch to research"
"add skill-a to research"
"list categories"
```

### 3. 自动 MCP 迁移

Claude Code 的 MCP 配置有个「全局泄漏」问题：任何在 `~/.claude.json` 中的 MCP 都会对所有项目生效。

skills-manager 会自动把这些 MCP 迁移到项目级控制：

```bash
/skills-manager scan
# 自动检测并迁移，还会备份
```

### 4. Skill-MCP 绑定

如果你的 skill 需要特定的 MCP，可以绑定：

```
/skills-manager bind codebase-memory --mcp codebase-memory-mcp
```

这样切换分类时，skill和 MCP 会一起切换。

### 5. 安全备份

修改 `~/.claude.json` 前总是会自动备份：

```
/skills-manager backup list    # 查看备份
```

## 与其他方案的对比

| 特性 | skills-manager | 桌面应用 | 其他 skills |
|------|---------------|----------|-------------|
| 在 Claude Code 中运行 | ✅ | ❌ | ✅ |
| 无需安装 | ✅ | ❌ | ✅ |
| 自然语言 | ✅ 中英文 | ❌ | ⚠️ 英文 |
| 自动 MCP 迁移 | ✅ | ❌ | ❌ |
| Skill-MCP 绑定 | ✅ | ❌ | ❌ |

## 如何使用

### 安装

```bash
git clone https://github.com/Xue-Sir/skills-manager.git ~/.claude/skills/skills-manager
```

### 首次使用

```
/skills-manager scan
```

### 创建分类

```
/skills-manager add-category research
/skills-manager add skill-a to research
/skills-manager add mcp-x to research
/skills-manager switch research
```

## 链接

- GitHub: https://github.com/Xue-Sir/skills-manager
- 有问题或建议？欢迎提 Issue！

---

*如果你觉得有用，欢迎 ⭐ Star 支持！*
