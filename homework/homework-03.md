# Домашнє завдання 3. Фіча через Spec-Driven Development

> Курс «Агентна розробка з ШІ» · до лекції 3 «Spec-Driven Development»
> **Термін:** до початку лекції 4
> **Інструмент:** будь-який агентний. Приклади наведено для Claude Code, відповідники — у розділі «Якщо у вас інший інструмент».

## Мета

Провести одну реальну задачу повним циклом **spec → plan → implement → verify** і побачити, де специфікація заощаджує ітерації, а де план «пливе».

## Що зробити

### Крок 1. Оберіть задачу

Реальна фіча або багфікс **середнього розміру**: зміни щонайменше у 2 файлах, нова поведінка або зміна публічного API. Занадто дрібна задача не покаже користі специфікації, занадто велика не вміститься в тиждень.

### Крок 2. Специфікація через інтерв'ю

Нехай агент проінтерв'ює вас і запише результат у `SPEC.md`:

```
I want to <short description of the feature>. Interview me about requirements,
edge cases and constraints one question at a time. When done, write SPEC.md
with problem, acceptance criteria, constraints, non-goals and edge cases.
```

Відредагуйте результат. Специфікація ваша, агент лише допомагає її сформулювати.

Шаблон `SPEC.md`:

```markdown
# SPEC: <feature name>

## Problem
<why this is needed, 1–3 sentences>

## Acceptance criteria
- <verifiable condition, ideally one test per line>
- ...

## Constraints
- <technical limits: no new dependencies, existing storage, API compatibility…>

## Non-goals
- <what we intentionally do NOT do>

## Edge cases
- <boundary behaviour and how to handle it>
```

**Перевірка якості:** кожен рядок acceptance criteria має перетворюватися на тест або команду. «Має працювати швидко» — не критерій, «p95 < 200 мс на тестовому наборі» — критерій.

### Крок 3. План

1. Увімкніть plan mode (Shift+Tab у Claude Code) і попросіть план реалізації `SPEC.md`.
2. Перевірте, що в плані є: кроки по порядку, файли, які зміняться, ризики, **спосіб перевірки кожного кроку**.
3. Збережіть план у `PLAN.md`. Він переживе `/clear`, і реалізацію можна почати в чистій сесії з `@PLAN.md`.

### Крок 4. Реалізація

- Реалізуйте план крок за кроком, з перевіркою після кожного кроку.
- Додайте до промпту: «якщо потрібно відійти від плану, зупинись і поясни».
- Відхилення від плану записуйте одразу.

### Крок 5. Перевірка проти специфікації

Попросіть рев'юера, який **не бачив процесу реалізації**:

```
use a subagent to review the diff against SPEC.md, check every requirement
is implemented and the listed edge cases have tests, report gaps that affect
correctness and skip style
```

Якщо субагентів немає, відкрийте нову сесію і дайте їй лише `SPEC.md` і дифф.

Просіть докази: не «тести пройшли», а команду і її вивід.

## Що здати

1. PR, у якому є `SPEC.md`, `PLAN.md` і код.
2. Звіт за шаблоном:

```markdown
## ДЗ 3 — <ім'я>

**Інструмент:** <назва>
**Задача:** <одне речення>
**Витрати на spec + plan:** <хв>   **На реалізацію:** <хв>   **Ітерацій виправлення:** <N>

**Що знайшов рев'юер:** <перелік або «нічого суттєвого»>
**Де реалізація відійшла від плану і чому:** <перелік>
**Що б я додав у специфікацію наступного разу:** <одне речення>
```

## Критерії оцінювання

| Зараховано | Добре | Відмінно |
|---|---|---|
| Spec і plan є в репозиторії | Spec має перевірювані acceptance criteria і non-goals | Реалізація відповідає плану, відхилення задокументовані, є висновок рев'юера |

## Якщо у вас інший інструмент

| | План без змін у коді | Незалежне рев'ю |
|---|---|---|
| **Claude Code** | Plan mode (Shift+Tab) | Субагент або `/code-review` |
| **Cursor** | Plan mode (Shift+Tab) | Субагент або нова розмова |
| **GitHub Copilot** | Режим планування (перевірте назву у своїй версії) | Custom agent-рев'юер або Copilot code review |
| **OpenAI Codex** | Plan mode | Субагент-рев'юер |
| **Gemini CLI** | `/plan` | Субагент-рев'юер |
| **Z.ai ZCode** | Plan Mode (Shift+Tab) | Субагент-рев'юер або `/goal` з незалежною перевіркою |

`SPEC.md` і `PLAN.md` — звичайні markdown-файли, їх читає будь-який агент. Хочете формалізований процес — спробуйте GitHub Spec Kit, він працює з кількома агентами.

## Поради

- Не пропускайте non-goals: без них агент «покращує» суміжний код, і дифф розростається.
- Якщо агент двічі зробив не те, не виправляйте втретє. Оновіть `SPEC.md` або `PLAN.md`, зробіть `/clear` і почніть з уточненим промптом.
- Рев'юер завжди щось знайде. Виправляйте те, що стосується коректності; стиль — для лінтера (лекція 4).

## Додаткові матеріали

**Основне:**
- [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) — розділи «Brainstorm with Claude first», «Give Claude a check to run», «Review against your spec».
- [Permission modes: plan mode](https://code.claude.com/docs/en/permission-modes)
- [Goal (`/goal`)](https://code.claude.com/docs/en/goal) — умова завершення з автоматичною перевіркою.

**Щоб глибше зрозуміти тему:**
- [GitHub Spec Kit](https://github.github.com/spec-kit/) і [репозиторій](https://github.com/github/spec-kit)
- [Diving Into Spec-Driven Development With GitHub Spec Kit (Microsoft)](https://developer.microsoft.com/blog/spec-driven-development-spec-kit/)
- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — патерн evaluator-optimizer.

**Інші інструменти:**
- [GitHub Copilot: customization cheat sheet](https://docs.github.com/en/copilot/reference/customization-cheat-sheet)
- [OpenAI Codex: customization](https://developers.openai.com/codex/concepts/customization)
- [Cursor docs](https://cursor.com/docs)
- [ZCode docs](https://zcode.z.ai/en/docs) — Plan Mode і Goal Mode.
