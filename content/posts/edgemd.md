---
title: "我做了一个把 Markdown 文件放在桌面侧边的小工具：EdgeMd"
date: 2026-09-18
draft: false
categories: ["Tool"]
tags: ["windows", "markdown", "desktop-app", "productivity"]
description: "记录 EdgeMd 的使用场景、文件读写方式、Windows 窗口功能和 v0.1.0.0 公开下载地址。"
summary: "EdgeMd 直接打开一个本地 Markdown 文件，把它放在 Windows 屏幕右侧，支持任务勾选、简单编辑和外部修改刷新。"
---

最近我做了一个 Windows 小工具，名字叫 **EdgeMd**。

我平时会把计划、任务和一些临时记录写在 Markdown 文件里。工作时，我经常只想看今天的任务，却要在编辑器里打开一整个文件，再在当前窗口和编辑器之间来回切换。于是我做了一个更简单的窗口：打开一份 Markdown 文件，把它固定放在屏幕右侧，需要时看一眼，完成任务后直接勾掉。

EdgeMd 直接使用本地的 .md 或 .markdown 文件。它可以读取文件、监听文件变化，也可以把任务勾选和简单编辑写回文件。项目源码、说明文档、测试和 Windows 发布附件都放在 [GitHub 项目](https://github.com/Xue-Sir/EdgeMd) 中。

<!--more-->

## 摘要

EdgeMd 目前面向 Windows x64，公开版本为 **v0.1.0.0**。下载 Release 后，解压便携版或直接运行单文件程序，就可以打开一份本地 Markdown 文档。

我把它做成了一个侧边窗口：窗口可以贴在屏幕右侧、保持置顶、调整透明度和主题；文档中的任务可以点击完成，也可以双击某一行进行编辑。其他编辑器保存文件后，EdgeMd 会跟着刷新。

Obsidian、VS Code、Typora、记事本都可以用来创建和编辑这份 Markdown 文件。EdgeMd 只需要文件本身，编辑器可以按照自己的习惯选择。

## 1. 我为什么做 EdgeMd

我需要的场景很具体：电脑屏幕旁边一直放着一份计划或任务清单。

这份清单可能是下面这样的普通 Markdown 文档：

    # 今日计划

    - [ ] 整理实验数据
      - [ ] 检查原始文件
      - [ ] 记录异常结果
    - [x] 回复邮件
    - [ ] 晚上复盘

我希望它满足几件事：

- 打开后一直在屏幕边上，工作时可以随时看到；
- 点击复选框就能完成任务；
- 任务修改后保存回原来的 Markdown 文件；
- 用其他程序编辑文件后，侧边窗口能及时更新；
- 关闭程序后，文件仍然可以用普通编辑器继续打开。

EdgeMd 目前就围绕这几个需求展开，重点放在计划、任务和清单这类内容上。

## 2. EdgeMd 怎样使用 Markdown 文件

EdgeMd 处理的是文件本身。文件放在哪里、由哪个编辑器创建、是否放在某个笔记目录里，都不会影响它打开和保存。

它的工作关系可以简单看成这样：

    任意编辑器  ──保存──▶  本地 Markdown 文件  ◀──读取/写回──  EdgeMd
                                          │
                                          └── 文件变化后自动刷新

我自己也会用 Obsidian 管理一些 Markdown 文档，所以测试时经常让 Obsidian 和 EdgeMd 同时打开同一份文件。Obsidian 负责整理和长篇编辑，EdgeMd 负责把计划放在桌面旁边。换成 VS Code、Typora、记事本或其他编辑器，使用方式也一样。

EdgeMd 还支持常见的双链写法：

    [[页面名]]
    [[页面名|显示文字]]

如果链接目标是当前文件目录下的 Markdown 文件，点击后会尝试打开它。这项功能只针对本地 Markdown 文件，使用时不需要额外安装某个笔记软件。

## 3. 下载后怎么打开

可以从 [v0.1.0.0 Release](https://github.com/Xue-Sir/EdgeMd/releases/tag/v0.1.0.0) 下载。

使用便携版时，解压压缩包，运行目录中的 **EdgeMd.exe**，再从窗口右下角的打开按钮选择 Markdown 文件。

也可以在命令行启动时直接传入文件路径：

    .\EdgeMd.exe "D:\Notes\today.md"

打开文件后，窗口会贴靠当前显示器的工作区右侧，不占用任务栏区域。窗口默认保持置顶，设置中可以关闭。文件名显示在顶部，文档内容显示在下面；打开文件按钮平时隐藏，把鼠标移到右下角就会出现。

日常操作如下：

1. 单击任务、标题或普通文字，保持阅读状态；
2. 双击一行，在原位置开始编辑；
3. 点击复选框，把任务状态写回 Markdown 文件；
4. 编辑时按 Enter 保存，按 Ctrl+Enter 保存并添加同级项目，按 Esc 取消；
5. 右键一行，可以编辑、添加同级项目、添加下级任务或删除当前行；
6. 普通模式下按住 Alt 拖动窗口内容，可以移动窗口。

窗口支持鼠标穿透。打开后，鼠标操作可以直接落到后面的程序；需要滚轮、编辑或勾选任务时，可以从托盘菜单关闭穿透，也可以点击窗口左下角的小按钮切换。

## 4. 当前支持的 Markdown 内容

EdgeMd 现在会识别这些内容：

- YAML frontmatter；
- 一级到六级标题；
- 任务项和嵌套任务；
- 普通项目符号和有序项目；
- 段落、空行和缩进；
- [[页面名]] 和 [[页面名|显示文字]] 形式的双链。

YAML frontmatter 会保留在文件中，阅读时隐藏。缩进、复选框、标题和链接语法也会尽量按原样保存。

我目前把显示和编辑做得比较轻量，图片、表格、代码块、复杂 Markdown 超链接以及第三方插件语法暂时不会按完整 Markdown 效果渲染。这些内容仍然会留在原文件里，EdgeMd 会按照当前的行内容展示。

## 5. 外部修改和保存

EdgeMd 会持续观察当前打开的文件。

例如，我用另一个编辑器添加一项任务并保存，EdgeMd 会重新读取文件并刷新窗口；在 EdgeMd 中点击复选框或编辑一行，修改也会保存回同一个文件。

如果编辑期间检测到其他程序同时改过这个文件，程序会弹出提示，让我选择重新载入外部内容、保留当前编辑，或者暂时不处理。这样可以在两边同时编辑时先确认一次，再决定保留哪一份内容。

设置保存在程序运行目录下的 settings.json 中。当前程序不会修改注册表、Windows 服务或系统启动项，也不会把 Markdown 内容上传到网络。

## 6. 窗口和托盘设置

设置窗口提供了这些选项：

- 透明度从 35% 到 100% 调整；
- 窗口宽度、高度和边缘大小；
- 内容字号、内容字体和界面字体；
- 8 组背景、文字和强调色主题；
- 置顶、右侧贴靠、圆角或直角窗口；
- 鼠标穿透；
- 启动时打开上次使用的文件。

Windows 右下角托盘菜单可以显示或隐藏窗口、打开 Markdown 文件、进入设置、切换鼠标穿透和退出程序。

主窗口没有单独的关闭按钮。需要退出时，从托盘菜单选择“退出 EdgeMd”。

## 7. GitHub 项目和下载附件

EdgeMd 的源码和发布文件都在 GitHub：

- [项目主页](https://github.com/Xue-Sir/EdgeMd)：源码、README、Issue 和项目说明；
- [用户说明书](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/USER-MANUAL.md)：首次使用和窗口操作；
- [开发文档](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/DEVELOPMENT.md)：构建、测试和发布；
- [MIT License](https://github.com/Xue-Sir/EdgeMd/blob/main/LICENSE)：许可证；
- [v0.1.0.0 Release](https://github.com/Xue-Sir/EdgeMd/releases/tag/v0.1.0.0)：版本说明和下载附件。

| 下载文件 | 说明 |
| --- | --- |
| [EdgeMd-v0.1.0.0-win-x64-portable.zip](https://github.com/Xue-Sir/EdgeMd/releases/download/v0.1.0.0/EdgeMd-v0.1.0.0-win-x64-portable.zip) | 便携目录版。解压后运行 EdgeMd.exe，适合长期放在固定目录使用。 |
| [EdgeMd-v0.1.0.0-win-x64.exe](https://github.com/Xue-Sir/EdgeMd/releases/download/v0.1.0.0/EdgeMd-v0.1.0.0-win-x64.exe) | 单文件自包含版，适合临时复制到另一台 Windows x64 电脑。 |
| [SHA256SUMS.txt](https://github.com/Xue-Sir/EdgeMd/releases/download/v0.1.0.0/SHA256SUMS.txt) | 下载后核对文件完整性。 |

当前版本没有代码签名证书。如果 Windows SmartScreen 或杀毒软件提示未知发布者，可以先下载 SHA256SUMS.txt，核对压缩包或程序的 SHA-256 值，再决定是否运行。程序不需要另外安装 .NET Runtime。

## 8. v0.1.0.0 的范围

这一版我先把“打开真实 Markdown 文件、放在屏幕右侧、快速修改任务”这条流程做完整，当前范围如下：

- 发布 Windows x64 版本；
- 提供便携目录版和单文件程序；
- 支持本地 Markdown 文件的读取、监视和写回；
- 支持任务勾选、行编辑、同级项目和下级任务；
- 支持标题、列表、缩进、frontmatter 和常见双链；
- 暂未提供安装器、ARM 版本和自动更新；
- 暂未完整渲染图片、表格、代码块和复杂 Markdown 格式；
- 暂未提供网络同步。

如果主要需求是写长文、插入图片、编辑表格或使用复杂插件，仍然可以继续用自己熟悉的 Markdown 编辑器；EdgeMd 适合把一份计划、任务清单或每日记录放在旁边查看。

## 9. 项目文档和反馈

仓库中已经放入 [需求与验收标准](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/REQUIREMENTS.md)、[产品体验审查](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/PRODUCT-REVIEW.md) 和 [安全说明](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/SECURITY.md)。

如果你在文件解析、编辑保存、窗口贴靠或任务操作方面遇到问题，欢迎直接在 [GitHub Issues](https://github.com/Xue-Sir/EdgeMd/issues) 提交反馈。

我会继续把 EdgeMd 当作一个小而实用的桌面工具来维护：一份 Markdown 文件放在屏幕边上，随时能看，改完也能回到原来的编辑器继续工作。
