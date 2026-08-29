# Onboarding

This walkthrough assumes you are setting up a Mac from scratch and have never used an agent operating system before. Every step says what to do and what you should see afterwards.

The short path is the guided one: open the project in Claude Code and run `/athena-setup`. Athena installs herself as a conversation, one layer at a time, shows a dry run before anything is written, and installs nothing without your yes. Below is the same road on foot.

Athena runs on macOS. You need an administrator account and network access for Homebrew, npm and Git.

1. **Get the project and read the installer before running it.**

   ```bash
   git clone https://github.com/zarubinvibe/athena.git "$HOME/athena"
   cd "$HOME/athena"
   less preinstall.sh
   ```

   You see the script you are about to run. Reading it first is the point, not a formality.

2. **Run the first step by hand.**

   ```bash
   ./preinstall.sh
   ```

   You see Xcode Command Line Tools, Homebrew, chezmoi, Node.js, Git and a pinned Claude Code CLI arrive. This step needs your Mac password: that part cannot be delegated.

3. **Open Claude Code in the repository.**

   ```bash
   claude
   ```

   You see the agent start inside the project folder.

4. **Run the guided setup.** Type `/athena-setup` and answer one question at a time. Each question says why it is asked and what a good answer looks like.

   You see your answers written down as a configuration that describes your machine, not someone else's.

5. **Look at the plan before anything is written.**

   ```bash
   ./bootstrap.sh --dry-run
   ```

   You see every layer print what it would do. Nothing is installed, cloned or overwritten during the preview.

6. **Build the layers.** Tools, dotfiles, plugins, the capability registry, your projects, an optional knowledge vault, secret storage and scheduled jobs, in that order.

   You see a workspace where agents, skills and projects are back in place.

7. **Run one layer again on purpose.**

   ```bash
   ./bootstrap.sh --only=6
   ```

   You see it finish cleanly. Layers are made to be repeatable: a second run is not a risk.

8. **Check the result.**

   ```bash
   shellcheck -S error bootstrap.sh preinstall.sh smoke/*.sh
   bash smoke/smoke.sh
   ```

   You see green checks, or a named file and line to fix.

9. **Watch the daily state.**

   ```bash
   node "$HOME/.agents/registry/scripts/athena-status.mjs" --days=7
   ```

   You see whether anything quietly failed this week. The command exits red when a job failed without a retry.

10. **Finish on your own work, not on an example.** Open a real project of yours in the rebuilt environment and do one actual task. An install that never ran the real thing is not finished.

## Keeping it current

Later, when a new version is published, do not clone it again: open the project in Claude Code and run `/athena-update`. It backs up first, shows the difference before applying it, pulls only fast-forward changes, leaves your private layer, keys, projects and vault alone, and re-runs the checks afterwards.

## If this helped

If Athena gave you your environment back, give it a star: [https://github.com/zarubinvibe/athena](https://github.com/zarubinvibe/athena). It takes a second and decides whether other people ever find the project.

You have run it end to end, which makes you the person who can improve it. The path is short: fork the repository, create a branch, commit your change, push the branch, then open a Pull Request. Do not push directly to `main`; the release gate rejects it.

Found a step that lies? Open an issue at [https://github.com/zarubinvibe/athena/issues](https://github.com/zarubinvibe/athena/issues) and say what you ran and what you saw.
