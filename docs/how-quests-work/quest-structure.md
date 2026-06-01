# Quest structure

A quest is a set of tasks grouped into stages. Each stage has exactly one **required** task and any number of **optional** ones. Stages are played in order: the next one opens only after the current stage's required task is completed.

Required tasks move the quest forward and decide its outcome. Optional tasks don't affect the outcome — they are side objectives and rewards alongside the main line.

## How a task completes

A task can complete in one of three ways:

- **by condition** — the system checks the task's [success or failure condition](/datapack-reference/task-completion-conditions.md) on its own;
- **by a button** in the [quest book](/player-view/items.md) — if the task has a `buttons` field;
- **by the [`quest complete ... task`](/command-api/command-reference.md#quest-complete--task) command** — with the chosen outcome.

A completed task thus ends up with an outcome — success, failure, or skip.

> Every field of a quest and task file — with types and defaults — is collected in [Quest file format](/datapack-reference/quest-file-format.md). It also covers which forms `title` and `description` accept and which styles work.

## How a quest's outcome is built

A quest's outcome is computed from the required tasks of all stages:

- **success** (`success`) — all required tasks were completed successfully or skipped, but not all skipped.
- **failure** (`failure`) — at least one required task failed; the quest finishes immediately, without waiting for the rest.
- **skipped** (`skipped`) — all required tasks were skipped.

There's no separate "finish the quest" step: the outcome is derived from the required tasks, and the moment the last of them is done the quest moves to completed. What these states mean and how to read them from a command — see [Quest and task statuses](/how-quests-work/quest-and-task-statuses.md).

> **When completion is registered.** A task that meets its **condition** finishes the quest on the same game tick — together with its `on.success` hook. But if you close a required task with a **command or a button**, the quest's completion is registered only on the next tick: the task's own status changes at once, while a following `execute if quest … complete` in the same function will still find the quest active.

---

## Quests with no stages

Stages and tasks are optional — a quest may have no stages at all. Such a quest:

- appears in the book as [`active`](/how-quests-work/quest-and-task-statuses.md) after `quest give`;
- isn't shown in the HUD — there's nothing to pin;
- **never finishes on its own** — it has no required task for the system to compute an outcome from.

This is handy for narrative entries — rumors, notes, fragments of lore — that should sit in the player's journal without asking for anything. A ready recipe — see [Quest note](/quest-patterns/quest-note.md).

---

## See also

- [Active stage](/how-quests-work/active-stage.md) — which stage the system treats as current and why it shifts on the fly.
- [Quest and task statuses](/how-quests-work/quest-and-task-statuses.md) — what `active`, `complete` and the three outcomes mean, and how to read them from a command.
- [Quest file format](/datapack-reference/quest-file-format.md) — the `stages`, `tasks`, `condition`, `buttons` fields in JSON.
- [Task completion conditions](/datapack-reference/task-completion-conditions.md) — what drives automatic completion.
- [Quest note](/quest-patterns/quest-note.md) — the no-task quest pattern.
