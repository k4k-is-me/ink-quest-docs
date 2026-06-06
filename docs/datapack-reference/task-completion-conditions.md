# Task completion conditions

Tasks can complete automatically when a given condition is met, through buttons in the quest book (the `buttons` field), or by an explicit command.

> Automatic condition checking is available only for datapack quests. Tasks created by commands are completed solely with the `quest complete` command.

Conditions are checked every tick on the tasks of the active stage of every active quest — pinned and unpinned alike.

For each task you can set a success condition and a failure condition:

```json
{
	"tasks": {
		"my_task": {
			"condition": {
				"success": {
					// The success condition
				},
				"failure": {
					// The failure condition
				}
			}
		}
	}
}
```

Both fields inside `condition` are optional — you can set just one or both.

---

## Condition format

A condition is an object with a `type` field and a set of fields unique to that type.

```json
{
	"type": "condition type"
	// Fields unique to each type
}
```

There are exactly seven types. There are no others — an unknown `type` causes a quest load error.

| Type | What it checks | HUD |
|---|---|---|
| [`score`](#score-condition-score) | A scoreboard objective's value for the context player | Progress bar |
| [`global_score`](#global-counter-global_score) | A scoreboard objective's value for a fixed holder | Progress bar |
| [`predicate`](#predicate-condition-predicate) | A Minecraft predicate | Binary |
| [`all`](#composite-condition-all) | All nested conditions | Progress bar (met / total) |
| [`any`](#composite-condition-any) | At least one nested condition | Binary |
| [`none`](#composite-condition-none) | No nested condition | Binary |
| [`optionals`](#optional-task-status-condition-optionals) | Statuses of the optional tasks in the active stage | Progress bar (`k / N`)    |

A **binary** condition is either met or not — it shows no progress bar. A **progress bar** means the condition reports numeric progress toward a target; the bar is shown only when the target value is greater than 1 (for example, `score` with a range of 0 → 1 or `all` with a single sub-condition — no bar).

---

## Score condition (`score`)

```json
{
	"type": "score",
	"objective": "<string>",
	"criterion": "<ScoreboardCriterion>",
	"from": 0,
	"to": "<int>",
	"reset": true
}
```

Uses Minecraft's scoreboard system. Checks the **context player's** score (the one doing the quest). If the range from `from` to `to` spans more than 1, a progress bar appears under the task in the HUD.

- `objective` — *required*. The scoreboard objective's name. If no such objective exists, it is created with the type from `criterion` when the task loads.
- `criterion` — the objective's type **at the moment it is created**. If an objective with this name already exists, the field is ignored. Any type Minecraft supports is allowed (full list on the [wiki](https://minecraft.wiki/w/Scoreboard#Criteria)). Default: `"dummy"`.
- `from` — the starting point. Default: `0`. If `from > to`, the condition works in **reverse**: the score must drop below `to`. Also defines the zero baseline for the progress bar.
- `to` — *required*. The target value. Ascending: `score >= to`. Descending: `score <= to`.
- `reset` — if `true` (default), the player's score in `objective` is set to `from` when the task loads. If `false`, the score is left untouched: the condition tracks the current value.

### Examples

#### Mine 5 oak logs
```json
{
	"type": "score",
	"objective": "oak_logs_mined",
	"criterion": "minecraft.mined:minecraft.oak_log",
	"to": 5
}
```

#### Survive a countdown timer (30 seconds = 600 ticks)
```json
{
	"type": "score",
	"objective": "my_timer",
	"from": 600,
	"to": 0
}
```

The score is set to 600 when the task loads. Each tick it decreases (via `on.tick` or another mechanism). The condition fires when the score reaches 0.

#### Track an existing counter without resetting it
```json
{
	"type": "score",
	"objective": "mob_kills",
	"to": 100,
	"reset": false
}
```

---

## Global counter (`global_score`)

```json
{
	"type": "global_score",
	"objective": "<string>",
	"player": "#GLOBAL",
	"from": 0,
	"to": "<int>"
}
```

Like `score`, but checks the score of a **fixed holder** (`player`) instead of the context player. The `criterion` and `reset` fields are absent: the objective criterion is fixed (`dummy`) and the scoreboard value is **never written** when the task loads — with many players, each load would otherwise reset the shared counter.

- `objective` — *required*. The scoreboard objective's name (created as `dummy` if it doesn't exist).
- `player` — the holder's name. Supports fake players. Default: `"#GLOBAL"`.
- `from` — the starting baseline (determines direction and the zero point of the progress bar). Default: `0`.
- `to` — *required*. The target value.

### Example

#### Kill 10,000 zombies server-wide
```json
{
	"type": "global_score",
	"objective": "zombies_killed_total",
	"to": 10000
}
```

Full breakdown of this pattern: [Global quest](/quest-patterns/global-quest.md).

---

## Predicate condition (`predicate`)

```json
{
	"type": "predicate",
	"predicate": "<Identifier>"
}
```

Uses Minecraft's predicate system. Predicates are conditions described in datapack JSON files; the predicate is evaluated against the player doing the quest. The task completes when the predicate returns true. A binary condition: no progress bar.

---

## Composite condition (`all`)

Lets you combine several conditions — the task completes only when all of them are met. HUD progress = the number of met sub-conditions.

```json
{
	"type": "all",
	"conditions": [
		{ "type": "..." },
		{ "type": "..." }
	]
}
```

Conditions inside `all` can be of any type and nested within one another without limit.

### Example

```json
{
	"type": "all",
	"conditions": [
		{
			"type": "score",
			"objective": "gold_collected",
			"to": 100
		},
		{
			"type": "predicate",
			"predicate": "example:has_key"
		}
	]
}
```

---

## Composite condition (`any`)

The task completes when **at least one** of the sub-conditions is met. Binary: no progress bar.

```json
{
	"type": "any",
	"conditions": [
		{ "type": "..." },
		{ "type": "..." }
	]
}
```

---

## Composite condition (`none`)

The task completes when **none** of the sub-conditions is met. Binary: no progress bar.

```json
{
	"type": "none",
	"conditions": [
		{ "type": "..." },
		{ "type": "..." }
	]
}
```

---

## Optional-task status condition (`optionals`)

Completes the task when the **optional tasks of the active stage** reach a given status.

```json
{
	"type": "optionals",
	"status": "success",
	"min": 2
}
```

The pool is built automatically: all optional tasks in the active stage, **excluding** the stage's required task and the task that carries this condition. This means a condition on the required task watches all optional tasks; a condition on an optional task watches all other optional tasks.

- `status` — the expected status: `"success"`, `"failure"` or `"skipped"`. If omitted, a task in **any terminal** status counts.
- `min` — the minimum number of tasks with the wanted status. If omitted, **all** optional tasks in the pool must match.

If the pool is empty (no other optional tasks exist), the condition is met immediately. When N > 1, the HUD shows a progress bar `k / N`, where `k` is the number of matching tasks and `N` is `min` or the pool size. When N ≤ 1, no bar is shown.

### Examples

#### All optional tasks must succeed

The stage's required task completes automatically once every optional task succeeds:

```json
{
	"type": "optionals",
	"status": "success"
}
```

#### Complete at least 2 out of 3 optional tasks

```json
{
	"type": "optionals",
	"status": "success",
	"min": 2
}
```

---

## Manual completion buttons (`buttons`)

Let the player finish a task themselves, straight from the quest book. This is an alternative to `condition` — instead of an automatic check, the designer gives the player an explicit choice.

```json
{
	"tasks": {
		"choose": {
			"title": "Make a choice",
			"buttons": ["success", "failure", "skip"]
		}
	}
}
```

Allowed values: `"success"`, `"failure"`, `"skip"`. The order in the array doesn't matter — in the book the buttons always show in the order success → failure → skip. Duplicates are collapsed (with a warning in the log).

The buttons are drawn under the task in the quest book. Hovering shows a tooltip. On click the server checks that the task is active and unfinished, then completes it with the chosen status.

`buttons` and `condition` are independent — you can use them together. The buttons fire regardless of the condition's state.

> Completing a task via a button, like `/quest complete`, moves the quest to `complete` only on the **next tick** — unlike automatic conditions. See [Quest structure](/how-quests-work/quest-structure.md#how-a-quests-outcome-is-built).

---

## See also

- [Quest file format](/datapack-reference/quest-file-format.md) — the `condition` field in the overall list of task fields.
- [Task lifecycle hooks](/datapack-reference/task-lifecycle-hooks.md) — reacting to a task finishing (`on.success` / `on.failure`).
- [Waiting for an event](/quest-patterns/waiting-for-an-event.md) — there's no dedicated timer type; delays and waiting are built with this pattern.
- [Common questions and errors](/troubleshooting/common-questions-and-errors.md#a-task-condition-doesnt-fire) — if a condition doesn't complete a task.
