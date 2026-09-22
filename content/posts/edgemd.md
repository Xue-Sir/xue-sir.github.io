---
title: "EdgeMd：面向本地 Markdown 文件的 Windows 侧边工具"
date: 2026-09-18
draft: false
categories: ["Tool"]
tags: ["windows", "markdown", "desktop-app", "productivity"]
description: "介绍 EdgeMd 的设计目标、文件模型、交互功能、发布方式和 v0.1.0.0 的使用范围。"
summary: "EdgeMd 面向 Windows 本地 Markdown 文件，提供右侧驻留、任务操作、轻量编辑、外部变更监视和公开发布附件。"
---

EdgeMd 是一款 Windows 桌面工具，面向本地 Markdown 文件提供侧边查看和轻量编辑能力。

项目针对一个具体的桌面工作流：计划、任务和临时记录以 Markdown 文件保存，工作期间需要持续查看和更新，但完整编辑器窗口会占用较多工作区域。EdgeMd 将选中的 Markdown 文件固定显示在屏幕右侧，并把任务操作和简单编辑直接写回原文件。

对于文档内容，当前打开的 Markdown 文件就是唯一的持久化来源。程序不建立独立文档库，不执行网络同步，也不要求用户更换现有编辑器。项目源码、说明文档、测试和 Windows 发布附件均已公开在 [GitHub 项目](https://github.com/Xue-Sir/EdgeMd) 中。

<!--more-->

## 摘要

EdgeMd 当前面向 Windows x64，公开版本为 **v0.1.0.0**。版本发布页提供便携目录版、单文件自包含版和 SHA256SUMS.txt 校验文件。

当前实现包括：

- 将 Markdown 文件固定在屏幕右侧显示；
- 读取和写回本地 .md、.markdown 文件；
- 任务勾选、行编辑、同级项目和下级任务；
- 标题、列表、缩进、YAML frontmatter 和常见双链解析；
- 外部文件变更监视和编辑冲突提示；
- 置顶、透明度、主题、鼠标穿透和托盘控制。

Obsidian、VS Code、Typora、记事本等程序都可以创建和编辑目标文件。EdgeMd 通过文件路径工作，编辑器可以根据个人工作流选择。

## 1. 设计目标

Markdown 适合保存周计划、任务清单、会议记录和每日记录。典型文档如下：

    # 今日计划

    - [ ] 整理实验数据
      - [ ] 检查原始文件
      - [ ] 记录异常结果
    - [x] 回复邮件
    - [ ] 晚上复盘

EdgeMd 围绕以下目标实现：

- 文档持续显示在工作区侧边；
- 点击复选框即可更新任务状态；
- 双击行内容可以进行轻量编辑；
- 编辑结果写回当前 Markdown 文件；
- 其他程序保存文件后，窗口自动重新载入；
- 程序退出后，文档仍保持标准 Markdown 格式。

当前版本将文档内容保持在行级模型中，优先支持计划和清单场景。窗口显示、任务修改和文件保存围绕同一份本地文件完成。

## 2. 文件模型和编辑器关系

EdgeMd 直接使用用户选择的文件路径。文件可以位于任意本地目录，也可以由其他程序同时打开：

    任意编辑器  ──保存──▶  本地 Markdown 文件  ◀──读取/写回──  EdgeMd
                                          │
                                          └── 文件变化后自动刷新

Obsidian 适合长篇编辑、目录整理和链接管理，EdgeMd 负责将其中一份计划固定显示在桌面侧边。使用 VS Code、Typora、记事本或其他编辑器时，文件读写流程保持一致。

EdgeMd 支持以下常见双链写法：

    [[页面名]]
    [[页面名|显示文字]]

当链接目标位于当前文档目录并且扩展名为 Markdown 文件时，程序会尝试打开目标文件。该功能围绕本地文件路径工作。

## 3. Markdown 解析范围

当前版本会识别和保留以下内容：

- YAML frontmatter；
- 一级到六级标题；
- 任务项和嵌套任务；
- 普通项目符号和有序项目；
- 段落、空行和缩进；
- [[页面名]] 和 [[页面名|显示文字]] 形式的双链。

YAML frontmatter 会保留在原文件中，阅读视图中隐藏。缩进、复选框、标题和链接语法在写回时会尽量保持原有形式。

以下内容当前暂未实现完整渲染：

- 图片；
- 表格；
- 代码块；
- 复杂 Markdown 超链接；
- 第三方插件语法；
- 复杂内联格式。

这些内容仍然保留在原文件中，EdgeMd 按当前行内容进行展示和保存。完整 Markdown 编辑需求可以继续交给现有编辑器处理。

## 4. 窗口和交互

打开文件后，窗口贴靠当前显示器的工作区右侧，不覆盖任务栏区域。窗口默认保持置顶，文件名显示在顶部，文档内容显示在下面。打开文件按钮平时隐藏，鼠标移动到右下角后显示。

主要操作如下：

1. 单击任务、标题或普通文字，保持阅读状态；
2. 双击一行，在原位置开始编辑；
3. 点击复选框，更新任务状态并保存到 Markdown 文件；
4. 编辑时按 Enter 保存；
5. 按 Ctrl+Enter 保存并添加同级项目；
6. 按 Esc 取消当前编辑；
7. 右键一行，添加同级项目、添加下级任务或删除当前行；
8. 普通模式下按住 Alt 拖动窗口内容，移动窗口位置。

窗口支持鼠标穿透。启用后，鼠标操作可以落到后方程序；需要滚轮、编辑或任务操作时，可以从托盘菜单关闭，也可以点击窗口左下角的按钮切换。

## 5. 文件监视和冲突处理

EdgeMd 会持续监视当前打开的文件。

当其他编辑器保存新内容时，程序重新读取文件并刷新窗口。在 EdgeMd 中勾选任务或编辑行内容时，修改写回同一个文件。

如果编辑期间检测到其他程序同时修改了当前文件，程序会显示冲突提示，提供重新载入外部内容、保留当前编辑或暂时不处理等选项。用户确认后再决定当前修改的处理方式。

设置保存在程序运行目录下的 settings.json 中。当前程序不会修改注册表、Windows 服务或系统启动项，也不会将 Markdown 内容上传到网络。

## 6. 设置和托盘功能

设置窗口提供以下选项：

- 35% 到 100% 的窗口透明度；
- 窗口宽度、高度和边缘大小；
- 内容字号、内容字体和界面字体；
- 8 组背景、文字和强调色主题；
- 置顶、右侧贴靠、圆角或直角窗口；
- 鼠标穿透；
- 启动时打开上次使用的文件。

Windows 右下角托盘菜单可以显示或隐藏窗口、打开 Markdown 文件、进入设置、切换鼠标穿透和退出程序。

主窗口没有单独的关闭按钮。需要结束程序时，从托盘菜单选择“退出 EdgeMd”。

## 7. 使用方式

可以从 [v0.1.0.0 Release](https://github.com/Xue-Sir/EdgeMd/releases/tag/v0.1.0.0) 下载程序。

使用便携版时，解压压缩包，运行目录中的 EdgeMd.exe，再从窗口右下角的打开按钮选择 Markdown 文件。

也可以在命令行启动时直接传入文件路径：

    .\EdgeMd.exe "D:\Notes\today.md"

程序不需要另外安装 .NET Runtime。便携版的程序、设置文件和相关运行文件位于同一目录，适合放在固定位置长期使用；单文件版本适合临时复制到其他 Windows x64 电脑。

## 8. GitHub 项目和发布附件

EdgeMd 的源码、文档、测试和构建配置均位于公开仓库：

- [项目主页](https://github.com/Xue-Sir/EdgeMd)：源码、README、Issue 和项目说明；
- [用户说明书](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/USER-MANUAL.md)：首次使用和窗口操作；
- [开发文档](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/DEVELOPMENT.md)：构建、测试和发布；
- [MIT License](https://github.com/Xue-Sir/EdgeMd/blob/main/LICENSE)：许可证；
- [v0.1.0.0 Release](https://github.com/Xue-Sir/EdgeMd/releases/tag/v0.1.0.0)：版本说明和下载附件。

| 下载文件 | 说明 |
| --- | --- |
| [EdgeMd-v0.1.0.0-win-x64-portable.zip](https://github.com/Xue-Sir/EdgeMd/releases/download/v0.1.0.0/EdgeMd-v0.1.0.0-win-x64-portable.zip) | 便携目录版。解压后运行 EdgeMd.exe，适合长期放在固定目录使用。 |
| [EdgeMd-v0.1.0.0-win-x64.exe](https://github.com/Xue-Sir/EdgeMd/releases/download/v0.1.0.0/EdgeMd-v0.1.0.0-win-x64.exe) | 单文件自包含版，适合临时复制到其他 Windows x64 电脑。 |
| [SHA256SUMS.txt](https://github.com/Xue-Sir/EdgeMd/releases/download/v0.1.0.0/SHA256SUMS.txt) | 下载后核对文件完整性。 |

当前版本没有代码签名证书。如果 Windows SmartScreen 或杀毒软件提示未知发布者，可以先下载 SHA256SUMS.txt，核对压缩包或程序的 SHA-256 值，再决定是否运行。

## 9. v0.1.0.0 的范围

v0.1.0.0 优先完成了“打开真实 Markdown 文件、固定到屏幕右侧、快速修改任务”这条流程，当前范围如下：

- Windows x64；
- 便携目录版和单文件程序；
- 本地 Markdown 文件的读取、监视和写回；
- 任务勾选、行编辑、同级项目和下级任务；
- 标题、列表、缩进、frontmatter 和常见双链；
- 暂未提供安装器、ARM 版本和自动更新；
- 暂未完整渲染图片、表格、代码块和复杂 Markdown 格式；
- 暂未提供网络同步。

如果项目需要长文写作、图片、表格或复杂插件能力，可以继续使用现有 Markdown 编辑器；EdgeMd 适合计划、任务清单和每日记录等轻量文档。

## 10. 项目文档和反馈

仓库中已经放入 [需求与验收标准](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/REQUIREMENTS.md)、[产品体验审查](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/PRODUCT-REVIEW.md) 和 [安全说明](https://github.com/Xue-Sir/EdgeMd/blob/main/docs/SECURITY.md)。

如果在文件解析、编辑保存、窗口贴靠或任务操作方面遇到问题，欢迎在 [GitHub Issues](https://github.com/Xue-Sir/EdgeMd/issues) 提交反馈。

EdgeMd 适合计划、任务清单和每日记录等轻量文档，完整 Markdown 写作、复杂排版和插件工作流仍可交给专用编辑器处理。
