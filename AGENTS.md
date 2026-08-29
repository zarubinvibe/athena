# Agent guide

Athena is a macOS bootstrap repository. Read `CLAUDE.md` and the relevant file in `specs/` before changing behavior.

## Working rules

- Keep paths relative to the repository or based on `$HOME`. Never add a literal user home path.
- Keep real manifests, credentials, private overlays, logs, sessions, and runtime state out of Git.
- Treat `rules/structure.md` as the filesystem layout contract.
- Keep root skills and their `chezmoi/dot_claude/skills/` copies in sync.
- Do not apply the repository to a live `~/.claude` while testing. Use dry runs and temporary homes.
- Keep English and Russian public documentation consistent in meaning.
- Prefer shell, Node.js, and other tools already used by the repository. Do not add a dependency for a small check.

## Required checks

Run these after behavior or documentation changes:

```bash
shellcheck -S error bootstrap.sh preinstall.sh smoke/*.sh
./bootstrap.sh --dry-run
bash smoke/smoke.sh
ATHENA_PRIVATE_DIR="$(mktemp -d)" bash smoke/dry-validate.sh
git diff --check
```

The smoke suite must finish with `SMOKE OK`. Do not describe work as complete when a required check fails.

## Public packaging

- `README.md` and `README.ru.md` follow the shared family anatomy: promise, badges, wide hero, table of
  contents, and the ten beginner headings with the ASCII workflow diagram inside `How It Works`.
- Workflow stages live in `.github/pantheon.json`. Change the stages there first, then the two READMEs.
- `AGENTS.md` holds the rules; `CLAUDE.md`, `GEMINI.md`, `.github/copilot-instructions.md`, and
  `.cursor/rules/*.mdc` only point here.
- Run `public-repo-gate check --repo . --release-intent public` before any push, and fix every blocker.
