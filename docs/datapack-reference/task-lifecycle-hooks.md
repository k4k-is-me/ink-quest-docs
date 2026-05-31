# Task lifecycle hooks

For each task you can define actions that fire at the key moments of its life. Hooks are set in the task's `"on"` field:

```json
{
	"tasks": {
		"my_task": {
			"on": {
				"load": {
					"functions": ["namespace:on_task_load"],
					"tags": ["task_started"]
				},
				"tick": {
					"functions": ["namespace:on_task_tick"]
				},
				"pinned_tick": {
					"functions": ["namespace:on_task_pinned_tick"]
				},
				"unload": {
					"functions": ["namespace:on_task_unload"]
				},
				"success": {
					"functions": ["namespace:on_task_success"],
					"tags": ["task_done"]
				},
				"failure": {
					"functions": ["namespace:on_task_failure"]
				}
			}
		}
	}
}
```

Every hook is optional — list only the events you need to react to. Each hook has two optional fields:
- `functions` — an array of datapack mcfunction identifiers
- `tags` — an array of scoreboard tags added to the player when the hook fires

You can give just functions, just tags, or both. When a hook fires, all `tags` are added first, then all `functions` run — within the same tick. So a function in that same hook already sees its tags: you can add a tag and use it right away in the same hook's function.

---

## When each hook fires

| Hook | When it fires |
|-----|-------------------|
| `load` | Once per task lifetime — when the task first becomes part of the player's **active stage**. The "task loaded" mark is saved with the world and survives a player relog, a server restart and a datapack reload — the hook isn't called again until the task passes through `unload`. For repeatable quests (`repeatable: true`), re-granting starts a new task lifetime, and `load` fires again. |
| `tick` | Every tick while the task is in the active stage and unfinished. |
| `pinned_tick` | Every tick while the task is in the active stage, unfinished **and the quest is pinned**. Fires in addition to `tick`, not instead of it. |
| `unload` | Once per task lifetime — when the task stops being active: the task itself finished (by success, failure or skip), its stage was passed, the quest finished, or the quest was dropped from the player. The counterpart to `load`: while the task is loaded, a relog or restart doesn't fire `unload`. Fires strictly after `load`. |
| `success` | Once, the moment the task succeeds. Terminal — the task finishes and is no longer active; it can't fire again. |
| `failure` | Once, the moment the task fails. Terminal — mutually exclusive with `success`; it can't fire again. |

Functions run as the player: `@s` inside a function is the player whose task fired.

**Skipping a task fires neither `success` nor `failure`.** If a task finishes by skip (the `skip` button or the `quest complete ... skip` command), both hooks stay silent — only `unload` fires.

---

## Examples

### Give an item on success

```json
{
	"on": {
		"success": {
			"functions": ["story:rewards/give_key"]
		}
	}
}
```

The `story:rewards/give_key` function:
```mcfunction
give @s minecraft:tripwire_hook{display:{Name:'{"text":"Dungeon key"}'}}
```

### Add a tag and run a function on success

```json
{
	"on": {
		"success": {
			"functions": ["story:ch1/complete"],
			"tags": ["ch1_done"]
		}
	}
}
```

The hook both adds the `ch1_done` tag and calls the function — their order of application is described above: the tag is already visible inside the `story:ch1/complete` function.

### Set up in `load`, work in `tick`

`load` is for one-time task setup, `tick` for repeating work every tick. Here `load` sets up a scoreboard once, and `tick` works on it:

```json
{
	"on": {
		"load": {
			"functions": ["story:zombies/init"]
		},
		"tick": {
			"functions": ["story:zombies/accumulate"]
		}
	}
}
```

The full working recipe with these functions — a server-side counter shared by all players — is in the [Global quest](/quest-patterns/global-quest.md) pattern.

---

## See also

- [Quest file format](/datapack-reference/quest-file-format.md) — the `on` field in the overall list of task fields.
- [Task completion conditions](/datapack-reference/task-completion-conditions.md) — the `success`/`failure` conditions the same-named hooks are tied to.
- [Global quest](/quest-patterns/global-quest.md) — a full `load` + `tick` example: accumulating a shared score across all players.
