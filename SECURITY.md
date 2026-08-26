# Security policy

Athena installs tools, renders dotfiles, clones configured repositories, and registers launchd jobs. Review scripts and configuration before running them on a machine that holds important data.

## Trust boundaries

- `./bootstrap.sh --dry-run` prints planned bootstrap commands before a real run.
- `preinstall.sh` downloads Homebrew and installs a pinned Claude Code package. It needs network access and may ask for the Mac administrator password through the upstream installers.
- `bootstrap.sh` can write under `$HOME`, run Homebrew and npm, clone Git repositories, apply chezmoi templates, and call `launchctl`.
- The included guards reject common secret paths and destructive shell patterns. They are policy checks, not an operating-system sandbox.
- Athena's own shell and Node.js scripts do not add telemetry. Installed third-party tools keep their own telemetry and privacy policies.

## Secrets

Keep secret values in macOS Keychain or a local directory with mode `700`, such as `~/.secrets`. Never commit `.env` files, private keys, filled manifests, authentication files, sessions, logs, or private overlays.

If a secret reaches Git history, revoke or rotate it first. Removing the file from the latest commit is not enough.

## Reporting a problem

Do not post credentials, private paths, or exploit details in a public issue. Use the repository's private vulnerability reporting channel when it is available. Otherwise open a minimal issue that contains no sensitive material and asks the maintainers for a private contact route.

This project does not publish a response-time SLA.
