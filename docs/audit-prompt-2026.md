# Полная перепроверка Athena OS — промпт для новой сессии

> Скопируй как первое сообщение в новой Claude Code сессии в директории `$HOME/Проекты/athena-os`.

---

## Контекст проекта

Репо: `~/Проекты/athena-os` (публичный, MIT, github.com/zarubinphil/athena).
Athena — переносимая агентная ОС для Mac. Bootstrap одной командой.

**Стек:**
- Shell scripts (`bootstrap.sh`, `preinstall.sh`, `smoke/*.sh`, `hooks/*.sh`)
- Node.js ES modules (`chezmoi/dot_agents/registry/scripts/*.mjs`)
- YAML конфиги (`chezmoi/dot_agents/*.yaml`, `chezmoi/dot_agents/role-passports/*.md`)
- chezmoi шаблоны (`chezmoi/**/*.tmpl`)
- CI: `.github/workflows/smoke.yml` (macOS-latest)

**Состояние:** Фазы 1–8 закрыты, смержены в main. Последние фазы (7+8) добавили ~1200 строк нового кода, который **никогда не проходил формальный review**.

---

## Задача

Провести полную перепроверку по трём осям: **безопасность · работоспособность · тестирование**.

**Правило:** не реализовывать фиксы без плана + моего ОК. Сначала аудит → отчёт → утверждение → фиксы.

---

## Известные проблемы (начать с них)

### P1.6 — незакрыт
`hooks/security-guard.sh` использует `sed` для парсинга JSON из `settings.json`. Кириллица и многобайтовые строки могут обходить паттерн-матч. Файл: `chezmoi/dot_claude/hooks/security-guard.sh`.

### C4-C11 backlog (из аудита 2026-06-16)
- **C4** — smoke-инверсия: проверить что root `skills/` ⊆ `chezmoi/dot_claude/skills/` (drift между репо и deployed)
- **C5** — удалить `zh/` директорию если она осталась
- **C6** — SessionEnd hook: async vs sync (может пропускать cleanup на быстром завершении)
- **C7** — skill `agentic-orchestration` — проверить актуальность
- **C8** — INDEX §5 — обновить секцию
- **C9** — routing-loop parity — `athena-router.md` eval-loop vs реальный `routing-evals.jsonl`
- **C10** — bak-cleanup: `*.bak` и `*.bak-phase*` файлы в репо
- **C11** — `ponytail→lite A/B` — проверить конфиг уровня

### Известный риск в parity-smoke.sh
`smoke/parity-smoke.sh` использует `comm` с process substitution `<()`. Требует bash (shebang OK), но `comm` требует отсортированный input — `fields_of()` вызывает `sort -u`, должно работать, но не протестировано на краевых случаях (пустой файл, Windows line endings).

---

## Порядок работы

### Шаг 1 — Безопасность (4 инструмента)

1. **`/security-scan`** — AgentShield scan `.claude/` конфига: `settings.json` deny-лист, hooks injection, CLAUDE.md промпт-инъекция
2. **Агент `security-reviewer`** — проверить все `.mjs` файлы в `chezmoi/dot_agents/registry/scripts/`:
   - `athena-status.mjs`, `athena-weekly-report.mjs`, `athena-postrun-report.mjs`, `athena-report-quality-gate.mjs`
   - Риски: path traversal через `--evals`/`--reports` флаги, stdin injection, insecure temp file handling
3. **`implementing-secret-scanning-with-gitleaks`** — публичный репо, полный git history scan
4. **Агент `gsd-security-auditor`** — проверить P1.6: `chezmoi/dot_claude/hooks/security-guard.sh` + все `hooks/*.sh`

### Шаг 2 — Работоспособность (3 инструмента)

5. **`/production-audit`** — bootstrap идемпотентность слоёв 0–6, launchd fail-closed поведение, smoke coverage completeness
6. **`/automation-audit-ops`** — CI `.github/workflows/smoke.yml`, `smoke/smoke.sh`, `smoke/agent-contract.sh`, launchd plist шаблоны; найти `SessionEnd async→sync` (C6)
7. **Агент `agent-architecture-audit`** — Фаза 7 архитектура: 7 паспортов + handoff-graph + FSM invariants:
   - `chezmoi/dot_agents/handoff-graph.yaml`
   - `chezmoi/dot_agents/job-lifecycle.yaml`
   - `chezmoi/dot_agents/role-passports/*.md`
   - Проверить: нет ли orphan переходов, все invariants enforcement реальный или только декларативный

### Шаг 3 — Тестирование (3 инструмента)

8. **Агент `silent-failure-hunter`** — найти все `|| true`, `2>/dev/null`, `|| :` паттерны в `smoke/*.sh`; проверить exit codes в `.mjs` скриптах
9. **`/ponytail-audit`** — весь репо на over-engineering; фокус на `chezmoi/dot_agents/registry/scripts/` (4 скрипта, добавлены в Ф7+Ф8)
10. **Агент `gsd-code-reviewer`** — формальный code review новых файлов Ф7+Ф8:
    - `chezmoi/dot_agents/job-lifecycle.yaml`
    - `claude-starter/project.yaml`
    - `smoke/parity-smoke.sh`
    - `chezmoi/dot_agents/registry/scripts/athena-status.mjs`
    - `chezmoi/dot_agents/registry/scripts/athena-weekly-report.mjs`

---

## Ожидаемый результат

Один сводный отчёт `docs/audit-report-2026.md` со структурой:

```
## Безопасность
### Критично (BLOCK)
### Высокий (WARN)
### Средний (INFO)

## Работоспособность
### Критично
### Высокий
### Средний

## Тестирование
### Критично
### Высокий
### Средний

## C4-C11 backlog — статус
| ID | Статус | Действие |

## P1.6 — статус
```

После отчёта — **стоп, ждать утверждения** перед любыми фиксами.

---

## Ограничения

- Не трогать боевой `~/.claude` без бэкапа
- Деструктивные операции — за явным подтверждением
- Секреты никуда не писать (в репо их не должно быть — verify это)
- `git push` только после явного OK
