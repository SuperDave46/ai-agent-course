# Домашнє завдання 1. Налаштувати агента під свій проєкт

> Курс «Агентна розробка з ШІ» · до лекції 1 «Агент — це не чат»
> **Термін:** до початку лекції 2
> **Інструмент:** будь-який агентний (Claude Code, Cursor, GitHub Copilot, Codex, Gemini CLI, Z.ai ZCode). Приклади нижче наведено для Claude Code, відповідники — у розділі «Якщо у вас інший інструмент».

## Мета

Після цього завдання агент у вашому проєкті:
- знає, як збирати, тестувати й лінтити код, і не вгадує щоразу;
- має один skill для задачі, яку ви робите регулярно.

Також ви отримаєте **базову лінію витрат**: з нею порівнюватимете результати в ДЗ 2 і у фінальному завданні.

## Що зробити

### Крок 1. Файл постійних інструкцій

Створіть `CLAUDE.md` (або `AGENTS.md`) у корені репозиторію.

**Писати:**
- команди збірки, тестів і лінтера — найважливіше, бо це спосіб для агента перевірити себе;
- конвенції, яких не видно з коду;
- неочевидні пастки (gotchas).

**Не писати:**
- опис архітектури і структури тек — агент прочитає це сам;
- самоочевидні правила на кшталт «пиши чистий код»;
- інструкції для рідкісних задач — їм місце в skills;
- КАПСЛОК, «IMPORTANT», «!!!».

**Орієнтир:** до 200 рядків. Добрий файл для невеликого проєкту часто вміщується в 20–60.

Шаблон для старту:

```markdown
# Commands
- Install: <command>
- Build: <command>
- Test (all): <command>
- Test (single file): <command>
- Lint / format: <command>

# Conventions
- <convention that is not obvious from the code>

# Gotchas
- <something that surprised you or a new teammate>
```

> Якщо згенерували файл через `/init`, сприймайте результат як чернетку: видаліть усе, що агент і так бачить у коді.

### Крок 2. Один skill

Оберіть задачу, яку ви вже кілька разів пояснювали агенту або колезі, наприклад:
- створити новий endpoint, екран або модуль за шаблоном команди;
- додати міграцію бази даних;
- підготувати реліз (changelog, версія, тег).

Створіть теку `.claude/skills/<назва>/` з файлом `SKILL.md`:

```markdown
---
name: new-endpoint
description: Create a REST endpoint following team conventions. Use when adding a new API route.
---

## Steps
1. ...
2. ...

## Checklist
- [ ] Tests added
- [ ] ...
```

**Найважливіше — поле `description`.** Агент обирає skill саме за ним, тому там має бути сказано, *що* робить skill і *коли* його застосовувати. Розмитий опис означає, що skill не спрацює автоматично.

Перевірте, що skill працює: дайте агенту відповідну задачу без явної згадки skill або викличте його через `/назва`.

### Крок 3. Базова лінія

1. Оберіть **типову задачу**, яку легко повторити в ДЗ 2. Хороші приклади: невеликий багфікс, додавання валідації, новий тест для наявної функції.
2. Збережіть точний текст промпту. Він знадобиться для повторення.
3. Почніть з чистої сесії (`/clear` або нова сесія).
4. Виконайте задачу до кінця.
5. Запишіть цифри з `/usage` (або аналога у вашому інструменті), кількість кроків агента і час виконання.

## Що здати

1. Посилання на гілку або PR з файлом інструкцій і skill.
2. Короткий звіт (до 10 рядків) за шаблоном:

```markdown
## ДЗ 1 — <ім'я>

**Інструмент:** <Claude Code / Cursor / Copilot / Codex / Gemini CLI / ZCode>
**Файл інструкцій:** <назва>, <N> рядків
**Skill:** <назва> — <що робить, одним реченням>. Спрацював автоматично: так / ні

**Базова лінія**
- Задача: <одне речення>
- Промпт: <точний текст>
- Витрати: <токени / запити / % ліміту>
- Кроків агента: <N>
- Час: <хв>

**Що здивувало:** <одне речення>
```

## Критерії оцінювання

| Зараховано | Добре | Відмінно |
|---|---|---|
| Файл інструкцій і skill існують | Файл коротший за 200 рядків, без очевидних речей і опису структури | Є базова лінія, skill реально спрацював на задачі |

## Якщо у вас інший інструмент

| | Файл інструкцій | Skills | Вимірювання |
|---|---|---|---|
| **Claude Code** | `CLAUDE.md` (можна імпортувати `@AGENTS.md`) | `.claude/skills/` | `/usage`, `/context` |
| **Cursor** | `.cursor/rules/`, `AGENTS.md` | Agent Skills | Usage dashboard |
| **GitHub Copilot** | `.github/copilot-instructions.md`, `AGENTS.md` | `.github/skills/` | Premium requests у налаштуваннях |
| **OpenAI Codex** | `AGENTS.md` | `.agents/skills/` | Статус сесії (перевірте у своїй версії) |
| **Gemini CLI** | `GEMINI.md` (або `AGENTS.md` через налаштування) | Skills | `/stats` |
| **Z.ai ZCode** | `AGENTS.md` (вміст `CLAUDE.md` копіюється під час онбордингу) | Skills (`SKILL.md`) | Лічильник токенів у панелі прогресу, квота GLM Coding Plan |

Формат `SKILL.md` — відкритий стандарт, тож skill з цього завдання зазвичай працює в різних інструментах без змін. Якщо ваш інструмент показує не токени, а запити або відсоток ліміту, записуйте ці одиниці.

**Z.ai:** GLM Coding Plan працює не лише в ZCode, а й у Claude Code через Anthropic-сумісний endpoint. У такому разі виконуйте завдання за інструкціями для Claude Code, а витрати звіряйте з квотою в кабінеті Z.ai.

## Поради

- Агент ігнорує правило з файлу інструкцій? Спершу скоротіть файл: у довгому файлі правила губляться.
- Не пишіть skill «на всі випадки». Один вузький робочий skill кращий за три загальні.
- Для базової лінії оберіть задачу на 5–15 хвилин роботи агента. Занадто дрібна дасть шумні цифри, занадто велика буде важкою для повторення.

## Додаткові матеріали

**Основне:**
- [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) — як писати інструкції, що таке детерміновані правила.
- [Extend Claude Code (features overview)](https://code.claude.com/docs/en/features-overview) — порівняння CLAUDE.md, skills, hooks і субагентів.
- [CLAUDE.md / memory](https://code.claude.com/docs/en/memory)
- [Skills](https://code.claude.com/docs/en/skills)

**Щоб глибше зрозуміти тему:**
- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — що таке агент і коли він потрібен.
- [Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) — як влаштовані skills і progressive disclosure.
- [How Claude Code works](https://code.claude.com/docs/en/how-claude-code-works) — агентний цикл зсередини.
- [Permission modes](https://code.claude.com/docs/en/permission-modes) — режими дозволів і plan mode.

**Стандарти та інші інструменти:**
- [AGENTS.md](https://agents.md) — відкритий формат файлу інструкцій.
- [Agent Skills specification](https://agentskills.io)
- [GitHub Copilot: customization cheat sheet](https://docs.github.com/en/copilot/reference/customization-cheat-sheet)
- [OpenAI Codex: customization](https://developers.openai.com/codex/concepts/customization)
- [Cursor docs](https://cursor.com/docs)
- [Gemini CLI docs](https://geminicli.com/docs/hooks/)
- [ZCode docs](https://zcode.z.ai/en/docs) і [GLM Coding Plan: підтримувані інструменти](https://docs.z.ai/devpack/overview)
