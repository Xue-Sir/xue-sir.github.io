---
title: "EdgeMd：给 Obsidian 侧边放一块持续同步的 Markdown 便签"
date: 2026-09-18
draft: false
categories: ["Tool"]
tags: ["windows", "markdown", "obsidian", "desktop-app"]
description: "介绍我做的 Windows Markdown 侧边便签 EdgeMd：直接读取并监视 Obsidian 笔记，支持任务勾选、原位编辑、双链打开和托盘控制。"
summary: "EdgeMd 是一个面向 Windows 的轻量 Markdown 侧边便签，让 Obsidian 继续负责完整编辑，同时把周计划和任务清单固定显示在屏幕右侧。"
---

我平时用 Obsidian 记录周计划、科研任务和日记，但完整编辑器并不适合一直占在屏幕旁边。做实验、写代码或查资料时，我更希望把当天要做的事情固定放在右侧，随时能看到，完成一项就直接勾掉。

所以我做了一个 Windows 小工具：**EdgeMd**。它不准备替代 Obsidian，也不复制一份笔记，而是直接读取真实的 Markdown 文件，把它显示成一块贴在屏幕右侧的便签。Obsidian 修改文件后，EdgeMd 会自动刷新；在 EdgeMd 中勾选任务或编辑文字，也会写回原文件。

<!--more-->

## 摘要

**Who is this for?** Windows users who keep plans and tasks in Obsidian or plain Markdown, but want a small always-visible side panel instead of a full editor window.

**Core idea:** Keep Obsidian as the complete editor and use EdgeMd as a lightweight reader and task surface. EdgeMd watches the real `.md` file, writes small edits back to that file, and does not create a database or synchronize notes over the network.

The first public release is **v0.1.0.0** for Windows x64. It includes a portable directory package, a self-contained single-file executable, and a `SHA256SUMS.txt` file for integrity checks. No separate .NET Runtime installation is required.

---

**这是写给谁的？** 使用 Obsidian 或普通 Markdown 管理计划、任务和日记，但希望把一小块内容长期放在屏幕边上的 Windows 用户。

**核心观点：** Obsidian 继续负责完整编辑，EdgeMd 负责持续显示和轻量修改。它读取真实的 `.md` 文件，监听外部变化，把任务勾选和行编辑直接写回原文件，不引入数据库，也不联网同步笔记。

当前公开版本是 **v0.1.0.0**，目标平台为 Windows x64。发布附件包含便携目录版、单文件自包含版和 `SHA256SUMS.txt` 校验文件，不需要另外安装 .NET Runtime。

## 1. 为什么需要一块 Markdown 便签

完整编辑器适合写作和整理资料，但不一定适合做长期显示的任务面板。把 Obsidian 窗口缩小后，编辑区、侧栏和各种按钮仍然会占用空间；如果只想看今天的几项任务，又需要在应用之间来回切换。

EdgeMd 的目标很简单：

- 把一个 Markdown 文件固定在屏幕右侧；
- 让任务清单在工作时始终可见；
- 勾选任务时直接修改原文件；
- Obsidian 在后台修改后，侧边内容自动跟上；
- 需要时可以在原位置快速改一行文字。

它更像一个“文件视图”，而不是新的笔记库。文件仍然属于用户，Obsidian 仍然是完整编辑器，EdgeMd 只提供一个更适合持续查看的窗口。

## 2. EdgeMd 的使用方式

最直接的用法是从 GitHub Release 下载便携目录版，解压后运行 `EdgeMd.exe`，再从窗口右下角打开一个 `.md` 文件。也可以把文件路径作为启动参数：

```powershell
.\EdgeMd.exe "D:\Obsidian\笔记库\2026年09月第1周.md"
```

打开文件后，窗口会贴靠当前显示器的工作区右侧，不覆盖任务栏。窗口默认置顶，但可以在设置中关闭。文件名显示在顶部，Markdown 内容显示在下面；打开文件按钮平时隐藏，把鼠标移到右下角才会出现。

日常操作保持得比较直接：

1. 单击任务、标题或普通行，保持阅读状态；
2. 双击一行，在原位置进入编辑；
3. 点击复选框，把 `- [ ]` 和 `- [x]` 写回 Markdown 文件；
4. 编辑时按 Enter 保存，按 `Ctrl+Enter` 保存并添加同级项目，按 Esc 取消；
5. 右键一行，可以编辑、添加同级项目、添加下级任务或删除当前行；
6. 普通模式下按住 `Alt` 拖动任意区域，可以移动窗口。

如果打开了鼠标穿透，窗口会尽量不拦截后方应用的操作。需要滚轮、编辑或勾选任务时，可以从托盘菜单关闭，也可以点击主窗口左下角的小按钮切换。

## 3. 它能读懂哪些 Markdown

EdgeMd 针对我实际使用的周计划格式做了轻量解析，而不是试图实现完整的 Markdown 排版引擎。

目前会识别并保留：

- YAML frontmatter；
- 一级到六级标题；
- 任务项和任务层级；
- 普通项目符号和有序项目；
- 段落、空行和缩进；
- `[[页面名]]` 和 `[[页面名|显示文字]]` 形式的 Obsidian 双链。

YAML frontmatter 在阅读视图中隐藏，但不会被删除或重排。缩进、复选框、标题和双链语法会尽量原样保留。点击双链时，如果目标是当前笔记目录下的 Markdown 文件，EdgeMd 会尝试打开它；非 Markdown 的本地路径不会被当作可执行文件启动。

这也意味着它有明确边界：当前版本不会渲染图片、表格、代码块、复杂 Markdown 超链接和完整的 Obsidian 插件语法。这些内容仍然保留在原文件中，但在 EdgeMd 中会按轻量文本处理。它的重点是“看计划、改任务”，不是“在侧边重做一个 Obsidian”。

## 4. 和 Obsidian 如何一起工作

EdgeMd 与 Obsidian 之间没有专用同步协议，二者共享同一个文件：

```text
Obsidian  ──编辑──▶ 真实的 Markdown 文件  ◀──读取/写回──  EdgeMd
                                  │
                                  └── 外部修改监听，自动刷新
```

当 Obsidian 保存文件时，EdgeMd 通过文件变化监听重新加载内容。当用户在 EdgeMd 中勾选任务或完成行编辑时，程序把修改写回当前文件。这样可以避免两份内容逐渐分叉，也不需要额外的数据库或导入导出步骤。

编辑期间如果检测到外部程序也修改了当前文件，EdgeMd 会弹出冲突提示，让用户选择重新载入外部文件、保留当前编辑，或暂时不处理。它不会在不提示的情况下静默覆盖另一边的修改。

设置保存在程序运行目录下的 `settings.json` 中。程序不写注册表，不创建 Windows 服务，不修改系统启动项，也不会联网同步笔记。

## 5. 窗口和托盘设置

设置窗口提供了几类常用选项：

- 透明度，可在 35% 到 100% 之间连续调整；
- 窗口宽度、高度和窗口边缘调整；
- 内容字号、内容字体和界面字体；
- 8 组背景、文字和强调色统一的主题；
- 置顶、右侧贴靠、圆角或直角窗口；
- 鼠标穿透和启动时打开上次文件。

Windows 右下角托盘菜单可以显示或隐藏窗口、打开 Markdown、进入设置、切换鼠标穿透和退出程序。主窗口没有单独的关闭按钮，选择托盘菜单中的“退出 EdgeMd”才会真正退出。

## 6. 下载公开版本

项目主页：<https://github.com/Xue-Sir/EdgeMd>

版本发布页：<https://github.com/Xue-Sir/EdgeMd/releases/tag/v0.1.0.0>

| 附件 | 适合场景 |
| --- | --- |
| [`EdgeMd-v0.1.0.0-win-x64-portable.zip`](https://github.com/Xue-Sir/EdgeMd/releases/download/v0.1.0.0/EdgeMd-v0.1.0.0-win-x64-portable.zip) | 推荐长期使用。解压后运行 `EdgeMd.exe`，设置文件和程序放在同一目录。 |
| [`EdgeMd-v0.1.0.0-win-x64.exe`](https://github.com/Xue-Sir/EdgeMd/releases/download/v0.1.0.0/EdgeMd-v0.1.0.0-win-x64.exe) | 单文件自包含版，适合临时复制到另一台 Windows x64 电脑。首次启动和杀毒软件扫描可能更慢。 |
| [`SHA256SUMS.txt`](https://github.com/Xue-Sir/EdgeMd/releases/download/v0.1.0.0/SHA256SUMS.txt) | 用于下载后校验附件完整性。 |

下载后如果 Windows SmartScreen 或杀毒软件提示未知发布者，这是因为当前版本还没有购买代码签名证书。可以先用 `SHA256SUMS.txt` 校验文件，再决定是否运行；后续如果长期发布，我会再考虑签名和安装器问题。

## 7. 当前版本的边界

v0.1.0.0 先把“右侧显示真实 Markdown、快速改任务、和 Obsidian 共用文件”这条路径做完整，暂时保留以下限制：

- 目前只发布 Windows x64 版本；
- 没有安装器、ARM 版本和自动更新；
- 不渲染图片、表格、代码块和复杂 Markdown 内联格式；
- 不提供网络同步，也不包含完整的 Obsidian 插件语法支持；
- 当前以轻量行模型处理内容，不追求完整 Markdown 编辑体验。

如果你的主要需求是写长文、插入图片、编辑表格或使用 Obsidian 插件，仍然应该回到 Obsidian。EdgeMd 更适合放一份周计划、任务清单或每日记录在旁边持续查看。

## 8. 项目文档与后续

GitHub 仓库中已经放入 [用户说明书](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/USER-MANUAL.md)、[开发文档](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/DEVELOPMENT.md)、[需求与验收标准](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/REQUIREMENTS.md) 和 [安全说明](https://github.com/Xue-Sir/EdgeMd/blob/main/SECURITY.md)。项目使用 MIT License，欢迎通过 Issue 反馈实际使用中的问题。

这个版本最重要的不是功能数量，而是把文件边界和使用边界说清楚：EdgeMd 不接管笔记，也不替用户决定数据放在哪里。它只是在 Obsidian 和工作区之间留出一条窄窄的、一直可见的视线。

如果你也习惯用 Markdown 管理周计划，欢迎试试 [EdgeMd](https://github.com/Xue-Sir/EdgeMd)。
