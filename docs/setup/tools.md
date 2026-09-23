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

## 2. Агент: Kilo Code

Единственный агент практикума. Живёт панелью внутри вашей IDE — отдельный интерфейс
осваивать не нужно. Для CI и терминала у него есть CLI `kilo` с теми же конфигами.

| | |
|---|---|
| Сайт и документация | <https://kilo.ai/docs> |
| Расширение VS Code | `kilocode.Kilo-Code` в маркетплейсе |
| CLI | npm-пакет `@kilocode/cli`, команда `kilo` |
| Проверено | расширение и CLI `7.7.9` |

### Установка

**VS Code, Cursor, Windsurf, VSCodium.** Панель расширений (`Cmd+Shift+X` / `Ctrl+Shift+X`)
→ «Kilo Code», издатель `kilocode` → Install. Иконка Kilo появится на боковой панели.

**JetBrains (PhpStorm, IntelliJ).** Settings → Plugins → Marketplace → «Kilo Code» →
Install → перезапустить IDE.

**CLI** (понадобится в днях 2–3 для CI, в классе не обязателен):

```bash
npm i -g @kilocode/cli
kilo --version        # ожидаем номер версии, например 7.7.9
```

### Ключ OpenRouter

Ключ **персональный**, выдаётся заранее, лимит **5 USD на все три дня**. Ключами не
меняемся: по ним считается расход каждого.

В расширении: панель Kilo → шестерёнка Settings → вкладка **Providers** → добавить
OpenRouter → вставить ключ. В CLI: `kilo auth login` → OpenRouter → ключ.

Ключ живёт **в настройках агента или в окружении вашего ноутбука** — не в репозитории,
не в `.env`, не в `kilo.jsonc`, не в промпте. Для CLI и CI — переменная окружения:

```bash
echo 'export OPENROUTER_API_KEY="<ваш ключ>"' >> ~/.zshrc && source ~/.zshrc
```

```powershell
# Windows PowerShell
setx OPENROUTER_API_KEY "<ваш ключ>"
```

Проверка ключа и остатка лимита (ключ целиком в терминал не печатаем):

```bash
curl -s https://openrouter.ai/api/v1/key -H "Authorization: Bearer $OPENROUTER_API_KEY"
# в ответе limit и limit_remaining
```

**Личная подписка вместо ключа не подходит.** Пользоваться ChatGPT Plus / Claude Pro никто
не запрещает, но агент в IDE и все замеры токенов идут через OpenRouter: цифры берутся из
OpenRouter → Activity, а подписка таких цифр не даёт. Ключ нужен всем.

### Выбор модели

Модели по умолчанию: **MiniMax M3** — генерация и частые прогоны, **GLM 5.3** — карта кода,
план, ревью, судья. Ревьюер и судья работают на модели, отличной от модели автора.

| Роль | ID в Kilo |
|---|---|
| MiniMax M3 | `openrouter/minimax/minimax-m3` |
| GLM 5.3 | `openrouter/z-ai/glm-5.3` |

Выбрать на лету — выпадающий список моделей в панели Kilo. Модель по умолчанию для
проекта уже задана в [`kilo.jsonc`](../../kilo.jsonc) в корне репозитория.

> **`sol` на ключах не выдан.** Если карточка или промпт его упоминает — это опечатка.

### Агенты вместо режимов

В Kilo режимы называются **агентами**. Переключатель — в панели Kilo или `Cmd+.` / `Ctrl+.`.

| Агент | Что может |
|---|---|
| `ask` | только читать и отвечать |
| `code` | читать и править файлы, запускать команды |
| `plan` | план в `.kilo/plans/`, код не трогает |
| `debug` | как `code`, с упором на поиск причины |

Свои агенты — файлы `.kilo/agents/<имя>.md`, см. ниже. **Agent Manager**
(`Cmd+Shift+M` / `Ctrl+Shift+M`) запускает сессии параллельно, каждую в своём git worktree
в `.kilo/worktrees/`.

### Неинтерактивный запуск (CI, дни 2–3)

```bash
kilo run "<промпт>" \
  --model openrouter/z-ai/glm-5.3 \
  --agent reviewer \
  --auto \
  --format json
```

| Флаг | Что делает |
|---|---|
| `-m`, `--model` | модель в виде `провайдер/модель` |
| `--agent` | какого агента из `.kilo/agents/` использовать |
| `--auto` | автоматически подтверждать разрешения, кроме явных `deny` — обязателен в job, иначе процесс повиснет на вопросе |
| `--format json` | сырые события JSON вместо форматированного вывода — удобно парсить в пайплайне |
| `-f`, `--file` | приложить файл к сообщению (например, diff PR) |

В GitHub Actions ключ — только из secrets:

```yaml
- name: Install Kilo CLI
  run: npm i -g @kilocode/cli
- name: AI review
  env:
    OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY_CI }}
  run: kilo run --auto --model openrouter/z-ai/glm-5.3 "$(cat prompt.txt)"
```

### Конфиги, которые понадобятся по ходу

Все — в корне репозитория, все идут в git (кроме ключей).

| Что | Где лежит | В каком модуле |
|---|---|---|
| Правила проекта | `AGENTS.md` в корне — Kilo подгружает его в каждую сессию сам | 1.8, 2.3 |
| Права агента | секция `permission` в `kilo.jsonc`: `"allow"` / `"ask"` / `"deny"`; после правки — Reload Window | 1.9, 2.16, 2.20, 3.7, 3.26 |
| Свои агенты и субагенты | `.kilo/agents/<имя>.md` — markdown с YAML-фронтматтером | 1.12, 1.13, 2.12, 3.12, 3.14, 3.31 |
| Скиллы | `.kilo/skills/<имя>/SKILL.md` | 2.1, 2.3 |
| Команды (сохранённые промпты) | `.kilo/commands/<имя>.md`, вызов `/<имя>` | дни 2–3 |
| Git-хуки | `.githooks/`, подключение `git config core.hooksPath .githooks` | 2.16, 3.7, 3.26 |
| Права человеческим языком | `docs/agent-rules.md` — читаемая копия блока `permission` | 1.9 ★, 2.20 |
| MCP-серверы | секция `mcp` в `kilo.jsonc` — см. [`mcp.md`](mcp.md) | 1.19 |

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
или `--agent reviewer` в `kilo run`. Проверить, что подхватился: `kilo agent list`.

> Каталоги внутри `.kilo/` — во множественном числе: `agents/`, `skills/`, `commands/`.
> Старые пути `.kilocode/` и `.kilocodemodes` Kilo при запуске переносит сам.

Скиллы Kilo ищет в шести местах, поэтому библиотека, написанная под Claude Code,
подхватывается без переделки:

```
.kilo/skills/<имя>/SKILL.md            ~/.config/kilo/skills/<имя>/SKILL.md
.claude/skills/<имя>/SKILL.md          ~/.claude/skills/<имя>/SKILL.md
.agents/skills/<имя>/SKILL.md          ~/.agents/skills/<имя>/SKILL.md
```

Во фронтматтере `SKILL.md` Kilo читает только `name`, `description`, `license`,
`compatibility`, `metadata`. `name` — строчные латинские буквы и дефисы, совпадает с именем
папки. Остальные поля игнорируются молча. Какие скиллы видит агент — `kilo debug skill`.

---

## 3. Экономия контекста (упражнения 1.17–1.18)

Три инструмента с одной целью и разным механизмом: **Caveman** режет то, что агент
**пишет**, **ast-index** — сколько кода агент **читает**, **RTK** — вывод команд, который
попадает в контекст. В классе ставим первые два (1.17–1.18), RTK — по желанию.

### Caveman

| | |
|---|---|
| Репозиторий | <https://github.com/JuliusBrussee/caveman> |
| Документация | <https://docs.caveman.so/docs/quickstart> |
| Лицензия | MIT (скилл и CLI), BSL-1.1 (рантайм прокси) |
| Что делает | заставляет агента отвечать телеграфным стилем; код, команды, пути и тексты ошибок не трогает |

Внешний открытый проект, не наша разработка.

```bash
# Малый камень: только скилл (так ставим в 1.18)
npx skills add JuliusBrussee/caveman -a kilo -g -y

# Большой камень: локальный прокси, режет ещё и вход агента
npm install -g @caveman-ai/cli && caveman setup --install
caveman kilo              # проверено автором только для Kilo CLI, не для расширения
```

Проверка: `kilo debug skill` показывает `caveman`; в сессии Kilo наберите `/caveman` и задайте любой вопрос — ответ должен
стать заметно короче. Нужен Node.js 22.13+.

### ast-index

| | |
|---|---|
| Репозиторий | <https://github.com/defendend/Claude-ast-index-search> |
| Что делает | структурный индекс кода: агент ищет по классам, функциям и вызовам, а не читает файлы целиком |

```bash
brew tap defendend/ast-index && brew install ast-index   # macOS, Linux
winget install --id defendend.ast-index                  # Windows
ast-index rebuild                                        # в корне проекта, один раз
ast-index search mileage                                 # проверка
```

Агенту достаточно строки в `AGENTS.md`: «Поиск по коду — через ast-index (search, class,
symbol, usages, callers), а не чтением файлов целиком».

### RTK (по желанию)

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
rtk init --agent kilocode   # подключение к Kilo Code, в корне проекта
rtk init --show             # проверить, что подключился
```

```bash
rtk --version               # ожидаем номер версии
rtk gain                    # дашборд экономии
```

> **Осторожно с одноимённым пакетом.** На crates.io есть другой проект `rtk` (Rust Type Kit).
> Если `rtk gain` падает с «unknown command» — у вас не тот. Ставьте из Homebrew или
> `cargo install --git https://github.com/rtk-ai/rtk`.
>
> «LinuxRTK» — это тот же RTK, отдельного инструмента с таким названием нет.

### Чем мерить экономию

Два источника цифр, и они не взаимозаменяемы:

```bash
kilo stats --days 1 --models    # локальная статистика: токены и $ по моделям за сутки
kilo stats --project ""         # только текущий проект
```

**OpenRouter → Activity** — вкладка в личном кабинете: input, output и $ по каждому запросу,
с точностью до времени. Это источник истины по расходу ключа; `kilo stats` считает
по своим сессиям и не видит прогоны из CI.

Для отчётов по 1.17–1.18, 2.5, 3.1 и 3.33 берите Activity; `kilo stats` — чтобы быстро
свериться, не выходя из терминала.

---

## 4. Понадобится позже

### Understand Anything (дни 2–3)

Машинная карта репозитория: строит граф файлов, функций и зависимостей и отдаёт его
интерактивным дашбордом.

| | |
|---|---|
| Репозиторий | <https://github.com/Egonex-AI/Understand-Anything> |
| Сайт и демо | <https://understand-anything.com> |
| Лицензия | MIT |

```bash
# Для Kilo берём платформу opencode: скиллы лягут в ~/.agents/skills, их Kilo читает
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
kilo run --auto --model openrouter/minimax/minimax-m3 \
  "Используй скилл understand и обнови граф репозитория"
```

Граф лежит в `.ua/knowledge-graph.json` и коммитится. Если перевалит за 10 МБ —
переводите на git-lfs (`git lfs track ".ua/*.json"`).

### Библиотеки скиллов (упражнение 2.1)

| Библиотека | Репозиторий | Что даёт |
|---|---|---|
| **superpowers** | <https://github.com/obra/superpowers> | дисциплина разработки: брейншторм до работы, план, TDD, систематическая отладка, worktree, запрос ревью |
| **ai-native-sdlc-skills** | <https://github.com/asaf-shitrit/ai-native-sdlc-skills> | 11 неофициальных скиллов под playbook Anthropic: `sdlc-intent`, `sdlc-spec`, `sdlc-plan`, `sdlc-hooks`, `sdlc-pr-review` и др. |

Установка любой библиотеки — тем же установщиком, что Caveman, в проект:

```bash
npx skills add obra/superpowers -a kilo -y
```

Если установщик библиотеку не берёт — клонируйте репозиторий и скопируйте папки скиллов
в `.kilo/skills/` вашего проекта. Скиллы кладём **в git, к проекту**, а не глобально:
в 2.1 сравниваются прогоны с разными библиотеками, и глобальная установка их перемешает.

> **Осторожно с путём.** Каталог проекта — `.kilo/skills/`, ровно так, как его
> ищет Kilo. Похожие, но другие имена (`.agent/`, `.agents/skill/`) он не
> сканирует: скилл, положенный туда, просто не подхватится и упражнение не сработает.
> Проверить список путей — в таблице выше.

> Внутренние библиотеки Hakku (`hakkuai_team_skills`) в практикуме **не используются** —
> это приватный репозиторий, доступа к нему у участников нет. Если увидите упоминание
> в чужой инструкции — пропускайте.

### gitleaks (дни 2–3)

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

### Линтер стиля PHP (дни 2–3)

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
kilo --version                          # если ставили CLI
echo "${OPENROUTER_API_KEY:0:7}…"       # непусто; ключ целиком не печатаем
```

И один живой прогон — он проверяет сразу и установку, и ключ, и доступ к модели:

```bash
curl -s https://openrouter.ai/api/v1/key -H "Authorization: Bearer $OPENROUTER_API_KEY"   # limit 5
```

А в самом Kilo — упражнение 1.4: первый запрос агенту в своём проекте.

---

## Если не ставится

Правило занятия: **на установку не тратим больше 10 минут**. Дальше — обходной путь,
а разбираемся в перерыве. Ни одно упражнение не заблокировано полностью.

| Симптом | Что это | Что делать прямо сейчас |
|---|---|---|
| Панели Kilo нет после установки | расширение не активировалось | Command Palette → «Developer: Reload Window»; не помогло — переустановить расширение |
| Kilo не видит правку `kilo.jsonc` или нового агента | проектный конфиг кэшируется | «Developer: Reload Window» |
| Модель отвечает `401` / `invalid api key` | ключ не подхватился | `echo "${OPENROUTER_API_KEY:0:7}…"` — пусто? переменная не в том шелле. Не пусто? ключ мог быть скопирован с пробелом на конце |
| Модель отвечает `402` / `insufficient credits` | упёрлись в лимит 5 USD | В чат «Помощь», параллельно продолжайте на MiniMax M3 — она дешевле |
| `docker compose` не работает | Docker не ставится на корпоративной машине | `composer install && make test` работает без Docker; для 1.19 ведущий даст адрес сервиса на стенде |
| Corporate proxy / SSL-ошибки при `curl ... \| bash` | инспекция трафика | Скачать бинарь со страницы releases руками и положить в PATH: [RTK](https://github.com/rtk-ai/rtk/releases) |
| `npm i -g` падает на правах | нет прав на глобальную папку npm | Расширение Kilo ставится из маркетплейса IDE без npm. Для CLI и RTK — установщики в домашнюю папку или `npm config set prefix ~/.npm-global` |
| Caveman / ast-index / RTK не встали | это ускорители, не фундамент | Пропустить. 1.17–1.18 тогда смотрите с экрана ведущего и доделайте дома |
| Ничего из перечисленного | — | Чат «Помощь», строкой: что делали, что выдал терминал. Скриншот терминала лучше пересказа |

Что **нельзя** обойти и без чего упражнения встанут: Git, `gh` с выполненным `gh auth login`,
Kilo и рабочий ключ OpenRouter. Остальное — опционально.
