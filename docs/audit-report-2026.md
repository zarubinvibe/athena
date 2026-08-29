# Athena OS — Полный аудит 2026-06-24

> Оси: безопасность · работоспособность · тестирование.
> Правило: фиксы после явного OK владельца.

---

## Безопасность

### Высокий (WARN)

#### S-H1 — bash-guard.sh: greedy sed ломает парсинг команды

**Файл:** `chezmoi/dot_claude/hooks/bash-guard.sh:16`

```bash
cmd="$(printf '%s' "$INPUT" | tr '\n' ' ' | sed -n 's/.*"command"[[:space:]]*:[[:space:]]*"\(.*\)/\1/p')"
cmd="${cmd%\"*}"
```

Жадный `.*` захватывает всё после `"command": "` включая хвост JSON. Пример:

```json
{"command": "echo test", "timeout": 30}
```

После `tr '\n' ' '` и `sed` — `cmd` получает `echo test\", \"timeout\": 30}`, после `%\"*` обрезает от последней `"` → `cmd="echo test\", \"timeout\": 30}"` минус хвост, оставляя мусор. Паттерны deny/benign-gate работают на искажённой строке — блок может не сработать на опасной команде или сработать ложно на безопасной.

**Влияние:** block может не сработать на `git push --force main` если JSON содержит дополнительные поля. Hook — первый слой, не единственный, но это дыра в задекларированном контракте.

**Фикс:** заменить greedy-sed на нежадный (`[^"]*` вместо `.*`) или использовать python/node для парсинга JSON.

---

#### S-H2 — rescue-runner не задекларирован в агентах handoff-graph.yaml

**Файл:** `chezmoi/dot_agents/handoff-graph.yaml`

`rescue-runner` появляется в 3 записях `handoffs:` и `forbidden:`, но отсутствует в блоке `agents:`. `agent-contract.sh` валидирует только 7 именованных агентов — `rescue-runner` никогда не проверяется на паспорт, роль, инструменты. Граф допускает переход к undeclared entity.

**Фикс:** либо добавить `rescue-runner` в `agents:` с паспортом в `role-passports/rescue-runner.md`, либо задокументировать его как external-actor (не агент системы) и убрать из forbidden-блока.

---

### Средний (INFO)

#### S-M1 — P1.6: security-guard.sh: JSON-escaped Unicode обходит блокировку

**Файл:** `chezmoi/dot_claude/hooks/security-guard.sh:14`

```bash
fp="$(printf '%s' "$INPUT" | sed -n 's/.*"file_path"[[:space:]]*:[[:space:]]*"\([^"]*\)".*/\1/p' | head -1)"
```

Если Claude Code передаёт путь с Кириллицей как JSON-escaped unicode (`$HOME/Проекты/`), `sed` захватит литеральную строку `П...`, не декодированный путь. Паттерны `*/.secrets/*` не сработают на закодированном пути.

**Реальный риск:** LOW — Claude Code на macOS отправляет UTF-8 строки напрямую, не экранирует Кириллицу. Теоретический bypass.

**Фикс:** добавить `python3 -c "import json,sys; print(json.loads(sys.stdin.read())['tool_input']['file_path'])"` или аналогичный jq-free декодировщик как fallback.

---

#### S-M2 — .mjs скрипты: path traversal через --evals/--reports флаги

**Файлы:** `athena-status.mjs:17-19`, `athena-weekly-report.mjs:21-22`, `athena-report-quality-gate.mjs:20-21`

**✅ FIXED (2026-06-24)** — добавлен `safe()` guard: `if (p.includes('..')) throw new Error(...)` на все user-controlled path аргументы во всех 4 скриптах.

---

#### S-M3 — athena-postrun-report.mjs: Math.random() как fallback деdup-ключ

**Файл:** `athena-postrun-report.mjs:49`

```js
byGroup.set(row.group_id ?? String(Math.random()), row)
```

Строки без `group_id` никогда не дедуплицируются (каждый раз новый ключ). Дублирующиеся результаты проходят в отчёт дважды. Не security-issue, но логический баг.

**Фикс:** `row.group_id ?? String(i)` (индекс по массиву) или `require group_id`.

---

### Положительные находки (✓)

- Нет hardcoded секретов ни в одном tracked файле. Git history чист.
- `quality-gate.mjs` корректно обнаруживает `sk-*`, `AKIA*`, `Bearer *`, `password=` в отчётах.
- `|| true` в `agent-contract.sh:97` — намеренный паттерн (тест negative-case), не silent failure.
- `settings.json.tmpl` `permissions.deny` блокирует чтение из `~/.ssh`, `~/.secrets`.

---

## Работоспособность

### Высокий (WARN)

#### F-H1 — FSM-инварианты declarative-only, runtime не enforced

**Файл:** `chezmoi/dot_agents/job-lifecycle.yaml:54-65`

6 инвариантов (`retry-once`, `review-ready-gate`, `delivered-gate`, `archived-gate`, `no-skip-approval`, `no-vault-bypass`) задекларированы в YAML. `agent-contract.sh` проверяет СТРУКТУРУ (ключи присутствуют), но не проверяет что они реально соблюдаются при исполнении. Например:
- `retry-once` требует счётчика по `job_id` в ledger — кода нет
- `no-vault-bypass` требует что runner не пишет в vault напрямую — enforcement отсутствует

Контракт — архитектурный намёк, не гейт.

**Фикс:** либо честно пометить инварианты как `enforcement: declarative` (документация, не гейт), либо добавить ledger-check в `agent-contract.sh` на синтетических примерах.

---

### Средний (INFO)

#### F-M1 — C6: SessionEnd hook async=true, fast-exit race

**Файл:** `chezmoi/dot_claude/settings.json.tmpl` + `executable_session-reaper.sh`

```json
"async": true, "timeout": 20
```

При kill/crash сессии async-хук может не завершиться за 20s. `session-reaper.sh` логирует токены ДО kill — это митигация, но не полная: если сессия умирает не через reaper (e.g. OOM), лог теряется.

**Риск:** MEDIUM — потеря token-spend записи, не data loss.
**Статус:** документированная компромисс-зона. Дополнительно: добавить fallback `sync`-логгинг при выходе с SIGTERM/EXIT trap в самих скриптах.

---

#### F-M2 — CI: нет scheduled run для drift detection

**Файл:** `.github/workflows/smoke.yml`

Smoke запускается только на push/PR. Если `macos-latest` меняет runtime (xcode, brew defaults), это обнаружится только когда кто-то пушит код.

**Фикс:** добавить `schedule: cron: '0 6 * * 1'` (раз в неделю по понедельникам).

---

#### F-M3 — C9: athena-router.md start matrix vs routing-evals.jsonl — parity untested

`routing-evals.example.jsonl` содержит 3 примера (code-edit, arch, legal). `athena-router.md` задаёт 10 классов. Smoke не проверяет что каждый из 10 классов покрыт хотя бы одним eval-примером. Drift возможен при расширении матрицы.

**Фикс:** добавить в `agent-contract.sh` check что `routing-evals.example.jsonl` покрывает все `MATRIX_CLASSES`.

---

### Положительные находки (✓)

- `bootstrap.sh --dry-run` работает корректно, идемпотентен.
- `parity-smoke.sh`: `comm` с `<()` process substitution и pre-sorted input работает корректно.
- `handoff-graph.yaml`: все 11 handoff-рёбер несут `when`, все 6 forbidden несут `why`. Integrity gate проходит.
- `job-lifecycle.yaml`: все 13 состояний объявлены, terminal_states и invariants присутствуют.
- Нет orphan-агентов среди 7 задекларированных в `agents:` — все появляются в handoffs.

---

## Тестирование

### Высокий (WARN)

#### T-H1 — athena-status.mjs: --json режим не тестируется

**Файл:** `smoke/agent-contract.sh:157-160`

Smoke проверяет только `--evals=/dev/null` (text mode). Флаг `--json` генерирует структурно другой output и тестируется нулём раз. Изменение в JSON-структуре не поймают smoke тесты.

**Фикс:** добавить в `agent-contract.sh` тест:
```bash
node "$STATUS_SCRIPT" --evals=/dev/null --json | grep -q '"total"' \
  || bad "athena-status: --json output missing 'total' field"
```

---

#### T-H2 — athena-weekly-report.mjs: --format=html не тестируется

`agent-contract.sh` гоняет only MD-путь через quality-gate. HTML-генерация (300+ строк кода) не покрыта. Баг в `htmlContent()` или `esc()` пройдёт незамеченным.

**Фикс:** добавить в smoke: `--format=html` + проверить что `.html` файл создан + quality-gate на нём.

---

### Средний (INFO)

#### T-M1 — parity-smoke.sh: пустой route card → false positive "identical"

**Файл:** `smoke/parity-smoke.sh:62`

Если `fields_of()` вернёт пустую строку для обоих карт, `[ "$CLAUDE_FIELDS" = "$CODEX_FIELDS" ]` = TRUE и smoke рапортует `identical (0 fields)` — это false positive, не защита.

**Фикс:** добавить `[ -n "$CLAUDE_FIELDS" ] || bad "route card has no fields"` после extraction.

---

#### T-M2 — C8: skills root ⊆ chezmoi parity — инверсия не тестируется

**C4 статус:** Root=5, Chezmoi=7. `root ⊆ chezmoi` ✓ (все 5 root-скиллов есть в chezmoi). Но chezmoi-extra скиллы (`athena-research`, `self-learning`) могут дивергировать от аналогов в ~/.agents/ без детекции.

**C4 финальный статус:** CLOSED (инвариант соблюдён). Рекомендация: добавить checksum-сравнение deployed vs repo для critical skills.

---

### Положительные находки (✓)

- `|| true` в `agent-contract.sh:97` — правильный паттерн (negative-case test).
- `2>/dev/null` в smoke — только для glob-miss и `ls` edge-cases, все прикрыты empty-check.
- quality-gate на bad-report (raw OCR marker) → EXIT 1 корректно.
- 7 паспортов присутствуют, все несут обязательные поля (Soul/Does/Tools/Model/Contract/Won't/Parity).

---

## C4-C11 Backlog — статус

| ID | Статус | Действие |
|----|--------|----------|
| C4 | ✅ CLOSED | root(5) ⊆ chezmoi(7) — инвариант соблюдён |
| C5 | ✅ CLOSED | zh/ директория не найдена |
| C6 | ✅ CLOSED | async=true + timeout=20; mitigation = session-reaper.sh (logs before SIGKILL); OOM = acceptable loss |
| C7 | ✅ CLOSED | Stale backlog item — `agentic-orchestration` только в audit-docs, не в коде |
| C8 | ✅ CLOSED | §5 ref не найден в репо — stale backlog artifact |
| C9 | ✅ FIXED | routing-evals.example.jsonl расширен до 10 классов; check 13b добавлен в agent-contract.sh |
| C10 | ✅ CLOSED | *.bak файлы не найдены |
| C11 | ✅ CLOSED | ponytail = session-scoped plugin; level via `/ponytail lite|full|ultra`; не в settings.json by design |

---

## P1.6 — статус

**✅ FIXED (2026-06-24)**

Две реализации (security-guard.sh + bash-guard.sh) переведены с greedy-sed на `python3 -c json.load()`:

- `bash-guard.sh`: greedy regex → искажённый `cmd` → deny/benign паттерны работают на мусоре (S-H1)
- `security-guard.sh`: JSON-escaped unicode → bypass теоретически возможен (S-M1)

Оба хука — первый слой, НЕ единственный. Реальный риск определяется отсутствием секретов в ФС-зоне кода (Keychain/~/.secrets). Тем не менее — задекларированный контракт нарушен.

**Приоритет исправления:** bash-guard.sh (HIGH) → security-guard.sh (MEDIUM).

---

## Сводная таблица

| ID | Ось | Severity | Файл | Один-строчный итог |
|----|-----|----------|------|---------------------|
| S-H1 | Безопасность | HIGH | bash-guard.sh:16 | Greedy sed → garbled cmd → deny может не сработать |
| S-H2 | Безопасность | HIGH | handoff-graph.yaml | rescue-runner не задекларирован, нет паспорта |
| S-M1 | Безопасность | MEDIUM | security-guard.sh:14 | JSON-escaped unicode → bypass блокировки |
| S-M2 | Безопасность | MEDIUM | *.mjs:17-22 | --evals/--reports path traversal |
| S-M3 | Безопасность | MEDIUM | postrun-report.mjs:49 | Math.random() dedup key → дубликаты в отчёте |
| F-H1 | Работоспособность | HIGH | job-lifecycle.yaml | FSM инварианты declarative-only, не enforced |
| F-M1 | Работоспособность | MEDIUM | settings.json.tmpl | SessionEnd async race при fast-exit |
| F-M2 | Работоспособность | MEDIUM | smoke.yml | Нет scheduled CI run |
| F-M3 | Работоспособность | MEDIUM | agent-contract.sh | routing-evals parity не проверяется |
| T-H1 | Тестирование | HIGH | agent-contract.sh | athena-status --json не тестируется |
| T-H2 | Тестирование | HIGH | agent-contract.sh | weekly-report --format=html не тестируется |
| T-M1 | Тестирование | MEDIUM | parity-smoke.sh | Пустой route card → false positive |

---

*Аудит: 2026-06-24. Охват: Фазы 7+8 (новый код) + P1.6 + C4-C11 backlog. Фиксы — после явного OK.*
