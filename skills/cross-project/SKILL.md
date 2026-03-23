---
name: cross-project
description: Используй этот скилл всякий раз, когда задача затрагивает несколько проектов. Триггеры: пользователь упоминает названия смежных проектов рядом с задачей ("в этой задаче понадобится union", "нужно проверить academy_frontend"), пишет "исследуй X", "изучи X", "отредактируй X", "запомни взаимодействие", или вызывает /cross-project. Use this skill whenever working across multiple projects — user mentions project names alongside a task, says "investigate X", "edit X project", "save interaction to memory", or calls /cross-project. Invoke proactively when the task clearly spans multiple codebases.
---

## Шаг 1: Определение директорий

Run `pwd` to get the current project directory. **Base dir** = one level up (parent directory).

- Windows bash: `/c/Users/<username>/PycharmProjects/api` → base: `/c/Users/<username>/PycharmProjects/`
- macOS/Linux: `/Users/<username>/Projects/api` → base: `/Users/<username>/Projects/`

## Шаг 2: Разбор аргументов и режим

Parse `$ARGUMENTS` or infer from conversation context if triggered automatically.

**Определение режима — English и русский:**

| Вызов | Режим |
|---|---|
| `/cross-project union frontend` | Исследование (по умолчанию) |
| `исследуй union` / `изучи union` / `проверь union` | Исследование |
| `/cross-project union --edit` | Редактирование |
| `отредактируй union` / `union --правка` / `измени в union` | Редактирование |
| `/cross-project --remember` / `запомни` / `сохрани в память` | Память |
| `исследуй union --запомни` / `/cross-project union --remember` | Исследование + память |

Если режим неоднозначен — используй **Исследование**.

---

## Режим исследования (по умолчанию)

> Модель субагента: **haiku** · Инструменты: **Read, Grep, Glob**

Spawn one **Explore subagent** per project **in parallel**. Each receives:

```
Project path: <base_dir>/<project_name>
If directory does not exist, say "NOT FOUND" and stop.

Task context from <current_project_dir>:
<what the main agent is trying to accomplish — be specific>

Return ONLY what is relevant to this task:
1. Language, framework, purpose (one line)
2. Key modules/files relevant to the task context
3. Relevant endpoints or interfaces
4. Auth mechanism if relevant
5. Specific integration points with <current_project_name>

Return in this exact format:
---
PROJECT: <name>
STATUS: found | not_found
FRAMEWORK: <one line>
RELEVANT_MODULES: <bullet list>
ENDPOINTS: <bullet list or "none relevant">
AUTH: <one line>
INTEGRATION: <specific points>
NOTES: <anything else useful>
---
```

After all return, consolidate:

---
## Контекст смежных проектов: <project1>, <project2>, ...

**<project_name>**
- Назначение: ...
- Ключевые модули: ...
- Эндпоинты: ...
- Авторизация: ...
- Интеграция с `<current>`: ...

*(Не найден: "Директория не найдена — пропущено")*

**Общие замечания:** общие auth-потоки, Kafka-топики, API-контракты, общие идентификаторы.

---

## Режим редактирования (`--edit` / `--правка`)

> Модель субагента: **sonnet** · Инструменты: **Read, Grep, Glob, Edit, Write, Bash** · Изоляция: **worktree**

Before dispatching, prepare full context — do NOT spawn until all sections are ready:

```
Working directory: <base_dir>/<project_name>

## Контекст из <current_project_name>
<Почему нужно это изменение — бизнес/техническая причина>
<Релевантные типы, схемы, API-контракты из текущего проекта>
<Ограничения и что нельзя трогать>

## Задача
<Конкретные файлы и что именно изменить>

## Ожидаемый результат
<Как должен выглядеть результат>

## Отчёт субагента в формате:
---
CHANGED_FILES: <список файлов с путями>
SUMMARY: <что и почему изменено, по каждому файлу>
NOTES_FOR_CALLER: <что нужно обновить в вызывающем проекте>
---
```

Include subagent reports in the final task summary.

---

## Режим памяти (`--remember` / `запомни`)

1. Review what was explored/edited in this session.
2. Check if `cross_project_interactions.md` exists in the project memory directory.
   - Актуально → ответить: *"Память актуальна, изменений нет."*
   - Устарело → обновить только изменившиеся части.
   - Отсутствует → создать.
3. Write/update following the auto-memory format (type: `project`):

```markdown
---
name: cross_project_interactions
description: Как этот проект взаимодействует со смежными проектами — эндпоинты, auth-потоки, Kafka, контракты
type: project
---

## Смежные проекты

### <project_name>
- **Роль**: ...
- **Точки интеграции**: ...
- **Auth/token flow**: ...
- **Kafka-события**: ...
- **Заметки**: ...
```

4. Update `MEMORY.md` index if not already listed.

---

## Справочник команд

### Исследование
```
/cross-project union
/cross-project union academy_frontend
исследуй union
изучи union и academy_frontend
проверь что в union
```

### Редактирование
```
/cross-project union --edit
отредактируй union
union --правка
```
Субагент получит полный контекст, будет работать на **sonnet**, изменения изолированы в worktree.

### Память
```
/cross-project --remember
запомни взаимодействие
сохрани в память как мы работали с union
исследуй union --запомни
```

### Комбинированные
```
/cross-project union academy_frontend --remember
исследуй union и academy_frontend --запомни
```

> Все проекты ищутся в **родительской директории** текущего проекта.
> Несуществующие проекты — пропускаются с предупреждением.
