> Long-form reference kept from the previous README. The short beginner page lives in [README.md](../README.md).

# Athena

Athena - переносимая агентная ОС для macOS: собирает окружение Claude Code и Codex из версионируемых шаблонов, манифестов и проверок.

[English](../README.md) | [Справочник функций](../docs/FEATURES.ru.md) | [Быстрый старт](#быстрый-старт) | [Безопасность](#безопасность-и-приватность) | [Участие](../CONTRIBUTING.md)

<p align="center">
  <img src="../docs/assets/pantheon/emblem.png" width="144" alt="Мраморная эмблема Athena с совой, круглым щитом, схемой знаний и классической колонной">
</p>

![Мраморная Athena с совой, круглым щитом, слоями знаний и классической колонной](../docs/assets/pantheon/hero.png)

Статус: публичная рабочая реализация для macOS. Автотесты проверяют shell-синтаксис, шаблоны, чистоту репозитория, агентные контракты и generic clean-room рендер. Полную установку на свежий Mac по-прежнему принимают вручную по чеклисту.

## Быстрый старт

Нужен Mac, учётная запись администратора и сеть для Homebrew, npm и Git-репозиториев.

### Установка с просмотром кода

Склонируй репозиторий, прочитай installer и запусти его:

```bash
git clone https://github.com/zarubinvibe/athena.git "$HOME/athena"
cd "$HOME/athena"
less preinstall.sh
./preinstall.sh
```

Скрипт ставит Xcode Command Line Tools, Homebrew, chezmoi, Node.js, Git и закреплённую версию Claude Code CLI. Репозиторий остаётся в `~/athena`.

Запусти Claude Code из каталога Athena, затем введи `/setup-os`:

```bash
cd "$HOME/athena"
claude
```

`/setup-os` спрашивает про интеграции и личные слои, записывает выбранную конфигурацию и запускает bootstrap. Любой необязательный пункт можно пропустить и добавить позже.

На свежем Mac доступен прямой запуск того же tracked-скрипта, но без локального просмотра:

```bash
curl -fsSL https://raw.githubusercontent.com/zarubinvibe/athena/main/preinstall.sh | bash
```

Если нужные инструменты уже стоят, сначала посмотри dry-run:

```bash
cd "$HOME/athena"
cp athena.config.example.sh athena.config.sh
./bootstrap.sh --dry-run
```

## Что собирает Athena

`bootstrap.sh` идёт по упорядоченным повторяемым слоям. Флаг `--only=<слой>` запускает один срез.

| Слой | Что происходит |
|---|---|
| `0` | Ставится базовый набор Homebrew из `Brewfile`. |
| `0b` | Клонируются необязательные инструменты из `tools.manifest`. |
| `1` | Публичные chezmoi-шаблоны смешиваются с необязательным приватным overlay и применяются. |
| `1b` | Восстанавливаются объявленные marketplaces и плагины Claude Code. |
| `2` | Пересобирается и проверяется локальный реестр способностей. |
| `3` | Клонируются проекты из `projects.manifest`. |
| `4` | Восстанавливается необязательный vault знаний. |
| `5` | Готовится хранилище секретов, показываются шаги MCP-авторизации, регистрируются launchd-задачи. |
| `6` | Запускаются smoke-проверки репозитория и установленного состояния. |

Три плоскости не смешиваются:

| Плоскость | Что хранит | Базовое место |
|---|---|---|
| Сознание | Правила агентов, skills, hooks и реестры | `~/.claude`, `~/.codex`, `~/.agents` |
| Знания | Личные заметки и синтезированные справочные материалы | `~/Мозг` |
| Работа | Проекты, активные файлы и архивы | Настраиваемые каталоги внутри `$HOME` |

В публичном репозитории лежат generic-шаблоны и примеры. Личные репозитории, заполненные манифесты, учётные данные и содержимое vault остаются в необязательном приватном overlay. Публичный generic-вариант работает и без него.

## Примеры

Посмотреть все слои или запустить один:

```bash
./bootstrap.sh --dry-run
./bootstrap.sh --only=1
./bootstrap.sh --only=6
```

Прогнать тот же набор, который использует workflow репозитория:

```bash
shellcheck -S error bootstrap.sh preinstall.sh smoke/*.sh
./bootstrap.sh --dry-run
bash smoke/smoke.sh
ATHENA_PRIVATE_DIR="$(mktemp -d)" bash smoke/dry-validate.sh
```

После установки агентного слоя можно посмотреть routing-активность или собрать недельный отчёт:

```bash
node "$HOME/.agents/registry/scripts/athena-status.mjs" --days=7
node "$HOME/.agents/registry/scripts/athena-weekly-report.mjs" --format=html
```

Первая команда читает локальные JSONL-записи и возвращает код `1`, если видит упавшие задачи без retry. Вторая пишет отчёты в `~/.agents/reports`.

## Как это работает

Athena копирует публичное дерево `chezmoi/` в локальный merged source. Если настроен `ATHENA_PRIVATE_REPO`, его overlay ложится сверху. Затем один `chezmoi init --apply` рендерит выбранный вариант. Остальные слои собирают реестры, клонируют настроенные проекты, готовят runtime и запускают проверки.

Повторный запуск сходится к тому же состоянию. Guards режут типичные записи в секретные пути и force push в защищённые ветки. Smoke проверяет tracked-файлы, шаблоны, паспорта ролей и паритет Claude/Codex.

Дотфайлы раскладывает [chezmoi](https://www.chezmoi.io/). Слой знаний использует synthesis-on-write из [встроенного описания метода](../skills/setup-os/references/karpathy-method.md), который популяризировал Андрей Карпаты. Название проекта отсылает к Афине, древнегреческой богине мудрости и стратегии.

Необязательный thin-session режим скрывает большую часть skills из стартового prompt и подбирает короткий список под задачу. Замер в [docs/thin-session.md](../docs/thin-session.md) сравнивает около 1400 видимых skills с малым allowlist. Экономия токенов зависит от реального набора.

## Безопасность и приватность

- Файлы: bootstrap пишет внутрь `$HOME` и в пути из локальной конфигурации.
- Shell и сеть: установка запускает Homebrew, npm, Git, chezmoi и launchd, а также скачивает настроенные репозитории.
- Секреты: значения хранятся в Keychain macOS или `~/.secrets`, но не в tracked-манифестах и шаблонах.
- Подтверждения: `/setup-os` спрашивает про необязательные интеграции. Прямой `bootstrap.sh` выполняет конфигурацию без вопроса перед каждой командой.
- Защита: встроенные hooks блокируют известные опасные паттерны, но не заменяют системный sandbox.
- Телеметрия: собственные shell- и Node.js-скрипты Athena её не добавляют. У сторонних инструментов свои правила.
- Восстановление: `athena-update` снимает бэкап и показывает chezmoi diff до правок живой конфигурации. Прямой bootstrap опирается на идемпотентность, историю Git и обычные системные бэкапы.

Перед установкой на важную машину прочитай [SECURITY.md](../SECURITY.md).

## Документация

- [Справочник функций](../docs/FEATURES.ru.md): реализованные слои и агентные контракты.
- [Пример конфигурации](../athena.config.example.sh): public/private source и пути манифестов.
- [Контракт файловой системы](../rules/structure.md): кому принадлежат каталоги.
- [Roadmap](../specs/00-roadmap.md): завершённые фазы и оставшаяся приёмка.
- [Архитектурные решения](../docs/decisions/): в том числе generic/private merge.
- [Clean-room протокол](../specs/05-clean-room-protocol.md): что именно доказывает автоматический рендер.
- [Live acceptance](../smoke/live-acceptance.md): чеклист свежего Mac и обновления.
- [Участие](../CONTRIBUTING.md): окружение и обязательные проверки.

## Статус и ограничения

В репозитории реализованы фазы 1-8 из [roadmap](../specs/00-roadmap.md): слоистый bootstrap, onboarding, local agent contract, parity checks, status snapshot и недельный отчёт.

Ограничения:

- Athena работает на macOS. Пароль Homebrew, диалоги Xcode и launchd привязаны к платформе.
- Clean-room тест рендерит шаблоны во временные каталоги. Он не заменяет полную установку и реальную регистрацию launchd на свежем Mac или VM.
- Приватные репозитории, credentials, MCP-аккаунты и личные знания добавляет пользователь. Публичный клон их не проверяет.
- Shell guards ловят типовые ошибки, но не изолируют процессы на уровне ядра.
- Версии сторонних пакетов и их авторизация меняются независимо от Athena.

## Как помочь проекту

Правила лежат в [CONTRIBUTING.md](../CONTRIBUTING.md). Короткий набор проверок:

```bash
shellcheck -S error bootstrap.sh preinstall.sh smoke/*.sh
bash smoke/smoke.sh
git diff --check
```



## Лицензия

Athena распространяется по [лицензии MIT](../LICENSE). Права сохраняются за авторами и участниками проекта.
