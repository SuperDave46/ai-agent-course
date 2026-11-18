# Домашнє завдання 4. Quality gates: hooks, захист тестів, evals

> Курс «Агентна розробка з ШІ» · до лекції 4 «Quality Gates»
> **Термін:** до початку лекції 5
> **Потрібно з ДЗ 1:** ваш skill
> **Інструмент:** будь-який агентний. Приклади наведено для Claude Code, відповідники — у розділі «Якщо у вас інший інструмент».

## Мета

Перетворити «прохання» в інструкціях на **гарантії** в конфігурації і навчитися вимірювати якість свого skill, а не оцінювати її «на око».

## Що зробити

### Частина 1. Hook-перевірка

Налаштуйте щонайменше один hook:

- **Stop hook** з тестами або перевіркою типів — агент не зможе завершити роботу, доки перевірка не пройде;
- або **`PostToolUse` hook** з форматером чи лінтером після кожного редагування.

Приклад для Claude Code (`.claude/settings.json`), Python-проєкт:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          { "type": "command", "command": "uv run pytest -q 1>&2 || exit 2" }
        ]
      }
    ]
  }
}
```

- `exit 2` блокує завершення, а вивід у stderr повертається агенту як причина.
- Повний набір тестів запускайте у `Stop`, швидкі перевірки — у `PostToolUse`, інакше робота сповільниться.
- Перевірте, що hook спрацьовує: дайте агенту задачу, яка ламає тест.

### Частина 2. Захист тестів

Агент, у якого падає тест, іноді «виправляє» сам тест. Заборонимо це на рівні конфігурації:

- правило deny на редагування теки з тестами під час реалізації;
- або `PreToolUse` hook, який блокує редагування файлів у `tests/`.

Перевірте: попросіть агента реалізувати щось, що ламає тест, і подивіться, чи він спробує змінити тест і чи спрацює заборона.

### Частина 3. Evals для вашого skill

1. Візьміть skill з ДЗ 1.
2. Сформулюйте 3–5 типових задач, для яких він створений.
3. Для кожної задачі напишіть **перевірку кодом**: тести проходять, файл існує, лінтер чистий, структура відповідає шаблону.
4. Запустіть кожну задачу **2–3 рази** в headless-режимі в чистій копії репозиторію.
5. Запишіть, скільки запусків пройшло.
6. Змініть skill, щоб виправити знайдену проблему, і запустіть evals ще раз.

Мінімальний скрипт (bash, Claude Code):

```bash
#!/usr/bin/env bash
# evals/run.sh — runs each task N times and counts passes.
# Run it in a separate clone: it resets the working tree before every run.
set -u
N=3
for task in evals/tasks/*.txt; do
  pass=0
  for i in $(seq "$N"); do
    git reset -q --hard && git clean -fdq -e evals/
    claude -p "$(cat "$task")" --permission-mode acceptEdits --max-turns 30 > /dev/null
    if bash "${task%.txt}.check.sh"; then pass=$((pass+1)); fi
  done
  echo "$(basename "$task"): $pass/$N"
done
git reset -q --hard
```

> ⚠️ Скрипт скидає всі незакомічені зміни. Запускайте його в окремій копії репозиторію (`git clone` або `git worktree add`), а теку `evals/` закомітьте заздалегідь.

Де `evals/tasks/01-new-endpoint.txt` — промпт задачі, а `evals/tasks/01-new-endpoint.check.sh` — перевірка, що повертає код 0 при успіху.

> Кожен запуск — повноцінна сесія агента, тож обирайте невеликі задачі.

## Що здати

1. PR з конфігурацією hooks, захистом тестів і теками `evals/`.
2. Звіт за шаблоном:

```markdown
## ДЗ 4 — <ім'я>

**Інструмент:** <назва>
**Hooks:** <які події і що перевіряють>
**Захист тестів:** <як зроблено> · Агент намагався змінити тест: так / ні

| Eval | До зміни skill | Після |
|---|---|---|
| 01 <назва> | 2/3 | 3/3 |
| ... | | |

**Що виправив у skill:** <одне-два речення>
```

## Критерії оцінювання

| Зараховано | Добре | Відмінно |
|---|---|---|
| Hook працює | Evals відтворювані, є кілька запусків | Evals знайшли проблему у skill, і її виправлено |

## Якщо у вас інший інструмент

| | Hooks | Headless для evals |
|---|---|---|
| **Claude Code** | `.claude/settings.json`: `PreToolUse`, `PostToolUse`, `Stop` | `claude -p` |
| **Cursor** | `.cursor/hooks.json`: `preToolUse`, `postToolUse`, `stop` | Cursor CLI |
| **GitHub Copilot** | `.github/hooks/*.json` (cloud agent і CLI; в IDE частково) | Copilot CLI |
| **OpenAI Codex** | Hooks (перевірте у своїй версії) | `codex exec` |
| **Gemini CLI** | `BeforeTool`, `AfterTool` | `gemini -p` |
| **Z.ai ZCode** | Hooks у складі плагінів (перевірте у своїй версії) | Agent CLI `zcode` (перевірте) |

**Якщо hooks немає або вони працюють лише в CLI**, а ви працюєте в IDE: зробіть git pre-commit hook і перевірку в CI. Вони спрацьовують пізніше (на коміті), але так само детерміновані. Для evals можна запускати задачі вручну в нових сесіях, головне — кілька запусків і перевірка кодом.

## Поради

- Hook виконується з вашими правами. Рев'юйте hooks, особливо з чужих репозиторіїв і плагінів.
- Нестабільний тест + Stop hook = агент у циклі. Спершу полагодьте flaky-тести.
- Перевірки кодом надійніші й дешевші за оцінку іншою моделлю (LLM-as-judge). Використовуйте модель лише там, де кодом не перевірити.

## Додаткові матеріали

**Основне:**
- [Hooks guide](https://code.claude.com/docs/en/hooks-guide) і [Hooks reference](https://code.claude.com/docs/en/hooks)
- [Best practices: Give Claude a check to run, Enforce the rules](https://code.claude.com/docs/en/best-practices)
- [Permissions](https://code.claude.com/docs/en/permissions) — правила allow / deny.
- [Headless mode (`claude -p`)](https://code.claude.com/docs/en/headless)

**Щоб глибше зрозуміти тему:**
- [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- [Subagents: обмеження інструментів](https://code.claude.com/docs/en/sub-agents) — рев'юер, який фізично не може змінити код.
- [Manage costs: Offload processing to hooks](https://code.claude.com/docs/en/costs) — hook, що фільтрує вивід тестів.

**Інші інструменти:**
- [GitHub Copilot: comparing customizations (hooks)](https://docs.github.com/en/copilot/concepts/customizations/comparing-customizations)
- [Gemini CLI hooks](https://geminicli.com/docs/hooks/)
- [Cursor docs (Hooks)](https://cursor.com/docs)
- [OpenAI Codex: customization](https://developers.openai.com/codex/concepts/customization)
- [ZCode docs](https://zcode.z.ai/en/docs) — плагіни та Goal Mode.
