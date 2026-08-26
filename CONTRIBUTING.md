# Contributing to Athena

Athena targets macOS and uses Bash, Node.js, Python, chezmoi templates, and launchd plists. Small, focused changes are easiest to review.

## Set up

```bash
git clone https://github.com/zarubinvibe/athena.git
cd athena
brew install shellcheck
```

Do not put real repository URLs, credentials, personal paths, or filled manifests in a contribution. Use the existing example files and temporary directories.

## Check a change

```bash
shellcheck -S error bootstrap.sh preinstall.sh smoke/*.sh
./bootstrap.sh --dry-run
bash smoke/smoke.sh
ATHENA_PRIVATE_DIR="$(mktemp -d)" bash smoke/dry-validate.sh
git diff --check
```

Changes to a root skill must include the matching copy under `chezmoi/dot_claude/skills/`. Changes to filesystem layout must also update `rules/structure.md` and the relevant spec.

Open a pull request with the reason for the change, the affected layer, and the commands you ran. A live clean-Mac or launchd test should say which macOS version and which checklist items from `smoke/live-acceptance.md` were completed.
