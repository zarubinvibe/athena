# Athena

Athena is a portable agent OS for macOS that rebuilds a Claude Code and Codex workspace from versioned templates, manifests, and checks.

[Russian](README.ru.md) | [Feature reference](docs/FEATURES.en.md) | [Quickstart](#quickstart) | [Security](#security-and-privacy) | [Contributing](CONTRIBUTING.md)

<p align="center">
  <img src="docs/assets/pantheon/emblem.png" width="144" alt="Athena marble emblem with an owl, round shield, knowledge lines, and classical column">
</p>

![Athena marble hero with an owl, round shield, layered knowledge planes, and classical column](docs/assets/pantheon/hero.png)

Status: public macOS reference implementation. Automated checks cover shell syntax, templates, repository hygiene, agent contracts, and generic clean-room rendering. A complete first install on a fresh Mac still needs the manual acceptance checklist.

## Quickstart

Athena targets macOS. The setup needs an administrator account and network access for Homebrew, npm, and Git repositories.

### Reviewable install

Clone the repository, inspect the installer, then run it:

```bash
git clone https://github.com/zarubinvibe/athena.git "$HOME/athena"
cd "$HOME/athena"
less preinstall.sh
./preinstall.sh
```

The installer sets up Xcode Command Line Tools, Homebrew, chezmoi, Node.js, Git, and a pinned Claude Code CLI. It leaves the repository at `~/athena`.

Start Claude Code from the repository and enter `/setup-os`:

```bash
cd "$HOME/athena"
claude
```

`/setup-os` asks which integrations and personal layers you want, writes the selected configuration, and runs the bootstrap layers. Choices can be skipped and added later.

For a fresh Mac, the direct installer runs the same tracked script without a local review step:

```bash
curl -fsSL https://raw.githubusercontent.com/zarubinvibe/athena/main/preinstall.sh | bash
```

If the required tools already exist, preview the engine before applying anything:

```bash
cd "$HOME/athena"
cp athena.config.example.sh athena.config.sh
./bootstrap.sh --dry-run
```

## What Athena builds

`bootstrap.sh` runs ordered, repeatable layers. Use `--only=<layer>` to run one slice.

| Layer | Responsibility |
|---|---|
| `0` | Install the Homebrew tool baseline from `Brewfile`. |
| `0b` | Clone optional tools from `tools.manifest`. |
| `1` | Merge public chezmoi templates with an optional private overlay, then apply the result. |
| `1b` | Reinstall declared Claude Code marketplaces and plugins. |
| `2` | Rebuild and validate the local capability registry. |
| `3` | Clone projects listed in `projects.manifest`. |
| `4` | Restore an optional knowledge vault. |
| `5` | Prepare secret storage, show MCP reauthentication work, and register launchd jobs. |
| `6` | Run repository and installed-state smoke checks. |

The layout keeps three kinds of state apart:

| Plane | Contents | Default location |
|---|---|---|
| Consciousness | Agent rules, skills, hooks, and registries | `~/.claude`, `~/.codex`, `~/.agents` |
| Knowledge | Personal notes and synthesized reference material | `~/Мозг` |
| Work | Projects, active files, and archives | Configured under `$HOME` |

The public repository contains generic templates and examples. Personal repositories, filled manifests, credentials, and knowledge content stay in an optional private overlay. Generic-only setup remains a supported path.

## Examples

Preview every layer or run one layer:

```bash
./bootstrap.sh --dry-run
./bootstrap.sh --only=1
./bootstrap.sh --only=6
```

Run the same checks used by the repository workflow:

```bash
shellcheck -S error bootstrap.sh preinstall.sh smoke/*.sh
./bootstrap.sh --dry-run
bash smoke/smoke.sh
ATHENA_PRIVATE_DIR="$(mktemp -d)" bash smoke/dry-validate.sh
```

After the agent layer is installed, inspect routing activity or build a weekly report:

```bash
node "$HOME/.agents/registry/scripts/athena-status.mjs" --days=7
node "$HOME/.agents/registry/scripts/athena-weekly-report.mjs" --format=html
```

The status command reads local JSONL records and exits with code `1` when it finds failed jobs without a retry. The weekly command writes reports under `~/.agents/reports`.

## How it works

Athena first copies the public `chezmoi/` tree into a local merged source. If `ATHENA_PRIVATE_REPO` is configured, its overlay is applied on top. One `chezmoi init --apply` then renders the selected result. The remaining layers build registries, clone configured work, prepare runtime files, and run checks.

The scripts are designed to converge when run again. Guards stop common secret-path writes and force pushes to protected branches, while smoke tests check tracked files, templates, role contracts, and Claude/Codex parity.

Dotfile deployment uses [chezmoi](https://www.chezmoi.io/). The knowledge layout follows the synthesis-on-write approach described in [the bundled method reference](skills/setup-os/references/karpathy-method.md), popularized by Andrej Karpathy. The project name refers to Athena, the Greek goddess of wisdom and strategy.

The optional thin-session path hides most installed skills from the initial model prompt and routes a short relevant list per task. The measured example in [docs/thin-session.md](docs/thin-session.md) compares about 1,400 listed skills with a small allowlist; token savings depend on the actual skill inventory.

## Security and privacy

- File access: bootstrap layers write under `$HOME` and any paths named in your local configuration.
- Shell and network: setup can run Homebrew, npm, Git, chezmoi, and launchd commands and can fetch configured repositories.
- Secrets: values belong in macOS Keychain or `~/.secrets`, never in tracked manifests or templates.
- Approvals: `/setup-os` asks about optional integrations. Direct `bootstrap.sh` follows the supplied configuration without per-command prompts.
- Guardrails: included hooks block known risky patterns, but they are not an operating-system sandbox.
- Telemetry: Athena's own scripts add no telemetry. Installed third-party tools have separate policies.
- Recovery: `athena-update` takes a backup and shows a chezmoi diff before applying live configuration changes. Direct bootstrap runs rely on idempotency, Git history, and your normal system backups.

Read [SECURITY.md](SECURITY.md) before using Athena on a machine with important data.

## Documentation

- [Feature reference](docs/FEATURES.en.md): implemented layers and agent contracts.
- [Configuration example](athena.config.example.sh): public/private source and manifest paths.
- [Filesystem contract](rules/structure.md): canonical directory ownership.
- [Roadmap](specs/00-roadmap.md): completed phases and remaining acceptance work.
- [Architecture decisions](docs/decisions/): design records, including generic/private merge behavior.
- [Clean-room protocol](specs/05-clean-room-protocol.md): what the automated render test proves.
- [Live acceptance](smoke/live-acceptance.md): fresh-Mac and update checklist.
- [Contributing](CONTRIBUTING.md): setup and required checks.

## Status and known limits

The repository implements phases 1 through 8 in [the roadmap](specs/00-roadmap.md), including the layered bootstrap, onboarding, local agent contract, parity checks, status snapshot, and weekly report.

Known limits:

- Athena supports macOS. Homebrew password prompts, Xcode dialogs, and launchd behavior are platform-specific.
- The clean-room test renders into temporary directories. It does not replace a full install and real launchd registration on a fresh Mac or VM.
- Private repositories, credentials, MCP accounts, and personal knowledge are user-supplied and cannot be validated by the public clone.
- Shell guards reduce common mistakes. They do not provide process isolation or a kernel sandbox.
- Third-party package versions and authentication flows can change independently of this repository.

## Contributing

Read [CONTRIBUTING.md](CONTRIBUTING.md). The short check path is:

```bash
shellcheck -S error bootstrap.sh preinstall.sh smoke/*.sh
bash smoke/smoke.sh
git diff --check
```

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

Athena is available under the [MIT License](LICENSE). Copyright remains with the Athena authors and contributors.
