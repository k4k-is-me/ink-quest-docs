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

There are exactly six types. There are no others — an unknown `type` causes a quest load error.

| Type | What it checks | HUD |
|---|---|---|
| [`score`](#score-condition-score) | A scoreboard objective's value | Progress bar |
| [`predicate`](#predicate-condition-predicate) | A Minecraft predicate | Binary |
| [`all`](#composite-condition-all) | All nested conditions | Progress bar (met / total) |
| [`any`](#composite-condition-any) | At least one nested condition | Binary |
| [`none`](#composite-condition-none) | No nested condition | Binary |
| [`tasks`](#task-status-condition-tasks) | Statuses of tasks in the same quest | Progress bar if the target > 1 |

A **binary** condition is either met or not — it shows no progress bar under the task. A **progress bar** means the condition reports numeric progress toward a target.

---

## Score condition (`score`)

```json
{
	"type": "score",
	"objective": "<string>",
	"criterion": "<ScoreboardCriterion>",
	"player": "<string>",
	"initial": "<int>",
	"target": "<int>"
}
```

Uses Minecraft's scoreboard system. A task with this condition shows a progress bar in the HUD.

- `objective` — *required*. The scoreboard objective's name. If no such objective exists, it is created with the type from `criterion` when the task loads.
- `criterion` — the objective's type **at the moment it is created**. If an objective with this name already exists, the field is ignored. Any type Minecraft supports is allowed (full list on the [wiki](https://minecraft.wiki/w/Scoreboard#Criteria)). Default: `"dummy"`.
- `player` — the name of the player whose score is checked. Supports fake players (just like the scoreboard), but not selectors. If omitted, the score of the player doing the quest is checked.
- `initial` — the score set for the player when the task loads. If omitted, the score isn't overwritten (the player's current value in the objective is used).
- `target` — *required*. The value the score must reach. If `initial` is set and is **greater** than `target`, the condition works in reverse: the score must drop below `target`.

### Examples

#### Mine 5 oak logs
```json
{
	"type": "score",
	"objective": "oak_logs_mined",
	"criterion": "minecraft.mined:minecraft.oak_log",
	"target": 5
}
```

#### Kill 10,000 zombies server-wide
This needs some extra setup through lifecycle hooks (see [Task lifecycle hooks](/datapack-reference/task-lifecycle-hooks.md)).

```json
{
	"type": "score",
	"objective": "zombies_killed_total",
	"player": "GLOBAL",
	"target": 10000
}
```

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
			"target": 100
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

## Task-status condition (`tasks`)

Checks how many tasks **in the same quest** have a given status.

```json
{
	"type": "tasks",
	"tasks": ["task_id_1", "task_id_2"],
	"status": "success",
	"count": 1
}
```

- `tasks` — the list of task IDs to check (the pool). If omitted or empty, the pool is built from the tasks of the **active stage**, **excluding the task that carries this condition**. An explicit list may name tasks from any stage of the same quest.
- `status` — the expected status: `"success"`, `"failure"` or `"skipped"`. If omitted, a task in **any terminal** status counts (success, failure or skipped).
- `count` — the minimum number of tasks with the wanted status. If omitted, **all** tasks in the pool must match.

A progress bar appears when the target is known from the file and greater than 1: the target is `count`, or, without it, the length of the `tasks` list. So the form `{"type": "tasks"}` (the whole active stage) is binary and shows no progress bar, even though it still requires completing every task in the pool.

### Examples

#### Both side tasks must succeed

The stage's required task completes automatically once both optional tasks are done:

```json
{
	"type": "tasks",
	"tasks": ["side_task_a", "side_task_b"],
	"status": "success"
}
```

#### Complete at least 2 of 3 tasks

```json
{
	"type": "tasks",
	"tasks": ["task_a", "task_b", "task_c"],
	"status": "success",
	"count": 2
}
```

#### Every task of the current stage must finish (in any status)

The pool is all the **other** tasks of the active stage: the task carrying the condition isn't in it. So a condition on the stage's required task completes it once every optional task is done.

```json
{
	"type": "tasks"
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

---

## See also

- [Quest file format](/datapack-reference/quest-file-format.md) — the `condition` field in the overall list of task fields.
- [Task lifecycle hooks](/datapack-reference/task-lifecycle-hooks.md) — reacting to a task finishing (`on.success` / `on.failure`).
- [Waiting for an event](/quest-patterns/waiting-for-an-event.md) — there's no dedicated timer type; delays and waiting are built with this pattern.
- [Common questions and errors](/troubleshooting/common-questions-and-errors.md#a-task-condition-doesnt-fire) — if a condition doesn't complete a task.
