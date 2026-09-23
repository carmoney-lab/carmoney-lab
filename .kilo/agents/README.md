# .kilo/agents/

Субагент — отдельная роль со своей моделью, своим промптом и своими правами.
Один файл `<роль>.md` = один агент. Имя файла и есть имя агента: `reviewer.md` →
`@reviewer` в сессии или `--agent reviewer` в `kilo run`.

Формат — markdown с YAML-фронтматтером:

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

Поля фронтматтера: `description`, `mode` (`primary` — виден в переключателе агентов, `subagent` — вызывается через `@имя` / `all`), `model`,
`temperature`, `permission`. Права в файле агента перекрывают общие из
[`kilo.jsonc`](../../kilo.jsonc) — ревьюеру, например, запрещают `edit`.

Проверить, что агент подхватился:

```bash
kilo agent list
```

Кто здесь появится по ходу практикума:

| Файл | Роль | Где |
|---|---|---|
| `planner.md` | свой агент: план в `docs/plan/`, код не трогает | 1.12 |
| `scout.md` | субагент-разведчик: ищет по коду, ничего не правит | 1.13 |
| `reviewer.md` | ревью diff по `REVIEW.md` | 2.12 |
| `test-writer.md` | пишет тесты, существующие не трогает | ДЗ 2.4 ★ |
| `orchestrator.md` | раздаёт issues и собирает PR | 3.12 |
| `implementer.md` | правит код в своём worktree | 3.14 |
| `verifier.md` | проверяет AC с чистого контекста | 3.14 |
| `oncall.md` | детектор инцидентов по метрикам стенда | 3.31 |
