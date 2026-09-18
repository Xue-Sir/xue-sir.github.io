---
title: "EdgeMd：把一个 Markdown 文档固定在桌面侧边的 Windows 小工具"
date: 2026-09-18
draft: false
categories: ["Tool"]
tags: ["windows", "markdown", "desktop-app", "productivity"]
description: "介绍 EdgeMd：一个直接打开本地 Markdown 文档、持续显示在屏幕右侧并支持任务勾选和轻量编辑的 Windows 小工具。"
summary: "EdgeMd 不依赖 Obsidian，也不是新的笔记库；它把一个普通的 .md 或 .markdown 文件变成可持续查看和修改的桌面侧边便签。"
---

最近做了一个 Windows 桌面小工具：**EdgeMd**。它的核心不是管理某个笔记软件，而是打开一个普通的 Markdown 文档，把内容固定显示在屏幕侧边。

只要是本地的 .md 或 .markdown 文件，就可以交给 EdgeMd。这个文件可以由 Obsidian、VS Code、记事本、Typora 或其他编辑器创建和修改。Obsidian 只是其中一种使用方式，不是 EdgeMd 的前提，也不需要安装 Obsidian 才能运行。

项目源代码、文档、测试、构建工作流和 Windows 发布附件都公开放在 [GitHub 仓库](https://github.com/Xue-Sir/EdgeMd) 中。EdgeMd 不复制一份笔记，也不建立自己的数据库，它始终围绕用户正在打开的那一个 Markdown 文件工作。

<!--more-->

## 摘要

**Who is this for?** Windows users who keep plans, tasks, notes, or checklists in ordinary Markdown files and want one file to stay visible beside their main work.

**Core idea:** EdgeMd is a lightweight side-panel viewer and editor for a local .md or .markdown file. It reads and writes the original file, watches for external changes, and keeps the workflow independent from any particular note-taking application.

The first public release is **v0.1.0.0** for Windows x64. The GitHub Release provides a portable directory package, a self-contained single-file executable, and SHA256SUMS.txt for integrity checks. No separate .NET Runtime installation is required.

---

**这是写给谁的？** 使用普通 Markdown 文件记录计划、任务、清单或日记，希望把其中一份文档长期放在工作区旁边的 Windows 用户。

**核心观点：** EdgeMd 是一个本地 Markdown 文件的侧边阅读和轻量编辑工具。它直接读取和写回原文件，监听其他程序的修改，不依赖某一个笔记软件，也不把内容复制到自己的数据库里。

当前公开版本是 **v0.1.0.0**，目标平台为 Windows x64。GitHub Release 提供便携目录版、单文件自包含版和 SHA256SUMS.txt 校验文件，不需要另外安装 .NET Runtime。

## 1. EdgeMd 解决的是什么问题

Markdown 很适合保存简单而长期有效的内容，例如周计划、今日任务、会议清单、读书记录和实验安排：

    # 今日计划

    - [ ] 整理实验数据
      - [ ] 检查原始文件
      - [ ] 记录异常结果
    - [x] 回复邮件
    - [ ] 晚上复盘

这样的文件不需要数据库，也不需要特定的云服务。它可以放在任意本地目录，用自己习惯的编辑器打开。

但在实际工作时，完整编辑器往往太大。为了看几项任务，需要一直占着一个窗口，或者在编辑器和当前工作之间反复切换。EdgeMd 做的事情很窄：把这一个 Markdown 文件变成一块贴在桌面右侧的持续可见内容。

它更像一个“文件视图”，而不是新的笔记应用：

- 文件仍然由用户自己保存和管理；
- EdgeMd 直接读取原始 .md 文件；
- 勾选任务和编辑行会写回原文件；
- 其他程序修改文件后，EdgeMd 会自动刷新；
- 关闭 EdgeMd 后，文件仍然是普通 Markdown，不会留下专用格式。

## 2. 不依赖 Obsidian，编辑器只是文件的另一个入口

EdgeMd 使用的是 Markdown 文件本身，不是 Obsidian 仓库、插件 API 或某个同步服务。

因此可以这样理解它们之间的关系：

    任意编辑器  ──保存──▶  本地 Markdown 文件  ◀──读取/写回──  EdgeMd
                                          │
                                          └── 外部修改后自动刷新

如果你使用 Obsidian，可以让 Obsidian 负责长文编辑、整理链接和管理文件，再让 EdgeMd 显示其中一份周计划。如果你不用 Obsidian，也完全不影响 EdgeMd：用记事本写一个 .md 文件，或者用 VS Code、Typora 等编辑后，都可以直接打开给 EdgeMd。

EdgeMd 额外支持 [[页面名]]、[[页面名|显示文字]] 这种常见的双链写法，但这只是显示和打开链接的功能，不代表软件依赖 Obsidian。目标 Markdown 文件只要是一个普通本地文件，就可以使用 EdgeMd。

## 3. 打开一个 Markdown 文件

最简单的方式是下载 GitHub Release 中的便携目录版，解压后运行 EdgeMd.exe，再从窗口右下角的打开按钮选择一个本地 Markdown 文件。

也可以在启动时直接传入文件路径：

    .\EdgeMd.exe "D:\Notes\today.md"

文件打开后，窗口会贴靠当前显示器的工作区右侧，不覆盖任务栏。窗口默认置顶，但可以在设置中关闭。文件名显示在顶部，Markdown 内容显示在下面；打开文件按钮平时隐藏，把鼠标移到右下角才会出现。

日常操作保持得比较直接：

1. 单击任务、标题或普通行，保持阅读状态；
2. 双击一行，在原位置进入编辑；
3. 点击复选框，把 - [ ] 和 - [x] 写回原 Markdown 文件；
4. 编辑时按 Enter 保存，按 Ctrl+Enter 保存并添加同级项目，按 Esc 取消；
5. 右键一行，可以编辑、添加同级项目、添加下级任务或删除当前行；
6. 普通模式下按住 Alt 拖动任意区域，可以移动窗口。

如果打开了鼠标穿透，窗口会尽量不拦截后方应用的操作。需要滚轮、编辑或勾选任务时，可以从托盘菜单关闭，也可以点击主窗口左下角的小按钮切换。

## 4. 它能处理哪些 Markdown 内容

EdgeMd 针对“计划和清单”这类简单 Markdown 做了轻量解析，而不是试图实现完整的 Markdown 排版引擎。

目前会识别并保留：

- YAML frontmatter；
- 一级到六级标题；
- 任务项和任务层级；
- 普通项目符号和有序项目；
- 段落、空行和缩进；
- [[页面名]] 和 [[页面名|显示文字]] 形式的双链。

YAML frontmatter 在阅读视图中隐藏，但不会被删除或重排。缩进、复选框、标题和链接语法会尽量原样保留。点击双链时，如果目标是当前文档目录下的 Markdown 文件，EdgeMd 会尝试打开它；非 Markdown 的本地路径不会被当作可执行文件启动。

当前版本不会渲染图片、表格、代码块、复杂 Markdown 超链接和完整的第三方插件语法。这些内容仍然保留在原文件中，但在 EdgeMd 中会按轻量文本处理。它的重点是“看文档、改任务”，不是在侧边重新实现一个完整 Markdown 编辑器。

## 5. 文件修改和冲突处理

EdgeMd 会持续监视当前打开的文件。当外部编辑器保存了新内容，侧边窗口会自动刷新。用户在 EdgeMd 中勾选任务或完成行编辑时，修改会写回当前文件。

编辑期间如果检测到其他程序也修改了当前文件，EdgeMd 会弹出冲突提示，让用户选择重新载入外部文件、保留当前编辑，或暂时不处理。它不会在没有提示的情况下静默覆盖另一边的修改。

设置保存在程序运行目录下的 settings.json 中。程序不写注册表，不创建 Windows 服务，不修改系统启动项，也不会联网同步 Markdown 内容。

## 6. 窗口和托盘设置

设置窗口提供了几类常用选项：

- 透明度，可在 35% 到 100% 之间连续调整；
- 窗口宽度、高度和窗口边缘调整；
- 内容字号、内容字体和界面字体；
- 8 组背景、文字和强调色统一的主题；
- 置顶、右侧贴靠、圆角或直角窗口；
- 鼠标穿透和启动时打开上次文件。

Windows 右下角托盘菜单可以显示或隐藏窗口、打开 Markdown、进入设置、切换鼠标穿透和退出程序。主窗口没有单独的关闭按钮，选择托盘菜单中的“退出 EdgeMd”才会真正退出。

## 7. GitHub 项目与公开下载

EdgeMd 是一个公开的 GitHub 项目，不只是一个放了二进制文件的下载页：

- [项目主页](https://github.com/Xue-Sir/EdgeMd)：查看源码、README、Issue 和项目文档；
- [用户说明书](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/USER-MANUAL.md)：了解首次使用和窗口操作；
- [开发文档](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/DEVELOPMENT.md)：了解构建、测试和发布方式；
- [MIT License](https://github.com/Xue-Sir/EdgeMd/blob/main/LICENSE)：项目许可证；
- [v0.1.0.0 Release](https://github.com/Xue-Sir/EdgeMd/releases/tag/v0.1.0.0)：下载附件和查看版本记录。

| 发布附件 | 适合场景 |
| --- | --- |
| [EdgeMd-v0.1.0.0-win-x64-portable.zip](https://github.com/Xue-Sir/EdgeMd/releases/download/v0.1.0.0/EdgeMd-v0.1.0.0-win-x64-portable.zip) | 推荐长期使用。解压后运行 EdgeMd.exe，设置文件和程序放在同一目录。 |
| [EdgeMd-v0.1.0.0-win-x64.exe](https://github.com/Xue-Sir/EdgeMd/releases/download/v0.1.0.0/EdgeMd-v0.1.0.0-win-x64.exe) | 单文件自包含版，适合临时复制到另一台 Windows x64 电脑。首次启动和杀毒软件扫描可能更慢。 |
| [SHA256SUMS.txt](https://github.com/Xue-Sir/EdgeMd/releases/download/v0.1.0.0/SHA256SUMS.txt) | 用于下载后校验附件完整性。 |

下载后如果 Windows SmartScreen 或杀毒软件提示未知发布者，这是因为当前版本还没有代码签名证书。可以先用 SHA256SUMS.txt 校验文件，再决定是否运行。软件不需要另外安装 .NET Runtime。

## 8. 当前版本的边界

v0.1.0.0 先把“右侧显示真实 Markdown、快速改任务、和其他编辑器共用文件”这条路径做完整，暂时保留以下限制：

- 目前只发布 Windows x64 版本；
- 没有安装器、ARM 版本和自动更新；
- 不渲染图片、表格、代码块和复杂 Markdown 内联格式；
- 不提供网络同步，也不包含完整的第三方插件语法支持；
- 当前以轻量行模型处理内容，不追求完整 Markdown 编辑体验。

如果你的主要需求是写长文、插入图片、编辑表格或使用复杂插件，仍然应该使用自己习惯的 Markdown 编辑器。EdgeMd 更适合放一份周计划、任务清单、每日记录或其他简单 Markdown 文档在旁边持续查看。

## 9. 项目文档与反馈

项目仓库中已经放入 [需求与验收标准](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/REQUIREMENTS.md)、[产品体验审查](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/PRODUCT-REVIEW.md) 和 [安全说明](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/SECURITY.md)。如果你发现文件解析、编辑保存或窗口操作方面的问题，可以直接在 GitHub 提交 Issue。

EdgeMd 的边界很明确：它不接管笔记，也不要求用户更换编辑器，只是在桌面侧边为一个普通 Markdown 文档留出一块一直可见的位置。
