# Справочник команд

Точный перечень всех команд InkQuest: что принимает, что возвращает, какими ошибками отвечает. Сюда заходят свериться — *как именно записать*. За моделью «квест как читаемое состояние» и разобранными связками — [ИНТЕГРАЦИЯ С КОМАНДАМИ](/command-api/integration-with-commands.md); за готовыми сборками — [ПАТТЕРНЫ КВЕСТОВ](/quest-patterns/quest-patterns.md).

Все команды требуют уровень оператора **2+**.

---

## Соглашения

В записи синтаксиса: `<обязательный>` — обязательный аргумент, `[необязательный]` — необязательный, `a|b|c` — выбор одного из вариантов.

| Аргумент | Тип | Значение |
| --- | --- | --- |
| `<player>` | селектор **одного** игрока (`@s`, `@p`, ник) | `@a` нельзя — кроме [`quest scroll`](#quest-scroll). |
| `<players>` | селектор нескольких игроков | Только у [`quest scroll`](#quest-scroll). |
| `<id>` | Identifier `namespace:path` | Идентификатор квеста, например `example:escape`. |
| `<taskId>` | строка без namespace | Идентификатор задачи внутри квеста, например `push_the_lever`. |
| `<title>`, `<description>` | Text | Строка в кавычках или JSON-текст. |
| `<icon>` | Identifier | Иконка квеста (см. [`quest modify ... icon`](#quest-modify--icon)). |
| `<index>`, `<n>` | int | Целое число. |

> **Два набора слов про статус — не путай.**
>
> - **Задать исход** задачи или этапа: `success` · `failure` · `skip` (команда [`quest complete`](#quest-complete--task)).
> - **Проверить или отфильтровать** статус: `active` · `complete` · `succeeded` · `failed` · `skipped` · `pinned` (команды `test`, `query`, `list`, `execute if`). Что они значат — [СТАТУСЫ КВЕСТОВ И ЗАДАЧ](/how-quests-work/quest-and-task-statuses.md).

> Каждая команда возвращает **число** — счётчик или `1`/`0`. Его можно положить в scoreboard через `execute store result score …` или ветвить через `execute if`. Что именно возвращается — указано у команды.

---

## Создание и редактирование квестов

> Команды этого раздела работают только с **динамическими** квестами (созданными через `quest new`). Квест из датапака (`static`) командами не редактируется и не удаляется — правь его JSON.

### `quest new`

```
/quest new <id> [<title>] [<description>]
```

Создаёт пустой динамический квест. `title` и `description` можно задать сразу или позже через `quest modify`.

**Ошибки:** квест с таким `id` уже существует.

### `quest modify ... title`

```
/quest modify <id> title <title>
```

Меняет заголовок квеста.

### `quest modify ... description`

```
/quest modify <id> description <description>
```

Меняет описание квеста.

### `quest modify ... icon`

```
/quest modify <id> icon <icon>
```

Меняет иконку квеста. `icon` — Identifier **текстуры** иконки, а не предмет Minecraft. Допустимые значения и значение по умолчанию — в [ФОРМАТ ФАЙЛА КВЕСТА](/datapack-reference/quest-file-format.md#поля-квеста).

### `quest modify ... index`

```
/quest modify <id> index <n>
```

Задаёт порядок сортировки в книге квестов: меньше — выше.

### `quest modify ... repeatable`

```
/quest modify <id> repeatable true|false
```

Включает повторное прохождение. При `true` завершённый квест можно выдать заново — прогресс прошлого прохождения сбрасывается. При `false` (по умолчанию) выдача уже завершённого квеста блокируется. Активный (незавершённый) квест нельзя перевыдать ни при каком значении. См. [ПОВТОРЯЕМЫЕ КВЕСТЫ](/how-quests-work/repeatable-quests.md).

### `quest modify ... pin_mode`

```
/quest modify <id> pin_mode auto|off|force
```

Режим автозакрепления при **автовыдаче** квеста как зависимого от только что завершённого. На ручную выдачу через `quest give` не влияет.

| Значение | Поведение |
| --- | --- |
| `auto` | Закрепить, если родительский квест был закреплён в HUD (по умолчанию). |
| `force` | Закрепить всегда. |
| `off` | Не закреплять. |

См. [ЗАКРЕПЛЕНИЕ](/how-quests-work/pinning.md).

### `quest modify ... tasks add`

```
/quest modify <id> tasks add required <taskId> [<title>] [<description>]
/quest modify <id> tasks add optional <taskId> [<title>] [<description>]
```

Добавляет задачу к квесту.

- `required` — создаёт новый этап с этой задачей как обязательной.
- `optional` — добавляет необязательную задачу в последний этап (требует хотя бы одного `required`).

Задачи, добавленные командой, **не имеют условий выполнения** — завершить их можно только командой [`quest complete`](#quest-complete--task).

**Ошибки:** задача с таким `taskId` уже есть в квесте · `optional` в квест без этапов.

### `quest modify ... tasks remove`

```
/quest modify <id> tasks remove <taskId>
```

Удаляет задачу из квеста.

**Ошибки:** задача не существует · квест кто-то отслеживает (сначала сними его через `quest drop` или [`quest purge`](#quest-purge)).

### `quest remove`

```
/quest remove <id>
```

Удаляет динамический квест целиком — после этого его нельзя выдать или изменить.

**Ошибки:** квест не существует · квест статический · квест кто-то отслеживает.

---

## Выдача и прохождение

### `quest give`

```
/quest give <player> <id> [pin]
```

Выдаёт квест игроку — квест становится `tracked`. С флагом `pin` сразу закрепляется в HUD.

**Ошибки:** квест не существует · квест уже активен у игрока · квест завершён и не повторяемый.

### `quest drop`

```
/quest drop <player> <id>
```

Снимает квест с игрока. Весь прогресс по квесту теряется; если квест был закреплён — открепляется.

**Ошибки:** квест не существует · игрок его не отслеживает.

### `quest complete ... task`

```
/quest complete <player> <id> task <taskId> [success|failure|skip]
```

Завершает одну задачу с указанным исходом (по умолчанию `success`). Работает с задачей **любого** этапа, не только активного. Если задача обязательная и условия перехода соблюдены — квест переходит на следующий этап или завершается.

**Возвращает** `1` при успехе.

**Ошибки:** квест не существует · игрок не отслеживает · квест уже завершён · задача уже завершена.

### `quest complete ... stage`

```
/quest complete <player> <id> stage [success|failure|skip] [full|required]
```

Завершает задачи **активного этапа** (по умолчанию `success required`).

- `required` — только обязательную задачу этапа.
- `full` — все задачи этапа (обязательную и необязательные).

**Возвращает** число завершённых задач.

**Ошибки:** квест не существует · игрок не отслеживает · квест уже завершён · у квеста нет активного этапа.

### `quest complete` (весь квест)

```
/quest complete <player> <id> [success|failure|skip] [full|required]
```

Завершает незавершённые задачи **всех этапов** (по умолчанию `success required`).

- `required` — обязательные задачи всех этапов.
- `full` — все задачи всех этапов.

**Возвращает** число завершённых задач.

**Ошибки:** квест не существует · игрок не отслеживает · квест уже завершён.

> **Завершение квеста — на следующем тике.** Любая форма `quest complete` помечает задачи завершёнными сразу, но сам квест переходит в `complete` (и разблокирует зависимые по `after` квесты) только на следующем серверном тике. Поэтому в одной функции «закрой задачу → тут же проверь квест» не сработает: идущая следом `execute if quest … complete` ещё застанет квест активным — разнеси проверку на следующий тик. Автозавершение по условию задачи этого лага не имеет: задача и квест завершаются в один тик. См. [СТРУКТУРА КВЕСТА](/how-quests-work/quest-structure.md#как-складывается-исход-квеста).

---

## Закрепление в HUD

### `quest pin`

```
/quest pin <player> <id>
/quest pin <player> <id> <taskId>
```

Закрепляет квест в HUD игрока.

- без `taskId` — закрепляет обязательную задачу активного этапа.
- с `taskId` — закрепляет указанную задачу (должна быть активной: в активном этапе и не завершённой).

**Ошибки (без `taskId`):** квест не существует · не отслеживается · квест завершён · уже закреплён.

**Ошибки (с `taskId`):** квест не существует · не отслеживается · задача не активна · задача уже закреплена.

### `quest unpin`

```
/quest unpin <player> <id>
```

Открепляет квест из HUD игрока.

**Ошибки:** квест не существует · не отслеживается · квест не закреплён.

---

## Чтение состояния

Три команды читают одно состояние в разной форме: `test` — да/нет числом, `execute if` — ветка в цепочке, `query` — величина. Какую брать и как они сочетаются — [ИНТЕГРАЦИЯ С КОМАНДАМИ](/command-api/integration-with-commands.md).

### `quest test`

```
/quest test <player> <id> active|complete|succeeded|failed|skipped|pinned
/quest test <player> <id> task <taskId> active|complete|succeeded|failed|skipped|pinned
```

Проверяет статус квеста или задачи. **Возвращает** `1`, если статус совпал, иначе `0`, и печатает результат в чат. Для «тихой» проверки внутри командной логики бери [`execute if quest`](#execute-ifunless-quest-и-execute-ifunless-task).

**Ошибки:** квест не существует · игрок не отслеживает · (для задачи) задача не существует.

### `quest query`

Возвращает **число** — прогресс квеста, этапа или условия задачи. Кладётся в scoreboard через `execute store result score`.

**Прогресс по этапам квеста:**

```
/quest query <player> <id> stages complete|total|percent
```

**Задачи этапа:**

```
/quest query <player> <id> stage active tasks complete|total|percent
/quest query <player> <id> stage <n> tasks complete|total|percent
```

`stage active` — задачи активного этапа; `stage <n>` — задачи этапа с номером `n` (нумерация с нуля).

**Прогресс условия конкретной задачи:**

```
/quest query <player> <id> task <taskId> success|failure value|target|percent
```

`success` / `failure` — условие выполнения или провала; `value` — текущее значение, `target` — целевое, `percent` — процент 0–100. Применимо к условиям с прогресс-баром (`score`, `all`, `tasks` — см. [УСЛОВИЯ ВЫПОЛНЕНИЯ ЗАДАЧИ](/datapack-reference/task-completion-conditions.md)).

| Метрика | Что возвращает |
| --- | --- |
| `complete` | Сколько единиц завершено (этапов с выполненной обязательной задачей / задач этапа). |
| `total` | Сколько единиц всего. Работает для квеста в любом состоянии. |
| `percent` | Процент завершения, 0–100. |

> `complete` и `percent` отвечают ошибкой и `0` для **завершённого** квеста: прогресс активного этапа после завершения недоступен. `stage active …` отвечает ошибкой, если активного этапа нет.

Пример — положить число пройденных этапов в scoreboard:

```
execute store result score @s quest_stages run quest query @s example:escape stages complete
```

**Ошибки:** квест не существует · игрок не отслеживает · квест завершён (`complete` / `percent`) · нет активного этапа (`stage active`) · номер этапа вне диапазона (`stage <n>`) · задача не существует (`task`).

### `/execute if|unless quest` и `/execute if|unless task`

```
/execute if|unless quest <player> <id> active|complete|succeeded|failed|skipped|pinned …
/execute if|unless task <player> <id> <taskId> active|complete|succeeded|failed|skipped|pinned …
```

Расширение ванильной `/execute`: статус квеста или задачи становится таким же условием, как `if score` или `if block`, и комбинируется с ними в одной строке.

```
execute if quest @s story:ch1 succeeded unless quest @s story:ch2 active run say С возвращением!
execute if task @s example:escape push_the_lever succeeded run setblock ~ ~ ~ air
```

Разбор модели и связок — [ИНТЕГРАЦИЯ С КОМАНДАМИ](/command-api/integration-with-commands.md).

---

## Просмотр

### `quest list`

```
/quest list [all|static|dynamic]
```

Список зарегистрированных квестов.

| Фильтр | Что показывает |
| --- | --- |
| `all` | Все квесты (по умолчанию). |
| `static` | Только загруженные из датапаков. |
| `dynamic` | Только созданные командами. |

**Возвращает** число квестов.

### `quest list trackedby`

```
/quest list trackedby <player> [all|active|complete|succeeded|failed|skipped|pinned]
```

Квесты, которые отслеживает игрок, с фильтром по статусу (по умолчанию `all`).

**Возвращает** число квестов.

### `quest list tasks`

```
/quest list tasks <id> [all|required|optional|unused]
```

Задачи квеста.

| Фильтр | Что показывает |
| --- | --- |
| `all` | Задачи по этапам и неиспользуемые (по умолчанию). |
| `required` | Только обязательные (по одной на этап). |
| `optional` | Только необязательные. |
| `unused` | Объявлены в `tasks`, но не попали ни в один этап. |

**Возвращает** число задач.

---

## Служебные

### `quest purge`

```
/quest purge <id>
```

Снимает квест **со всех** игроков, включая офлайн: онлайн-игроки получают корректные события (HUD обновится), у офлайн — тихое удаление из сохранения. Нужна перед структурными правками ([`tasks remove`](#quest-modify--tasks-remove), [`quest remove`](#quest-remove)) или для сброса ивента. Уничтожает прогресс всех по этому квесту — применяй осознанно.

**Возвращает** число затронутых игроков.

**Ошибки:** квест не существует.

### `quest scroll`

```
/quest scroll <id> [<players>]
```

Выдаёт игрокам [свиток квеста](/player-view/items.md) — предмет, выдающий квест `<id>` по ПКМ. Без `<players>` свиток получает исполнитель команды, если это игрок. В отличие от остальных команд, `<players>` — **множественный** селектор (можно `@a`).

**Возвращает** число игроков, получивших свиток.

**Ошибки:** квест не существует.

---

## См. также

- [ИНТЕГРАЦИЯ С КОМАНДАМИ](/command-api/integration-with-commands.md) — модель «квест как состояние» и разобранные связки `test` / `query` / `execute if`.
- [СТАТУСЫ КВЕСТОВ И ЗАДАЧ](/how-quests-work/quest-and-task-statuses.md) — что значат `active`, `complete`, `succeeded`, `failed`, `skipped`, `pinned`.
- [ЗАКРЕПЛЕНИЕ](/how-quests-work/pinning.md) — как `pin` выглядит на стороне игрока.
- [ФОРМАТ ФАЙЛА КВЕСТА](/datapack-reference/quest-file-format.md) — те же поля (`title`, `icon`, `repeatable`, `pin_mode`) в JSON датапака.
- [ПРЕДМЕТЫ](/player-view/items.md) — свиток и книга квеста.
- [ПАТТЕРНЫ КВЕСТОВ](/quest-patterns/quest-patterns.md) — готовые сборки на этих командах.
