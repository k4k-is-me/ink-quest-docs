# Quest and task statuses

A quest and its tasks move through a small set of states: a quest is granted, played, and finishes with one of three outcomes. This page is about that state model and the vocabulary that names it — what each status means and how to read it from a command.

## A quest's lifecycle

The moment a quest is granted to a player it is **tracked** (`tracked`) — it lands in the book and is recorded against that player. A tracked quest goes through two phases:

- **active** (`active`) — not finished yet, progress is counted;
- **complete** (`complete`) — it has an outcome, progress no longer changes.

```text
tracked — quest granted to the player
├── active — being played, progress counted
│   └── pinned — pinned to the HUD
└── complete — has an outcome, progress frozen
    ├── success — succeeded
    └── failure — failed
```

**A completed quest is frozen.** Its status and progress no longer change, and edits to the quest itself don't reach it — for someone who has already finished it, it stays exactly as it ended. Why editing a quest affects those playing it now but not those who finished it — see [Active stage](/how-quests-work/active-stage.md).

---

## Two outcomes

A completed quest is always in one of two outcomes:

- **success** — all required tasks are complete (succeeded or skipped).
- **failure** — at least one required task failed. The quest finishes immediately, without waiting for the rest.

> Skipping all required tasks with `/quest complete ... skip` also yields `success`, not a separate status. Skip describes the outcome of an individual task, not the quest as a whole.

The outcome is computed from the required tasks of the stages — optional tasks don't affect it. How a playthrough is built from stages and tasks — see [Quest structure](/how-quests-work/quest-structure.md).

---

## Pinning is a separate dimension

`pinned` isn't a phase of the lifecycle but a visibility mark on top of it: a pinned quest is shown in the on-screen HUD. Only an **active** quest can be pinned; completing it removes it from the HUD. Pinning doesn't affect how conditions are evaluated — only what the player sees in front of them. Details — see [Pinning](/how-quests-work/pinning.md).

---

## Task statuses

A task has the same set of states as a quest. A task is **active** (`active`) while it is in the [active stage](/how-quests-work/active-stage.md) and not yet completed; tasks of future and finished stages are not active. When a task is completed it becomes complete with the same outcome — `success`, `failure`, or `skipped`. An active task can be **pinned** — in the HUD it rises to the top of the block.

---

## Reading a status from a command

A status is readable state: you can check it with a command (`quest test`, `/execute if quest`) and filter a player's quest list by it (`quest list trackedby`). What each command takes and returns — see [Command reference](/command-api/command-reference.md#reading-state); how to weave a status into command logic — see [Integration with commands](/command-api/integration-with-commands.md).

Five statuses are available to check:

| Status | Means |
| --- | --- |
| `active` | Granted and not yet finished. |
| `complete` | Finished with any outcome. |
| `succeeded` | Finished with success (outcome `success`). |
| `failed` | Finished with failure (outcome `failure`). |
| `pinned` | Pinned to the HUD (only an active one can be). |

> **Two sets of status words — don't mix them up.** You *set* an outcome with one set of words and *check* a status with another:
>
> - set an outcome (`quest complete`): `success` · `failure` · `skip`;
> - check a status (`test`, `execute if`, `list`): `active` · `complete` · `succeeded` · `failed` · `pinned`.
>
> The check words `succeeded` / `failed` are past tense, unlike the outcome words `success` / `failure`. The word `skip` sets an individual task's outcome but creates no separate quest-level status.

---

## A quest with no stages

> A quest with no stages stays `active` forever: it has no required tasks to compute an outcome from, so it never finishes on its own. This is how narrative entries are made — rumors, notes, fragments of lore. A ready recipe — see [Quest note](/quest-patterns/quest-note.md).

---

## See also

- [Quest structure](/how-quests-work/quest-structure.md) — how a quest's outcome is built from stages and tasks.
- [Active stage](/how-quests-work/active-stage.md) — why editing a quest affects those playing it but not those who finished.
- [Pinning](/how-quests-work/pinning.md) — what `pinned` does on the player's side.
- [Command reference](/command-api/command-reference.md#reading-state) — exact syntax of `quest test`, `execute if quest`, `quest list trackedby`.
- [Integration with commands](/command-api/integration-with-commands.md) — how to read a quest's state from command blocks and functions.
