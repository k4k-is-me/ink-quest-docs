# Pinning

Pinning (`pin`) a quest means displaying it in the player's HUD: the quest title and all tasks of its active stage, each with a progress bar (for conditions with numeric progress, such as `score` or `all`). Pinning is about **visibility**, not logic: it determines what the player sees on screen, but does not affect how conditions are evaluated.

For how the HUD and quest book look — see [Player interface](/player-view/player-interface.md). This page covers the mechanics: how a quest ends up in the HUD and who controls that.

---

## What gets pinned

- **Multiple quests** can be pinned at the same time — each as a separate block in the HUD.
- Within a single quest, the HUD shows **all tasks of the active stage** — required and optional. Pinning a specific task only determines order: it moves to the top of the block, with the rest below. By default, the required task is on top; an operator can bring another active task forward: `quest pin <player> <id> <taskId>`.
- When the active stage changes, the task set in the HUD updates to the new stage's tasks and pinning is recalculated automatically.

---

## Who pins a quest

| Method | When it applies |
| --- | --- |
| Command `quest pin` | Explicitly, at any time. See [Command reference](/command-api/command-reference.md#quest-pin). |
| Flag `pin` on `quest give` | At the moment of manual quest assignment. |
| Quest field `pin_mode` | Only on **auto-assignment** by dependency (when completing one quest unlocks the next). |
| Player — from the quest book | Manually, by clicking a task. Controlled by gamerule (see below). |

When assigning manually (`quest give` without `pin`), the quest **is not pinned**. For auto-assignment by chain, behavior is determined by the `pin_mode` field.

### `pin_mode` values

The `pin_mode` field controls auto-pinning **only on auto-assignment by dependency chain** — that is, when a quest is assigned automatically because its dependency was completed. It has no effect on manual assignment with `quest give`.

| Value | Behavior on auto-assignment |
| --- | --- |
| `auto` (default) | Pin if the parent (just completed) quest was pinned. |
| `force` | Always pin. |
| `off` | Do not pin. |

Quest chains and auto-assignment are covered in [Quest chain](/quest-patterns/quest-chain.md).

---

## Pinning and condition checks

A common misconception: "conditions are only checked for pinned quests." That's not true.

Task conditions are checked every tick for tasks in the active stage of **all active quests** — pinned or not. Pinning only affects two things:

1. What is displayed in the HUD.
2. The `on.pinned_tick` hook — it fires only while the quest is pinned (unlike `on.tick`, which fires for all active quests). See [Task lifecycle hooks](/datapack-reference/task-lifecycle-hooks.md).

In other words, an unpinned quest keeps progressing and can complete on its own — the player just won't see it in the HUD until they open the quest book.

---

## Gamerule `allowManualQuestPin`

```
/gamerule allowManualQuestPin true|false
```

Default: **true**. A standard Minecraft gamerule — the value is saved with the world.

Controls whether the player can **manually** pin and unpin tasks from the quest book (by clicking or pressing Enter):

| `allowManualQuestPin` | Pinning from quest book | Commands `quest pin`/`unpin` | Auto-pin on assignment and stage change |
| --- | --- | --- | --- |
| `true` (default) | Allowed | Work | Works |
| `false` | Silently ignored | Work | Works |

With `false`, a player's attempt to pin a task from the book does nothing and shows no message — but the operator and automation still control pinning in full.

!!! tip
    Useful for maps where HUD direction is entirely in the author's hands: the player can't override a pinned quest, and the operator sets focus with commands and hooks.

---

## See also

- [Player interface](/player-view/player-interface.md) — how the HUD and quest book look.
- [Quest chain](/quest-patterns/quest-chain.md) — auto-assignment and `pin_mode` in practice.
- [Command reference](/command-api/command-reference.md) — commands `quest pin`, `quest unpin`, flag `pin` on `quest give`.
