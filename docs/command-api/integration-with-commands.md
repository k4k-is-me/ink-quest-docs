# Integration with commands

The core idea: **a quest's state can be read by commands**. A quest is not a sealed black box but a source of data — you can query it straight from command blocks, functions, and `/execute` chains.

This page is about the mental model and how the tools fit together. The exact syntax of each command is in [Command reference](/command-api/command-reference.md).

---

## Three ways to read state

| Tool | What it returns | When to use it |
| --- | --- | --- |
| `quest test` | `1` / `0` and a chat printout | To check a status by hand. |
| `execute if quest` / `execute if task` | An `/execute` branch (yes / no) | To weave a status into a command chain — with no chat output. |
| `quest query` | A number — progress, counter, percentage | To get a **value**, not just yes/no. |

All three read the same state, just in a different form. Choose by what you need on the output: `quest test` answers in chat when you check by hand; `execute if` branches commands silently; `quest query` hands back a number for the scoreboard.

---

## `execute if` — status as a condition

`/execute if|unless quest` and `/execute if|unless task` are an extension of vanilla `/execute`. A quest's status becomes the same kind of condition as `if block` or `if score`, and combines with them in a single line:

```
execute if quest @s story:ch1 succeeded unless quest @s story:ch2 active run say Welcome back!
```

This is the most direct way to tie an in-game event to progress: open a door, spawn an NPC, hand out a reward — to exactly the players whose state fits.

---

## `quest query` — progress as a number

`quest query` hands back a numeric value that you can conveniently store on the scoreboard with `store result`:

```
execute store result score @s dungeon_pct run quest query @s story:dungeon stages percent
```

From there the number is an ordinary score: you can show it on the scoreboard, compare it with `if score`, display it in a bossbar. This is how quest progress builds into progress bars, thresholds, and indicators that live entirely on the vanilla scoreboard.

> `quest query` reads progress at every level: you can ask about a quest's stages, about a stage's tasks, and about the value of a specific condition (`value` / `target` / `percent`). The full list of forms is in [Command reference](/command-api/command-reference.md#quest-query).

---

## The flip side: commands change state

Reading is half of the integration. The other half is that the quest itself is **driven** by commands: granting (`quest give`), completing tasks (`quest complete`), pinning (`quest pin`). And tasks, in turn, can **run** your functions through the `on` hooks — see [Task lifecycle hooks](/datapack-reference/task-lifecycle-hooks.md).

The result is a closed loop: a command block grants a quest → on completion the task calls your function → the function reads the state of other quests and decides what to grant next. Every [pattern](/quest-patterns/quest-patterns.md) is built on this loop — from chains to server-wide events.

---

## A direct link to the scoreboard

The `score` condition reads an ordinary scoreboard objective, and the `on` hooks can change that objective through their functions. This is the second point of contact with vanilla commands: you count whatever you like with Minecraft's own tools (statistics, custom counters), and the quest merely watches for the target to be reached. The clearest example is the server-wide counter in [Global quest](/quest-patterns/global-quest.md).

---

## See also

- [Command reference](/command-api/command-reference.md) — the exact syntax of `quest test`, `quest query`, `execute if quest`.
- [Task lifecycle hooks](/datapack-reference/task-lifecycle-hooks.md) — how a task calls your functions.
- [Task completion conditions](/datapack-reference/task-completion-conditions.md) — the `score` condition as a bridge to the scoreboard.
- [Quest patterns](/quest-patterns/quest-patterns.md) — ready-made builds on these mechanisms.
