# 上手引导

这份引导假设你在从零配置一台 Mac，而且从来没用过智能体操作系统。每一步都写清楚要做什么，以及之后应该看到什么。

最短的路是对话式的：在 Claude Code 里打开项目并运行 `/athena-setup`。雅典娜会自己完成安装，一层一层来，写入之前先给你看干跑结果，没有你的同意不装任何东西。下面是同一条路，靠双脚走。

Athena 只在 macOS 上运行。你需要管理员账户，以及 Homebrew、npm 和 Git 的网络访问。

1. **拿到项目，并在运行前先读安装脚本。**

   ```bash
   git clone https://github.com/zarubinvibe/athena.git "$HOME/athena"
   cd "$HOME/athena"
   less preinstall.sh
   ```

   你会看到即将运行的脚本。先读它是这一步的意义，不是形式。

2. **第一步用手动完成。**

   ```bash
   ./preinstall.sh
   ```

   你会看到 Xcode 命令行工具、Homebrew、chezmoi、Node.js、Git 和固定版本的 Claude Code 装好。这一步需要你的 Mac 密码，这部分无法代劳。

3. **在仓库里打开 Claude Code。**

   ```bash
   claude
   ```

   你会看到智能体在项目目录里启动。

4. **运行带引导的安装。** 输入 `/athena-setup`，一次回答一个问题。每个问题都说明为什么问，以及好的回答长什么样。

   你会看到答案被写成描述你这台机器的配置，而不是别人的。

5. **在写入任何东西之前先看计划。**

   ```bash
   ./bootstrap.sh --dry-run
   ```

   你会看到每一层会做什么。预览期间不安装、不克隆、不覆盖。

6. **搭建各层。** 工具、配置文件、插件、能力注册表、你的项目、可选的知识库、密钥存储和定时任务，按这个顺序。

   你会看到智能体、技能和项目重新归位的工作环境。

7. **故意再跑一遍某一层。**

   ```bash
   ./bootstrap.sh --only=6
   ```

   你会看到它安静地又跑完一次。各层是可重复的：第二次运行不是风险。

8. **检查结果。**

   ```bash
   shellcheck -S error bootstrap.sh preinstall.sh smoke/*.sh
   bash smoke/smoke.sh
   ```

   你会看到全绿的检查，或者指名到文件和行号的问题。

9. **关注日常状态。**

   ```bash
   node "$HOME/.agents/registry/scripts/athena-status.mjs" --days=7
   ```

   你会看到这一周有没有什么悄悄失败。任务失败且没有重试时，这条命令会变红。

10. **在你自己的真实工作上收尾，而不是在样例上。** 在重建好的环境里打开你自己的项目，做一件真实的事。从没跑过真实任务的安装，不算完成。

## 以后怎么更新

以后有了新版本，不用重新克隆：在 Claude Code 里打开项目并运行 `/athena-update`。它先备份，应用之前先给你看差异，只做快进式更新，不动你的私有层、密钥、项目和知识库，更新之后重新跑检查。

## 如果这份引导帮到了你

如果 Athena 把你的环境还给了你，请点亮星标：[https://github.com/zarubinvibe/athena](https://github.com/zarubinvibe/athena)。这只要一秒，却决定别人能不能找到这个项目。

你已经从头到尾走过一遍，所以你正是能改进它的人。路径很短：先 fork 仓库，建一个分支 branch，提交 commit，推送 push，然后开一个 Pull Request。请不要直接向 `main` 推送，发布闸门会拒绝。

发现某一步写错了？到 [https://github.com/zarubinvibe/athena/issues](https://github.com/zarubinvibe/athena/issues) 开一个 issue，写清楚你运行了什么、看到了什么。
