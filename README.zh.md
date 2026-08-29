# Athena

Athena 在一台干净的 Mac 上重建你的工作环境：规则、技能、注册表和项目都回到原位。

[English](README.md) · [Русский](README.ru.md)

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE) [![Stars](https://img.shields.io/github/stars/zarubinvibe/athena?style=flat&color=C9A87A)](https://github.com/zarubinvibe/athena/stargazers) [![Status](https://img.shields.io/badge/status-reference-brightgreen.svg)](https://github.com/zarubinvibe/athena) [![Olympuz](https://img.shields.io/badge/olympuz-family-B8D6EA.svg)](https://github.com/zarubinvibe/athena#olympuz-family)

<p align="center"><img src="docs/assets/pantheon/hero.png" alt="白色大理石的雅典娜带着猫头鹰和圆盾站在古典石柱旁，身后是一层层展开的知识平面" width="100%"></p>

<!-- owner-welcome:start -->

> 你好。我把自己的配置手动搬到新 Mac 上太多次，每次都忘掉一半：规则、技能、钩子、注册表、克隆好的项目。
>
> Athena 把这套配置变成一段程序，而不是一段记忆。如果它能在一个晚上把环境还给你，就拿去，把它变成你自己的。
>
> — Filipp Zarubin

<!-- owner-welcome:end -->

## 目录

- [这是什么](#这是什么)
- [它解决什么问题](#它解决什么问题)
- [最大的优势](#最大的优势)
- [工作流程](#工作流程)
- [快速开始](#快速开始)
- [简单对比](#简单对比)
- [简单词汇](#简单词汇)
- [安全与隐私](#安全与隐私)
- [局限](#局限)
- [点亮星标与参与](#点亮星标与参与)

<!-- beginner-readme:start -->

## 这是什么

Athena 是一套可携带的智能体环境。你花几个月调好的东西，它保存成版本化的模板和检查，然后在另一台机器上一层层重建。不是一份说明清单，而是一段可以跑第二遍的程序。

## 它解决什么问题

一套配置要长半年：配置文件、技能、钩子、注册表、克隆好的项目。换台笔记本，一个晚上全没，其中一半再也想不起来。Athena 把这套配置变成你喝咖啡的时候自己重建好的东西。

## 最大的优势

**最大的优势：** 环境由程序重建，不靠记忆。

**为什么这样更好：** 任何一层都能再跑一遍而不破坏已有的东西，`--dry-run` 会在写下第一个文件之前把整个计划给你看。

## 工作流程

Bootstrap 按顺序分层执行。可以先看计划，可以只跑一层，也可以把整条链跑完。

<!-- workflow-diagram:start -->

```text
  ┌──────┐   ┌──────┐   ┌──────┐
  │ 准备 │ ▶ │ 选择 │ ▶ │ 干跑 │
  └──────┘   └──────┘   └──────┘
      ▼
  ┌──────┐   ┌──────┐   ┌──────┐
  │ 搭建 │ ▶ │ 检查 │ ▶ │ 报告 │
  └──────┘   └──────┘   └──────┘
```

<!-- workflow-diagram:end -->

| 阶段 | 会发生什么 |
|---|---|
| 1. 准备 | 先落地基础工具：Xcode 命令行工具、Homebrew、chezmoi、Node、Git |
| 2. 选择 | 一段问答挑出集成和个人层 |
| 3. 干跑 | 写任何东西之前，先把整个计划打印出来 |
| 4. 搭建 | 工具、dotfiles、插件、注册表、项目、知识库、launchd |
| 5. 检查 | shell 语法、模板、契约、洁净渲染 |
| 6. 报告 | 本地路由状态和每周 HTML 报告 |

### 第 1 步：先把 Mac 准备好

`preinstall.sh` 装上基础工具和一个固定版本的 Claude Code 命令行，然后把仓库留在 `~/athena`。脚本可以先读再跑。

**你会得到：** 一台可以承接后续全部配置的 Mac。

### 第 2 步：选择你要的东西

在仓库里启动 Claude Code，输入 `/setup-os`。它会问你要哪些集成和个人层，并写下选中的配置。跳过的部分之后还能补。

**你会得到：** 一份描述你这台机器的配置文件，而不是别人的。

### 第 3 步：先看完整计划

`./bootstrap.sh --dry-run` 打印每一层会做什么。预览过程中不安装、不克隆、不覆盖。

**你会得到：** 一份看得懂的计划，正式执行前可以接受或修改。

### 第 4 步：把各层搭起来

各层按顺序执行：Homebrew 基线、可选工具、合并后的 chezmoi 模板、Claude Code 插件、能力注册表、项目克隆、可选知识库、密钥存储和定时任务。

**你会得到：** 一个智能体、技能和项目都归位的工作环境。

### 第 5 步：跑一遍检查

仓库工作流里用的那套检查在本地同样可用：shellcheck、smoke 套件，以及写进临时目录的洁净渲染。

**你会得到：** 全绿的检查，或者指名到文件和行号的问题。

### 第 6 步：盯住每天的状态

智能体层装好之后，状态命令会读本地记录，任务失败且没有重试时它会变红。每周命令用同一批数据生成 HTML 报告。

**你会得到：** 对“这一周有没有什么悄悄坏掉”的一个简短回答。

## 快速开始

Athena 只在 macOS 上运行。需要管理员账户，以及 Homebrew、npm 和 Git 的网络访问。

```bash
git clone https://github.com/zarubinvibe/athena.git "$HOME/athena"
cd "$HOME/athena"
less preinstall.sh
./preinstall.sh
```

在新 Mac 上赶时间？直接执行同一个受版本管理的脚本：`curl -fsSL https://raw.githubusercontent.com/zarubinvibe/athena/main/preinstall.sh | bash`。完全没有 Git？拿 [ZIP](https://github.com/zarubinvibe/athena/archive/refs/heads/main.zip) 解压。之后在这个目录里打开 Claude Code，输入 `/setup-os`。 第一次用？在 Claude Code 里打开项目并运行 `/athena-setup`：安装以对话方式一层层进行，写入之前先给你看干跑结果。

第一次做这件事？[上手引导](docs/ONBOARDING.zh.md) 会一步一步带你走完第一次运行，并写清楚每条命令之后你会看到什么。

**你会得到：** 基础工具就位，仓库在 `~/athena`，`/athena-setup` 准备好问你想要什么。

## 简单对比

| 方案 | 适合什么时候 | 你会得到 | 代价 |
|---|---|---|---|
| **Athena** | 你要的是整套智能体环境，不只是 dotfiles | 有顺序的分层、干跑、注册表、项目、检查 | 只支持 macOS |
| 手动配置 | 一台机器，只配一次 | 完全掌控，不多不少 | 一天的工作，而且无法重复 |
| 只用 dotfiles 仓库 | 你主要需要 shell 和编辑器配置 | 简单、可移植 | 不会恢复智能体技能、注册表和项目 |
| 云端开发环境 | 工作发生在容器里 | 团队环境完全一致 | 你本地的机器和本地的智能体仍然没有配置 |

## 简单词汇

| 词 | 简单解释 |
|---|---|
| Repository | 仓库：Git 保存并记录版本的项目文件夹 |
| Terminal | 终端：你输入命令的窗口 |
| Command | 命令：给电脑的一条指令 |
| Branch | 分支：不影响 `main` 的另一条修改线 |
| Pull Request | 合并请求：请别人审阅并接受你的修改 |
| Layer | 层：安装过程中的一个有顺序的步骤，可以安全地重跑 |
| Dry run | 干跑：只打印计划，不在磁盘上改任何东西 |

## 安全与隐私

- 文件访问：各层写入你的主目录，以及你自己配置里写明的路径。
- Shell 与网络：安装会运行 Homebrew、npm、Git、chezmoi 和 launchd 命令，并拉取你指定的仓库。
- 密钥：值放在 macOS 钥匙串或 `~/.secrets`，绝不放进受版本管理的清单和模板。
- 确认：`/setup-os` 会问可选部分；直接执行引导脚本则按你的配置走，不再逐条询问。
- 护栏：自带的钩子会拦住已知的危险写法，但它不是系统级沙箱。
- 遥测：Athena 自己的脚本什么都不发送。第三方工具有各自的策略。
- 恢复：`athena-update` 先备份，再展示差异，然后才应用改动。

在存有重要数据的机器上运行 Athena 之前，请先读 [SECURITY.md](SECURITY.md)。

## 局限

状态：面向 macOS 的公开参考实现。检查覆盖语法、模板、卫生和洁净渲染。

- 只支持 macOS：Homebrew 的密码提示、Xcode 的弹窗和 launchd 的行为都与平台有关。
- 在全新的 Mac 上首次安装，仍然需要人工的验收清单。
- 洁净渲染测试写进临时目录，不能代替一次真实安装。
- 私有仓库、凭据和个人知识属于你，公开克隆无法验证它们。
- 第三方工具的版本和登录方式按它们自己的节奏变化。

想更深：[完整参考](docs/DETAILS.md)、[功能说明](docs/FEATURES.en.md)、[文件系统契约](rules/structure.md)、[路线图](specs/00-roadmap.md)、[真机验收](smoke/live-acceptance.md)。

## 点亮星标与参与

觉得有用？给 Athena 点亮星标：[https://github.com/zarubinvibe/athena](https://github.com/zarubinvibe/athena)。这只要一秒，却决定别人能不能找到这个项目。

想改点什么？流程很短：先 fork 仓库，建一个分支 branch，提交 commit，推送 push，然后开一个 Pull Request。请不要直接向 `main` 推送，发布闸门会拒绝。

发现问题？到 [https://github.com/zarubinvibe/athena/issues](https://github.com/zarubinvibe/athena/issues) 开一个 issue，写清楚你运行了什么、发生了什么。

<!-- beginner-readme:end -->

<!-- pantheon-family:start -->
## Olympuz 家族

这是 [Olympuz 家族](https://github.com/zarubinvibe/athena#olympuz-family) 的公开项目之一。表格里的每一行都可以打开仓库，或者直接下载源码压缩包。

| 类型 | 名称 | 做什么 | 获取 |
|---|---|---|---|
| 项目 | Athena | 可携带的智能体操作系统：在新的 Mac 上重建 Claude 与 Codex 的工作环境。 | [仓库](https://github.com/zarubinvibe/athena) · [ZIP](https://github.com/zarubinvibe/athena/archive/refs/heads/main.zip) |
| 项目 | Helioz | 全天候的智能体工作传送带，带可验证的完成标记和按目标做出的夜间决策。 | [仓库](https://github.com/zarubinvibe/helioz) · [ZIP](https://github.com/zarubinvibe/helioz/archive/refs/heads/main.zip) |
| 项目 | Mnemazine | 本地优先的记忆系统：把原始材料变成可复用的、已核验的知识。 | [仓库](https://github.com/zarubinvibe/mnemazine) · [ZIP](https://github.com/zarubinvibe/mnemazine/archive/refs/heads/main.zip) |
| 项目 | Themis | 面向俄罗斯诉讼的多智能体助手，本地识别扫描件，五位法学家组成合议审阅。 | [仓库](https://github.com/zarubinvibe/themis) · [ZIP](https://github.com/zarubinvibe/themis/archive/refs/heads/main.zip) |
| 项目 | Zeuz | 工作流工厂：把一个想法变成带规则、闸门、可观测性和回放的多智能体系统。 | [仓库](https://github.com/zarubinvibe/zeuz) · [ZIP](https://github.com/zarubinvibe/zeuz/archive/refs/heads/main.zip) |
<!-- pantheon-family:end -->

## 许可证

MIT。见 [LICENSE](LICENSE)。
