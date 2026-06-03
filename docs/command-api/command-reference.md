# Command reference

The exact list of every InkQuest command: what it takes, what it returns, which errors it answers with. Come here to check *exactly how to write it*. For the "quest as readable state" model and worked-out combinations, see [Integration with commands](/command-api/integration-with-commands.md); for ready-made builds, see [Quest patterns](/quest-patterns/quest-patterns.md).

Every command requires operator permission level **2+**.

---

## Conventions

In the syntax notation: `<required>` is a required argument, `[optional]` is optional, `a|b|c` is a choice of one option.

| Argument | Type | Meaning |
| --- | --- | --- |
| `<player>` | selector for a **single** player (`@s`, `@p`, name) | `@a` is not allowed — except in [`quest scroll`](#quest-scroll). |
| `<players>` | selector for multiple players | Only in [`quest scroll`](#quest-scroll). |
| `<id>` | Identifier `namespace:path` | Quest identifier, e.g. `example:escape`. |
| `<taskId>` | string without a namespace | Task identifier within a quest, e.g. `push_the_lever`. |
| `<title>`, `<description>` | Text | A quoted string or JSON text. |
| `<icon>` | Identifier | Quest icon (see [`quest modify ... icon`](#quest-modify--icon)). |
| `<index>`, `<n>` | int | An integer. |

> **Two sets of status words — don't mix them up.**
>
> - **Set an outcome** for a task or stage: `success` · `failure` · `skip` (the [`quest complete`](#quest-complete--task) command).
> - **Check or filter** by quest status: `active` · `complete` · `succeeded` · `failed` · `pinned`; by task status: the same plus `skipped` (the `test`, `query`, `list`, `execute if` commands). What they mean — [Quest and task statuses](/how-quests-work/quest-and-task-statuses.md).

> Every command returns a **number** — a counter or `1`/`0`. You can store it on the scoreboard with `execute store result score …` or branch on it with `execute if`. What exactly is returned is noted under each command.

---

## Creating and editing quests

> The commands in this section work only with **dynamic** quests (created via `quest new`). A datapack (`static`) quest cannot be edited or deleted by commands — edit its JSON instead.

### `quest new`

```
/quest new <id> [<title>] [<description>]
```

Creates an empty dynamic quest. `title` and `description` can be set right away or later via `quest modify`.

**Errors:** a quest with this `id` already exists.

### `quest modify ... title`

```
/quest modify <id> title <title>
```

Changes the quest's title.

### `quest modify ... description`

```
/quest modify <id> description <description>
```

Changes the quest's description.

### `quest modify ... icon`

```
/quest modify <id> icon <icon>
```

Changes the quest's icon. `icon` is the Identifier of the icon **texture**, not a Minecraft item. The allowed values and the default are in [Quest file format](/datapack-reference/quest-file-format.md#quest-fields).

### `quest modify ... index`

```
/quest modify <id> index <n>
```

Sets the sort order in the quest book: lower comes higher.

### `quest modify ... repeatable`

```
/quest modify <id> repeatable true|false
```

Enables replaying. With `true`, a completed quest can be granted again — the progress of the previous run is reset. With `false` (the default), granting an already completed quest is blocked. An active (unfinished) quest cannot be re-granted under any value. See [Repeatable quests](/how-quests-work/repeatable-quests.md).

### `quest modify ... pin_mode`

```
/quest modify <id> pin_mode auto|off|force
```

The auto-pin mode used when a quest is **auto-granted** as a dependent of a just-completed one. It does not affect manual granting via `quest give`.

| Value | Behavior |
| --- | --- |
| `auto` | Pin if the parent quest was pinned in the HUD (the default). |
| `force` | Always pin. |
| `off` | Never pin. |

See [Pinning](/how-quests-work/pinning.md).

### `quest modify ... tasks add`

```
/quest modify <id> tasks add required <taskId> [<title>] [<description>]
/quest modify <id> tasks add optional <taskId> [<title>] [<description>]
```

Adds a task to the quest.

- `required` — creates a new stage with this task as its required task.
- `optional` — adds an optional task to the last stage (requires at least one `required` first).

Tasks added by command **have no completion conditions** — they can only be finished with the [`quest complete`](#quest-complete--task) command.

**Errors:** a task with this `taskId` already exists in the quest · `optional` on a quest with no stages.

### `quest modify ... tasks remove`

```
/quest modify <id> tasks remove <taskId>
```

Removes a task from the quest.

**Errors:** the task does not exist · someone is tracking the quest (drop it first via `quest drop` or [`quest purge`](#quest-purge)).

### `quest remove`

```
/quest remove <id>
```

Removes a dynamic quest entirely — after this it cannot be granted or modified.

**Errors:** the quest does not exist · the quest is static · someone is tracking the quest.

---

## Granting and progression

### `quest give`

```
/quest give <player> <id> [pin]
```

Grants a quest to the player — the quest becomes `tracked`. With the `pin` flag it is pinned in the HUD right away.

**Errors:** the quest does not exist · the quest is already active for the player · the quest is completed and not repeatable.

### `quest drop`

```
/quest drop <player> <id>
```

Drops a quest from the player. All progress on the quest is lost; if the quest was pinned, it is unpinned.

**Errors:** the quest does not exist · the player is not tracking it.

### `quest complete ... task`

```
/quest complete <player> <id> task <taskId> [success|failure|skip]
```

Completes a single task with the given outcome (`success` by default). Works on a task in **any** stage, not just the active one. If the task is required and the transition conditions are met, the quest advances to the next stage or finishes.

**Returns** `1` on success.

**Errors:** the quest does not exist · the player is not tracking it · the quest is already finished · the task is already finished.

### `quest complete ... stage`

```
/quest complete <player> <id> stage [success|failure|skip] [full|required]
```

Completes the tasks of the **active stage** (`success required` by default).

- `required` — only the stage's required task.
- `full` — every task of the stage (required and optional).

**Returns** the number of tasks completed.

**Errors:** the quest does not exist · the player is not tracking it · the quest is already finished · the quest has no active stage.

### `quest complete` (whole quest)

```
/quest complete <player> <id> [success|failure|skip] [full|required]
```

Completes the unfinished tasks of **all stages** (`success required` by default).

- `required` — the required tasks of every stage.
- `full` — every task of every stage.

**Returns** the number of tasks completed.

**Errors:** the quest does not exist · the player is not tracking it · the quest is already finished.

> **Quest completion lands on the next tick.** Any form of `quest complete` marks the tasks finished at once, but the quest itself turns `complete` (and unlocks quests that depend on it via `after`) only on the next server tick. So "complete a task → check the quest right away" won't work in a single function: a following `execute if quest … complete` will still find the quest active — defer the check to the next tick. Automatic completion by a task's condition has no such lag: the task and the quest finish on the same tick. See [Quest structure](/how-quests-work/quest-structure.md#how-a-quests-outcome-is-built).

---

## Pinning in the HUD

### `quest pin`

```
/quest pin <player> <id>
/quest pin <player> <id> <taskId>
```

Pins the quest in the player's HUD.

- without `taskId` — pins the required task of the active stage.
- with `taskId` — pins the specified task (it must be active: in the active stage and unfinished).

**Errors (without `taskId`):** the quest does not exist · not tracked · the quest is finished · already pinned.

**Errors (with `taskId`):** the quest does not exist · not tracked · the task is not active · the task is already pinned.

### `quest unpin`

```
/quest unpin <player> <id>
```

Unpins the quest from the player's HUD.

**Errors:** the quest does not exist · not tracked · the quest is not pinned.

---

## Reading state

Three commands read the same state in different forms: `test` — yes/no as a number, `execute if` — a branch in a chain, `query` — a value. Which to use and how they combine — [Integration with commands](/command-api/integration-with-commands.md).

### `quest test`

```
/quest test <player> <id> active|complete|succeeded|failed|pinned
/quest test <player> <id> task <taskId> active|complete|succeeded|failed|skipped|pinned
```

Checks the status of a quest or task. **Returns** `1` if the status matches, otherwise `0`, and prints the result to chat. For a silent check inside command logic, use [`execute if quest`](#execute-ifunless-quest-and-execute-ifunless-task).

**Errors:** the quest does not exist · the player is not tracking it · (for a task) the task does not exist.

### `quest query`

Returns a **number** — the progress of a quest, a stage, or a task condition. Store it on the scoreboard with `execute store result score`.

**Progress over a quest's stages:**

```
/quest query <player> <id> stages complete|total|percent
```

**Tasks of a stage:**

```
/quest query <player> <id> stage active tasks complete|total|percent
/quest query <player> <id> stage <n> tasks complete|total|percent
```

`stage active` — tasks of the active stage; `stage <n>` — tasks of the stage numbered `n` (zero-based).

**Progress of a specific task's condition:**

```
/quest query <player> <id> task <taskId> success|failure value|target|percent
```

`success` / `failure` — the success or failure condition; `value` — the current value, `target` — the target, `percent` — a percentage 0–100. Applies to conditions with a progress bar (`score`, `all`, `optionals` — see [Task completion conditions](/datapack-reference/task-completion-conditions.md)).

| Metric | What it returns |
| --- | --- |
| `complete` | How many units are completed (stages whose required task is done / tasks of a stage). |
| `total` | How many units there are in total. Works for a quest in any state. |
| `percent` | Completion percentage, 0–100. |

> `complete` and `percent` answer with an error and `0` for a **finished** quest: the active-stage progress is no longer available once the quest is done. `stage active …` answers with an error if there is no active stage.

Example — store the number of completed stages on the scoreboard:

```
execute store result score @s quest_stages run quest query @s example:escape stages complete
```

**Errors:** the quest does not exist · the player is not tracking it · the quest is finished (`complete` / `percent`) · no active stage (`stage active`) · stage number out of range (`stage <n>`) · the task does not exist (`task`).

### `/execute if|unless quest` and `/execute if|unless task`

```
/execute if|unless quest <player> <id> active|complete|succeeded|failed|pinned …
/execute if|unless task <player> <id> <taskId> active|complete|succeeded|failed|skipped|pinned …
```

An extension of vanilla `/execute`: the status of a quest or task becomes the same kind of condition as `if score` or `if block`, and combines with them in a single line.

```
execute if quest @s story:ch1 succeeded unless quest @s story:ch2 active run say Welcome back!
execute if task @s example:escape push_the_lever succeeded run setblock ~ ~ ~ air
```

The model and combinations are worked out in [Integration with commands](/command-api/integration-with-commands.md).

---

## Listing

### `quest list`

```
/quest list [all|static|dynamic]
```

Lists the registered quests.

| Filter | What it shows |
| --- | --- |
| `all` | All quests (the default). |
| `static` | Only those loaded from datapacks. |
| `dynamic` | Only those created by commands. |

**Returns** the number of quests.

### `quest list trackedby`

```
/quest list trackedby <player> [all|active|complete|succeeded|failed|pinned]
```

The quests the player is tracking, filtered by status (`all` by default).

**Returns** the number of quests.

### `quest list tasks`

```
/quest list tasks <id> [all|required|optional|unused]
```

The tasks of a quest.

| Filter | What it shows |
| --- | --- |
| `all` | Tasks grouped by stage, plus unused ones (the default). |
| `required` | Only required tasks (one per stage). |
| `optional` | Only optional tasks. |
| `unused` | Declared in `tasks` but not placed in any stage. |

**Returns** the number of tasks.

---

## Utility

### `quest purge`

```
/quest purge <id>
```

Drops the quest from **every** player, including offline ones: online players get the proper events (the HUD updates), and offline players have it quietly removed from their save. Needed before structural edits ([`tasks remove`](#quest-modify--tasks-remove), [`quest remove`](#quest-remove)) or to reset an event. It destroys everyone's progress on this quest — use it deliberately.

**Returns** the number of players affected.

**Errors:** the quest does not exist.

### `quest scroll`

```
/quest scroll <id> [<players>]
```

Gives players a [quest scroll](/player-view/items.md) — an item that grants quest `<id>` on right-click. Without `<players>`, the scroll goes to the command's executor if they are a player. Unlike the other commands, `<players>` is a **multiple** selector (`@a` is allowed).

**Returns** the number of players who received the scroll.

**Errors:** the quest does not exist.

---

## See also

- [Integration with commands](/command-api/integration-with-commands.md) — the "quest as state" model and worked-out `test` / `query` / `execute if` combinations.
- [Quest and task statuses](/how-quests-work/quest-and-task-statuses.md) — what `active`, `complete`, `succeeded`, `failed`, `pinned` mean; why a quest never receives `skipped`.
- [Pinning](/how-quests-work/pinning.md) — how `pin` looks on the player's side.
- [Quest file format](/datapack-reference/quest-file-format.md) — the same fields (`title`, `icon`, `repeatable`, `pin_mode`) in datapack JSON.
- [Items](/player-view/items.md) — the quest scroll and quest book.
- [Quest patterns](/quest-patterns/quest-patterns.md) — ready-made builds on these commands.
