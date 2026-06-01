# Repeatable quests

A normal quest is done once: after it is completed, it can't be granted to the same player again. A repeatable quest lifts that limit — a completed quest can be granted again, as many times as needed. This is the basis of daily tasks, farming quests, and recurring events.

A quest becomes repeatable through the `repeatable` flag:

```json
{
	"version": 3,
	"variant": 1,
	"title": "Daily wood gathering",
	"repeatable": true
}
```

By default `repeatable: false` — the quest is one-time.

---

## How re-granting works

When a repeatable quest is granted to a player who has **already completed** it — via [`quest give`](/command-api/command-reference.md#quest-give) or a [quest scroll](/player-view/items.md) — the system resets the progress of the previous run before granting it anew. The quest starts over from scratch: tasks are incomplete again, and the active stage is the first one again.

The key boundary: only a **completed** quest can be re-granted.

| Quest state for the player | `repeatable: false` | `repeatable: true` |
|---|---|---|
| Not granted | Granted | Granted |
| Active (unfinished) | Re-granting blocked | Re-granting blocked |
| Completed (success/failure/skip) | Re-granting blocked | Granted anew, progress reset |

An active quest can't be re-granted under any value of `repeatable` — it must first be completed or dropped via [`quest drop`](/command-api/command-reference.md#quest-drop).

> Re-granting starts a new "life" for the quest's tasks: the `on.load` hook fires again, just as on the first grant. Any side effects attached to it — handing out items, lines of dialogue, tags — will repeat. See [Task lifecycle hooks](/datapack-reference/task-lifecycle-hooks.md).

---

## Enabling via a command

For a dynamic quest, the flag can be toggled on the fly:

```mcfunction
/quest modify example:daily_quest repeatable true
```

The command works only with dynamic quests (created via `quest new`); for a quest from a datapack the flag is set in JSON. See [Command reference](/command-api/command-reference.md#quest-modify--repeatable).

---

## Resetting on a schedule

On its own, `repeatable` does not "refresh" a quest on a timer — it only allows re-granting. To get a real daily, a datapack function triggers the re-grant on its own schedule — for example, by the in-game time of day. The full recipe with ready-made JSON and mcfunction is in the [Repeatable quest](/quest-patterns/repeatable-quest.md) pattern.

---

## See also

- [Repeatable quest](/quest-patterns/repeatable-quest.md) — the pattern: a complete daily, with a schedule and reward.
- [Quest file format](/datapack-reference/quest-file-format.md#quest-fields) — the `repeatable` field in the full list of fields.
- [Quest and task statuses](/how-quests-work/quest-and-task-statuses.md#the-three-outcomes) — what "completed" means and why only a completed quest is re-granted.
