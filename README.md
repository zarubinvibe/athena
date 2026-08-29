# Athena

Athena rebuilds a full Claude Code and Codex workspace on a fresh Mac from versioned templates and checks.

[Русский](README.ru.md)

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE) [![Stars](https://img.shields.io/github/stars/zarubinvibe/athena?style=flat&color=C9A87A)](https://github.com/zarubinvibe/athena/stargazers) [![Status](https://img.shields.io/badge/status-reference-brightgreen.svg)](https://github.com/zarubinvibe/athena) [![Olympuz](https://img.shields.io/badge/olympuz-family-B8D6EA.svg)](https://github.com/zarubinvibe/athena#olympuz-family)

<p align="center"><img src="docs/assets/pantheon/hero.png" alt="Athena in white marble with an owl, a round shield, and layered knowledge planes beside the classical column" width="100%"></p>

## Contents

- [What This Is](#what-this-is)
- [Why It Helps](#why-it-helps)
- [The Main Advantage](#the-main-advantage)
- [How It Works](#how-it-works)
- [Quickstart](#quickstart)
- [Simple Comparison](#simple-comparison)
- [Simple Words](#simple-words)
- [Safety And Privacy](#safety-and-privacy)
- [Limits](#limits)
- [Star And Contribute](#star-and-contribute)

<!-- beginner-readme:start -->

## What This Is

Athena is a portable agent operating system for macOS. It keeps your agent setup as tracked templates, manifests, and checks, then rebuilds that setup on a new machine layer by layer. Rules, skills, projects, and knowledge come back where they belong.

## Why It Helps

A working agent setup grows for months: configs, skills, hooks, registries, cloned projects. A new laptop or a broken profile wipes it. Athena turns that setup into something you can rebuild in one evening instead of remembering it by hand.

## The Main Advantage

**Main advantage:** the setup is a repeatable program, not a memory.

**Why this is better:** every layer can run again without breaking what already exists, and `--dry-run` shows the whole plan before a single file is written.

## How It Works

Bootstrap runs ordered layers. You can preview them, run one slice, or run the whole chain.

<!-- workflow-diagram:start -->

```text
  ┌─────────┐   ┌─────────┐   ┌─────────┐
  │ Prepare │ ▶ │ Choose  │ ▶ │ Dry run │
  └─────────┘   └─────────┘   └─────────┘
       ▼
  ┌─────────┐   ┌─────────┐   ┌─────────┐
  │ Build   │ ▶ │ Checks  │ ▶ │ Report  │
  └─────────┘   └─────────┘   └─────────┘
```

<!-- workflow-diagram:end -->

| Stage | What happens |
|---|---|
| 1. Prepare | Base tools land first: Xcode CLT, Homebrew, chezmoi, Node, Git |
| 2. Choose | An interview picks integrations and personal layers |
| 3. Dry run | The full plan is printed before anything is written |
| 4. Build | Tools, dotfiles, plugins, registry, projects, vault, launchd |
| 5. Checks | Shell syntax, templates, contracts, clean-room render |
| 6. Report | Local routing status and a weekly HTML report |

### Step 1: Prepare the Mac

`preinstall.sh` installs the base tools and a pinned Claude Code CLI, then leaves the repository at `~/athena`. You can read the script before running it.

**You get:** a Mac that can run the rest of the setup.

### Step 2: Choose what you want

Start Claude Code in the repository and type `/setup-os`. It asks which integrations and personal layers you want and writes the chosen configuration. Anything skipped can be added later.

**You get:** a configuration file that describes your machine, not someone else's.

### Step 3: Preview every layer

`./bootstrap.sh --dry-run` prints what each layer would do. Nothing is installed, cloned, or overwritten during the preview.

**You get:** a readable plan you can accept or edit before the real run.

### Step 4: Build the layers

Layers run in order: Homebrew baseline, optional tools, merged chezmoi templates, Claude Code plugins, the capability registry, project clones, an optional knowledge vault, secret storage and scheduled jobs.

**You get:** a workspace where agents, skills, and projects are back in place.

### Step 5: Run the checks

The same checks the repository workflow runs are available locally: shellcheck, the smoke suite, and a clean-room render into a temporary directory.

**You get:** green checks, or a named file and line to fix.

### Step 6: Watch the daily state

After the agent layer is installed, a status command reads local records and exits red when a job failed without a retry. A weekly command builds an HTML report from the same data.

**You get:** a short answer to the question "did anything quietly break this week".

## Quickstart

Athena targets macOS. You need an administrator account and network access for Homebrew, npm, and Git.

```bash
git clone https://github.com/zarubinvibe/athena.git "$HOME/athena"
cd "$HOME/athena"
less preinstall.sh
./preinstall.sh
```

In a hurry on a fresh Mac? Run the same tracked script directly: `curl -fsSL https://raw.githubusercontent.com/zarubinvibe/athena/main/preinstall.sh | bash`. No Git at all? Take [the ZIP](https://github.com/zarubinvibe/athena/archive/refs/heads/main.zip) and unpack it. Then open Claude Code in the folder and type `/setup-os`.

**You get:** the base tools are installed, the repository sits at `~/athena`, and `/setup-os` is ready to ask what you want.

## Simple Comparison

| Choice | Best when | What you get | Trade-off |
|---|---|---|---|
| **Athena** | You want the whole agent workspace back, not just dotfiles | Ordered layers, dry run, registry, projects, checks | macOS only |
| Setting it up by hand | One machine, once | Full control and nothing extra | A day of work and no way to repeat it |
| A dotfiles repository alone | You mainly need shell and editor configs | Simple and portable | It does not restore agent skills, registries, or projects |
| A cloud dev environment | Work happens in a container | Identical environment for a team | Your local machine and local agents stay unconfigured |

## Simple Words

| Word | Simple meaning |
|---|---|
| Repository | The project folder that Git stores and versions |
| Terminal | The window where you type commands |
| Command | One instruction you give the computer |
| Branch | A separate line of changes that does not touch `main` |
| Pull Request | A request to review your change and accept it |
| Layer | One ordered step of the setup that can run again safely |
| Dry run | A preview that prints the plan and changes nothing |

## Safety And Privacy

- File access: layers write under your home folder and the paths named in your own configuration.
- Shell and network: setup runs Homebrew, npm, Git, chezmoi, and launchd commands and fetches configured repositories.
- Secrets: values belong in the macOS Keychain or `~/.secrets`, never in tracked manifests or templates.
- Approvals: `/setup-os` asks about optional parts; a direct bootstrap run follows your configuration without prompts.
- Guardrails: bundled hooks block known risky patterns, but they are not a system sandbox.
- Telemetry: Athena's own scripts send nothing. Installed third-party tools keep their own policies.
- Recovery: `athena-update` takes a backup and shows a diff before applying live changes.

Read [SECURITY.md](SECURITY.md) before running Athena on a machine that holds important data.

## Limits

Status: public macOS reference implementation. Automated checks cover shell syntax, templates, hygiene, agent contracts, and a clean-room render.

- macOS only: Homebrew prompts, Xcode dialogs, and launchd behaviour are platform specific.
- A first install on a fresh Mac still needs the manual acceptance checklist.
- The clean-room test renders into temporary folders and does not replace a real install.
- Private repositories, credentials, and personal knowledge are yours and cannot be validated by the public clone.
- Third-party versions and login flows change on their own schedule.

Deeper reading: [the full reference](docs/DETAILS.md), [feature reference](docs/FEATURES.en.md), [filesystem contract](rules/structure.md), [roadmap](specs/00-roadmap.md), and [live acceptance](smoke/live-acceptance.md).

## Star And Contribute

Useful? Give Athena a star: [https://github.com/zarubinvibe/athena](https://github.com/zarubinvibe/athena). It takes a second and it decides whether other people ever find the project.

Want to change something? The path is short: fork the repository, create a branch, commit your change, push the branch, then open a Pull Request. Do not push directly to `main`; the release gate rejects it.

Found a problem instead? Open an issue at [https://github.com/zarubinvibe/athena/issues](https://github.com/zarubinvibe/athena/issues) and say what you ran and what happened.

<!-- beginner-readme:end -->

<!-- pantheon-family:start -->
## Olympuz family

This is one of the public [Olympuz projects](https://github.com/zarubinvibe/athena#olympuz-family). Each row opens the repository or downloads its source as a ZIP.

| Type | Name | What it does | Source |
|---|---|---|---|
| project | Athena | Portable agent OS that restores a complete Claude and Codex setup on a new Mac. | [Repository](https://github.com/zarubinvibe/athena) · [ZIP](https://github.com/zarubinvibe/athena/archive/refs/heads/main.zip) |
| project | Helioz | 24/7 agent work conveyor with verified completion markers and goal-based overnight decisions. | [Repository](https://github.com/zarubinvibe/helioz) · [ZIP](https://github.com/zarubinvibe/helioz/archive/refs/heads/main.zip) |
| project | Mnemazine | Local-first memory system that turns raw inputs into verified reusable knowledge. | [Repository](https://github.com/zarubinvibe/mnemazine) · [ZIP](https://github.com/zarubinvibe/mnemazine/archive/refs/heads/main.zip) |
| project | Themis | Multi-agent assistant for Russian litigation with local OCR and review by a five-jurist council. | [Repository](https://github.com/zarubinvibe/themis) · [ZIP](https://github.com/zarubinvibe/themis/archive/refs/heads/main.zip) |
| project | Zeuz | Factory that turns an idea into a governed multi-agent workflow with gates, observability, and replay. | [Repository](https://github.com/zarubinvibe/zeuz) · [ZIP](https://github.com/zarubinvibe/zeuz/archive/refs/heads/main.zip) |
<!-- pantheon-family:end -->

## License

MIT. See [LICENSE](LICENSE).
