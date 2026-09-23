# Инструменты: что поставить на ноутбук

Всё ставится **локально, к себе**. Ничего из этого не требует прав администратора на
корпоративной машине, кроме Docker Desktop и, возможно, Homebrew.

Версии в колонке «проверено» — те, на которых прогонялась подготовка тренинга (macOS,
сентябрь 2026). У вас может быть новее; важно, чтобы команда проверки отработала без ошибки.

**Ставьте до занятия.** На первое упражнение дня 1 заложено 10 минут, а не час.
Если что-то не встало — см. [«Если не ставится»](#если-не-ставится) в конце.

---

## 1. База

| Что | Зачем | Проверка | Проверено |
|---|---|---|---|
| Git | без него ничего | `git --version` | 2.50.1 |
| GitHub CLI (`gh`) | PR и issues из терминала | `gh --version`, `gh auth status` | 2.89.0 |
| Docker + Docker Compose | поднять сервис локально | `docker compose version` | — |
| PHP 8.3+ и Composer | тесты и линтер без Docker | `php -v`, `composer -V` | 8.5.10 / 2.10.3 |
| IDE | VS Code / Cursor / JetBrains | — | — |

### Git

Уже есть почти везде. Если нет: macOS — `xcode-select --install`,
Windows — <https://git-scm.com/download/win>, Linux — `sudo apt install git`.

### GitHub CLI

Официальная инструкция: <https://github.com/cli/cli#installation>

```bash
brew install gh                          # macOS, Linux
winget install --id GitHub.cli           # Windows
sudo apt install gh                      # Debian/Ubuntu, см. ссылку выше про репозиторий
```

Один раз залогиниться — иначе `gh pr create` будет ругаться:

```bash
gh auth login        # выбрать GitHub.com → HTTPS → Login with a web browser
gh auth status       # ожидаем: ✓ Logged in to github.com account <ваш-логин>
```

### Docker Desktop

Официальные инструкции: <https://docs.docker.com/desktop/>
— [macOS](https://docs.docker.com/desktop/setup/install/mac-install/) ·
[Windows](https://docs.docker.com/desktop/setup/install/windows-install/) ·
[Linux](https://docs.docker.com/desktop/setup/install/linux/)

```bash
docker compose version     # ожидаем: Docker Compose version v2.x.x
```

> **Docker'а нет и поставить нельзя?** Это рабочий вариант, не блокер. Локально останутся
> `make test` и `make lint` (им хватает PHP и Composer), а сам сервис вы поднимете на стенде —
> упражнение 1.18. Скажите об этом ведущему в первом же перерыве, чтобы вам завели место на стенде.

### PHP и Composer

```bash
brew install php composer                # macOS
sudo apt install php-cli php-xml composer   # Debian/Ubuntu
winget install PHP.PHP.8.3                  # Windows, Composer — getcomposer.org/download
```

---

## 2. Агент: OpenCode

Единственный агент практикума. Работает в терминале, в том числе во встроенном терминале
вашей IDE — отдельный интерфейс осваивать не нужно.

| | |
|---|---|
| Репозиторий | <https://github.com/anomalyco/opencode> (ранее `sst/opencode`) |
| Документация | <https://opencode.ai/docs> |
| Лицензия | MIT |
| Проверено | `1.18.32` |
| Модули | все три дня |

### Установка

```bash
# macOS / Linux — официальный установщик, кладёт бинарь в ~/.opencode/bin
# и сам дописывает PATH в ~/.zshrc или ~/.bashrc
curl -fsSL https://opencode.ai/install | bash

# macOS / Linux — Homebrew (альтернатива)
brew install anomalyco/tap/opencode

# Windows
scoop install opencode
choco install opencode

# Любая ОС, если уже есть Node.js
npm i -g opencode-ai@latest
```

После установки **откройте новый терминал** — иначе PATH ещё старый.

```bash
opencode --version     # ожидаем номер версии, например 1.18.32
```

> Если раньше стоял OpenCode версии 0.x — снесите его перед установкой, конфиги несовместимы.

### Ключ OpenRouter

Ключ **персональный**, выдаётся заранее, лимит **5 USD на все три дня**. Ключами не
меняемся: по ним считается расход каждого.

Способ 1 — через сам OpenCode (ключ ляжет в `~/.local/share/opencode/auth.json`):

```
opencode          # запустить TUI
/connect          # выбрать OpenRouter, вставить ключ
```

Способ 2 — переменная окружения. Живёт **в окружении вашего ноутбука** — не в репозитории,
не в `.env`, не в промпте:

```bash
echo 'export OPENROUTER_API_KEY="<ваш ключ>"' >> ~/.zshrc && source ~/.zshrc
```

```powershell
# Windows PowerShell
setx OPENROUTER_API_KEY "<ваш ключ>"
```

Проверка — ключ целиком в терминал не печатаем:

```bash
echo "${OPENROUTER_API_KEY:0:7}…"      # должно быть непусто
```

**Личная подписка вместо ключа не подходит.** Примерно у половины участников есть ChatGPT
Plus / Claude Pro и т.п. Пользоваться ими никто не запрещает, но агент в IDE и все замеры
токенов идут через OpenRouter: в упражнениях 1.4, 2.5, 3.1, 3.33 сравниваются цифры из
OpenRouter → Activity, а подписка таких цифр не даёт. Ключ нужен всем.

### Выбор модели

Модели по умолчанию: **MiniMax M3** — генерация и частые прогоны, **GLM 5.3** — план, ревью,
судья. Ревьюер и судья работают на модели, отличной от модели автора.

Полные идентификаторы в OpenCode — `провайдер/модель`:

| Роль | ID в OpenCode |
|---|---|
| MiniMax M3 | `openrouter/minimax/minimax-m3` |
| GLM 5.3 | `openrouter/z-ai/glm-5.3` |

Выбрать на лету — команда `/models` в TUI. Зафиксировать для проекта — `opencode.json`
в корне репозитория:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "openrouter/minimax/minimax-m3"
}
```

> **`sol` на ключах не выдан.** Если карточка или промпт его упоминает — это опечатка.

### Запуск внутри IDE

**VS Code, Cursor, Windsurf, VSCodium.** Откройте встроенный терминал и наберите `opencode` —
расширение доустановится само. Дальше `Cmd+Esc` (macOS) / `Ctrl+Esc` (Windows, Linux)
открывает OpenCode в сплите, `Cmd+Shift+Esc` — новую сессию,
`Cmd+Option+K` / `Alt+Ctrl+K` вставляет ссылку на файл вида `@File#L37-42`.
Если расширение не поставилось — проверьте, что в PATH есть команда запуска редактора
(`code`, `cursor`, `windsurf`): `Cmd+Shift+P` → «Shell Command: Install 'code' command in PATH».

**JetBrains (PhpStorm, IntelliJ).** Расширения нет и не нужно: вкладка **Terminal** внизу,
`opencode`, работаем там.

### Неинтерактивный запуск (CI, модули 2.23 и 3.13)

```bash
opencode run "<промпт>" \
  --model openrouter/z-ai/glm-5.3 \
  --agent reviewer \
  --auto \
  --format json
```

| Флаг | Что делает |
|---|---|
| `--model` | модель в виде `провайдер/модель` |
| `--agent` | какого агента из `.opencode/agents/` использовать |
| `--auto` | автоматически подтверждать разрешения, кроме явных `deny` — обязателен в job, иначе процесс повиснет на вопросе |
| `--format json` | сырые события JSON вместо форматированного вывода — удобно парсить в пайплайне |
| `-f`, `--file` | приложить файл к сообщению (например, diff PR) |

В GitHub Actions ставим тем же установщиком, ключ — только из secrets:

```yaml
- name: Install OpenCode
  run: curl -fsSL https://opencode.ai/install | bash && echo "$HOME/.opencode/bin" >> $GITHUB_PATH
- name: AI review
  env:
    OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY_CI }}
  run: opencode run --auto --model openrouter/z-ai/glm-5.3 "$(cat prompt.txt)"
```

### Конфиги, которые понадобятся по ходу

Все — в корне репозитория, все идут в git (кроме ключей).

| Что | Где лежит | В каком модуле |
|---|---|---|
| Правила проекта | `AGENTS.md` в корне | 1.3, 2.3 |
| Права агента | секция `permission` в `opencode.json`: `"allow"` / `"ask"` / `"deny"` | ДЗ.6, 2.16, 2.20, 3.7, 3.26 |
| Субагенты | `.opencode/agents/<роль>.md` — markdown с YAML-фронтматтером | 2.12, 3.12, 3.14, 3.31 |
| Скиллы | `.opencode/skills/<имя>/SKILL.md` | 2.1, 2.3 |
| Команды (сохранённые промпты) | `.opencode/commands/<имя>.md`, вызов `/<имя>` | 1.12 ★★, ДЗ.3, ДЗ.5 |
| Git-хуки | `.githooks/`, подключение `git config core.hooksPath .githooks` | 1.17, 2.16, 3.7, 3.26 |
| Права человеческим языком | `docs/agent-rules.md` — читаемая копия блока `permission` | ДЗ.6 |
| MCP-серверы | секция `mcp` в `opencode.json` — см. [`mcp.md`](mcp.md) | 1.5 ★★ |

Формат файла субагента:

```markdown
---
description: Ревьюер кода по REVIEW.md
mode: subagent
model: openrouter/z-ai/glm-5.3
temperature: 0.1
permission:
  edit: deny
  bash: ask
---

Ты — ревьюер. Читаешь diff, не правишь код...
```

Имя файла = имя агента: `reviewer.md` → агент `reviewer`, вызывается `@reviewer` в сессии
или `--agent reviewer` в `opencode run`. Проверить, что подхватился: `opencode agent list`.

> Каталоги внутри `.opencode/` — во множественном числе: `agents/`, `skills/`, `commands/`,
> `plugins/`. Единственное число (`agent/`) тоже работает, это обратная совместимость.

Скиллы OpenCode ищет в шести местах, поэтому библиотека, написанная под Claude Code,
подхватывается без переделки:

```
.opencode/skills/<имя>/SKILL.md        ~/.config/opencode/skills/<имя>/SKILL.md
.claude/skills/<имя>/SKILL.md          ~/.claude/skills/<имя>/SKILL.md
.agents/skills/<имя>/SKILL.md          ~/.agents/skills/<имя>/SKILL.md
```

Во фронтматтере `SKILL.md` OpenCode читает только `name`, `description`, `license`,
`compatibility`, `metadata`. `name` — строчные латинские буквы и дефисы, совпадает с именем
папки. Остальные поля игнорируются молча.

---

## 3. Экономия контекста (упражнение 1.4)

Два инструмента с одной целью и разным механизмом: **Caveman** режет то, что агент
**пишет**, **RTK** — то, что агент **читает**.

### Caveman

| | |
|---|---|
| Репозиторий | <https://github.com/JuliusBrussee/caveman> |
| Документация | <https://docs.caveman.so/docs/quickstart> |
| Лицензия | MIT (скилл и CLI), BSL-1.1 (рантайм прокси) |
| Что делает | заставляет агента отвечать телеграфным стилем; код, команды, пути и тексты ошибок не трогает |

Внешний открытый проект, не наша разработка.

```bash
# Малый камень: только скилл. Работает в 30+ агентах, включая OpenCode.
npx skills add JuliusBrussee/caveman -g

# Большой камень: локальный прокси, режет ещё и вход агента
npm install -g @caveman-ai/cli && caveman setup --install
caveman opencode
```

Проверка: в сессии OpenCode наберите `/caveman` и задайте любой вопрос — ответ должен
стать заметно короче. Нужен Node.js 22.13+.

### RTK

| | |
|---|---|
| Репозиторий | <https://github.com/rtk-ai/rtk> |
| Сайт | <https://www.rtk-ai.app> |
| Лицензия | Apache-2.0 |
| Проверено | `0.49.0` |
| Что делает | прокси для shell: сжимает вывод `git`, `ls`, тестов, `grep`, `docker ps` до того, как он попадёт в контекст |

Внешний открытый проект, не наша разработка.

```bash
brew install rtk                    # macOS, Linux
winget install rtk-ai.rtk           # Windows
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh   # без Homebrew, ставит в ~/.local/bin
```

После установки — подключить к агенту и перезапустить его:

```bash
rtk init -g --opencode      # плагин для OpenCode
rtk init --show             # проверить, что подключился
```

```bash
rtk --version               # ожидаем номер версии
rtk gain                    # дашборд экономии — цифры для отчёта по 1.4
```

> **Осторожно с одноимённым пакетом.** На crates.io есть другой проект `rtk` (Rust Type Kit).
> Если `rtk gain` падает с «unknown command» — у вас не тот. Ставьте из Homebrew или
> `cargo install --git https://github.com/rtk-ai/rtk`.
>
> «LinuxRTK» — это тот же RTK, отдельного инструмента с таким названием нет.

### Чем мерить экономию

Два источника цифр, и они не взаимозаменяемы:

```bash
opencode stats --days 1 --models    # локальная статистика: токены и $ по моделям за сутки
opencode stats --project ""         # только текущий проект
```

**OpenRouter → Activity** — вкладка в личном кабинете: input, output и $ по каждому запросу,
с точностью до времени. Это источник истины по расходу ключа; `opencode stats` считает
по своим сессиям и не видит прогоны из CI.

Для отчётов по 1.4, 2.5, 3.1 и 3.33 берите Activity; `opencode stats` — чтобы быстро
свериться, не выходя из терминала.

---

## 4. Понадобится позже

### Understand Anything (упражнение 1.9)

Машинная карта репозитория: строит граф файлов, функций и зависимостей и отдаёт его
интерактивным дашбордом.

| | |
|---|---|
| Репозиторий | <https://github.com/Egonex-AI/Understand-Anything> |
| Сайт и демо | <https://understand-anything.com> |
| Лицензия | MIT |

```bash
# OpenCode (и Codex, Gemini CLI, Cursor, Copilot — платформа передаётся аргументом)
curl -fsSL https://raw.githubusercontent.com/Egonex-AI/Understand-Anything/main/install.sh | bash -s opencode
```

```powershell
# Windows
iwr -useb https://raw.githubusercontent.com/Egonex-AI/Understand-Anything/main/install.ps1 | iex
```

Установщик клонирует репозиторий в `~/.understand-anything/repo` и делает симлинки под
выбранную платформу. **После установки перезапустите агент.**

Дальше в сессии: `/understand` — построить граф (`.ua/knowledge-graph.json`),
`/understand-dashboard` — открыть дашборд, `/understand-explain <файл>` — разбор файла.
Если слеш-команды не распознаются, просто попросите словами: «используй скилл understand».

> Первый прогон `/understand` по всему репозиторию съедает заметно токенов. На `carmoney-lab`
> это терпимо (проект маленький), но не запускайте его «на всякий случай» дважды: повторные
> прогоны инкрементальные и дешёвые только если первый уже прошёл.

**В CI (день 3, упражнение 3.3 ★).** Отдельной headless-команды у Understand Anything нет —
это скилл внутри агента. Два рабочих варианта:

```bash
# 1. Хук после коммита: граф инкрементально досчитывается сам
/understand --auto-update

# 2. В job — через неинтерактивный запуск агента
opencode run --auto --model openrouter/minimax/minimax-m3 \
  "Используй скилл understand и обнови граф репозитория"
```

Граф лежит в `.ua/knowledge-graph.json` и коммитится. Если перевалит за 10 МБ —
переводите на git-lfs (`git lfs track ".ua/*.json"`).

### Библиотеки скиллов (ДЗ.1, упражнение 2.1)

| Библиотека | Репозиторий | Что даёт |
|---|---|---|
| **superpowers** | <https://github.com/obra/superpowers> | дисциплина разработки: брейншторм до работы, план, TDD, систематическая отладка, worktree, запрос ревью |
| **ai-native-sdlc-skills** | <https://github.com/asaf-shitrit/ai-native-sdlc-skills> | 11 неофициальных скиллов под playbook Anthropic: `sdlc-intent`, `sdlc-spec`, `sdlc-plan`, `sdlc-hooks`, `sdlc-pr-review` и др. |

Установка superpowers в OpenCode — попросите самого агента:

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
```

Для остальных библиотек: клонируйте репозиторий и скопируйте папки скиллов
в `.opencode/skills/` вашего проекта. Скиллы кладём **в git, к проекту**, а не глобально:
в 2.1 сравниваются прогоны с разными библиотеками, и глобальная установка их перемешает.

> **Осторожно с путём.** Каталог проекта — `.opencode/skills/`, ровно так, как его
> ищет OpenCode. Похожие, но другие имена (`.agent/`, `.agents/skill/`) он не
> сканирует: скилл, положенный туда, просто не подхватится и упражнение не сработает.
> Проверить список путей — в таблице выше.

> Внутренние библиотеки Hakku (`hakkuai_team_skills`) в практикуме **не используются** —
> это приватный репозиторий, доступа к нему у участников нет. Если увидите упоминание
> в чужой инструкции — пропускайте.

### gitleaks (упражнение 1.17)

Поиск секретов в diff, вызывается из `pre-commit`.

Репозиторий: <https://github.com/gitleaks/gitleaks>

```bash
brew install gitleaks                    # macOS, Linux — проверено 8.30.1
winget install --id Gitleaks.Gitleaks    # Windows
docker run --rm -v "$PWD:/path" zricethezav/gitleaks:latest detect --source /path   # без установки
```

```bash
gitleaks version        # ожидаем номер версии
```

### OWASP ZAP (упражнение 3.30 ★)

Baseline-скан развёрнутого сервиса: пассивные проверки, без атак. Ставить локально не нужно —
работает из Docker-образа.

| | |
|---|---|
| Репозиторий | <https://github.com/zaproxy/zaproxy> |
| Инструкция по baseline | <https://www.zaproxy.org/docs/docker/baseline-scan/> |
| Образ | `zaproxy/zap-stable` |

```bash
docker run --rm -t zaproxy/zap-stable zap-baseline.py -t http://<host>:<port>
```

В GitHub Actions удобнее готовым экшеном — <https://github.com/zaproxy/action-baseline>.
По умолчанию он заводит issue с найденными алертами и **не** роняет job; чтобы ронял —
`fail_action: true`.

### Линтер стиля PHP (упражнение 1.17 ★)

Ставится **в проект**, а не глобально — тогда у всей команды одна версия:

```bash
composer require --dev squizlabs/php_codesniffer
./vendor/bin/phpcs --version        # ожидаем: PHP_CodeSniffer version x.y.z
```

Альтернатива — `friendsofphp/php-cs-fixer`, ставится так же.

---

## Проверка готовности

Прогоните целиком до занятия. Всё должно отработать без единой ошибки:

```bash
git --version
gh --version && gh auth status
docker compose version                  # пропускаем, если Docker не ставили
php -v && composer -V
opencode --version
echo "${OPENROUTER_API_KEY:0:7}…"       # непусто; ключ целиком не печатаем
```

И один живой прогон — он проверяет сразу и установку, и ключ, и доступ к модели:

```bash
opencode run --model openrouter/minimax/minimax-m3 "Ответь одним словом: работает"
```

---

## Если не ставится

Правило занятия: **на установку не тратим больше 10 минут**. Дальше — обходной путь,
а разбираемся в перерыве. Ни одно упражнение не заблокировано полностью.

| Симптом | Что это | Что делать прямо сейчас |
|---|---|---|
| `opencode: command not found` сразу после установки | PATH ещё старый | Открыть **новый** терминал. Не помогло — `export PATH="$HOME/.opencode/bin:$PATH"` |
| `opencode` ставится, но в IDE расширения нет | нет команды запуска редактора в PATH | Работать во вкладке Terminal как есть — на упражнения это не влияет |
| Модель отвечает `401` / `invalid api key` | ключ не подхватился | `echo "${OPENROUTER_API_KEY:0:7}…"` — пусто? переменная не в том шелле. Не пусто? ключ мог быть скопирован с пробелом на конце |
| Модель отвечает `402` / `insufficient credits` | упёрлись в лимит 5 USD | В чат «Помощь», параллельно продолжайте на MiniMax M3 — она дешевле |
| `docker compose` не работает | Docker не ставится на корпоративной машине | `composer install && make test` работает без Docker; сервис поднимете на стенде (1.18) |
| Corporate proxy / SSL-ошибки при `curl ... \| bash` | инспекция трафика | Скачать бинарь со страницы releases руками и положить в PATH: [OpenCode](https://github.com/anomalyco/opencode/releases), [RTK](https://github.com/rtk-ai/rtk/releases) |
| `npm i -g` падает на правах | нет прав на глобальную папку npm | Не воевать: у OpenCode и RTK есть установщики в домашнюю папку (см. выше) |
| Caveman / RTK / Understand Anything не встали | это ускорители, не фундамент | Пропустить. 1.4 и 1.9 делаются и без них — просто цифры экономии будут чужие, с экрана ведущего |
| Ничего из перечисленного | — | Чат «Помощь», строкой: что делали, что выдал терминал. Скриншот терминала лучше пересказа |

Что **нельзя** обойти и без чего упражнения встанут: Git, `gh` с выполненным `gh auth login`,
OpenCode и рабочий ключ OpenRouter. Остальное — опционально.
