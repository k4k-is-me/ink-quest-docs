# Quest file format

A static quest is a single JSON file in a datapack at `data/<namespace>/quests/<id>.json`, where `<id>` becomes the path part of the quest's identifier (`<namespace>:<id>`). This page is the precise reference: which fields exist, what type each one is, what's required and what isn't.

> New here? Read [Quick start](/getting-started/quick-start.md) and [Quest structure](/how-quests-work/quest-structure.md) first. This page is the field dictionary you'll want to come back to.

The minimal valid quest is the two version fields plus a title:

```json
{
	"version": 3,
	"variant": 1,
	"title": "Rumor of a missing caravan"
}
```

Such a quest exists, can be granted to a player and shows up in the book, but never completes on its own — it has no tasks (see [Quest note](/quest-patterns/quest-note.md)).

---

## Format version: `version` and `variant`

Two required numeric fields at the top of the file. They guard against loading a quest into an incompatible version of the mod.

```json
{
	"version": 3,
	"variant": 1
}
```

| Field | What it means | What happens on a mismatch |
|---|---|---|
| `version` | Incompatible format version. Changes when an edit **breaks** old files: a required field added or removed, a field's type changed, an allowed value of a field removed. | The quest **won't load** — an incompatibility error goes to the log. |
| `variant` | Compatible format version. Changes when an edit is **backward-compatible**: an optional field added or removed, or a new allowed value added to a field. | The quest **loads**, but a warning goes to the log: some new features may be ignored. |

Rule of thumb: use the mod's current values — **`version: 3`, `variant: 1`**. If a quest stopped loading after a mod update, check these numbers first.

---

## Quest fields

Set at the top level of the file.

| Field         | Type                       | Required | Default                      | Description                                                                                                                                          |
| ------------- | -------------------------- | :------: | ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `version`     | `int`                      |   yes    | —                            | Format version (see above). Currently `3`.                                                                                                           |
| `variant`     | `int`                      |   yes    | —                            | Format variant (see above). Currently `1`.                                                                                                           |
| `title`       | Text                       |   yes    | —                            | The quest's title. See [Text fields and styling](#text-fields-and-styling).                                                                          |
| `description` | Text                       |    no    | —                            | The quest's description.                                                                                                                             |
| `icon`        | Identifier                 |    no    | `travelcorequesting:default` | The quest's icon in the HUD and the book. Atlas `textures/icons/<path>.png`.                                                                         |
| `index`       | `int`                      |    no    | `0`                          | Sort order in the book: lower comes higher.                                                                                                          |
| `repeatable`  | `bool`                     |    no    | `false`                      | Whether the quest can be granted again after completion. See [Repeatable quests](/how-quests-work/repeatable-quests.md).                             |
| `pin_mode`    | `auto` \| `off` \| `force` |    no    | `auto`                       | Auto-pin mode when the quest is auto-granted as a dependent. See [Pinning](/player-view/pinning.md).                                                 |
| `after`       | list of Identifier groups  |    no    | —                            | Dependencies on other quests. See [Dependencies between quests](/how-quests-work/quest-dependencies.md).                                             |
| `require`     | object                     |    no    | —                            | Extra unlock conditions (`tags`, `predicate`). Ignored without `after`. See [Dependencies between quests](/how-quests-work/quest-dependencies.md).   |
| `tasks`       | object `taskId → task`     |    no    | —                            | The quest's task dictionary. See [Task fields](#task-fields).                                                                                        |
| `stages`      | list of `taskId` lists     |    no    | —                            | Stages: each stage is a list of task IDs, the first one is required. See [Active stage](/how-quests-work/active-stage.md).                           |
| `$schema`     | string                     |    no    | —                            | Path to a JSON schema for your editor. Ignored by the mod.                                                                                           |

> `tasks` defines which tasks **exist**, and `stages` defines how they're **laid out into stages**. A task declared in `tasks` but not placed in any `stages` stage takes no part in the quest (you can inspect it with `/quest list tasks <id> unused`).

### Example with all the main fields

```json
{
	"version": 3,
	"variant": 1,
	"title": "The missing caravan",
	"description": "The traders are overdue. Someone has to find out what happened.",
	"icon": "travelcorequesting:note",
	"index": 10,
	"after": [["story:prologue"]],
	"require": { "tags": ["joined_guild"] },
	"tasks": {
		"find_tracks": {
			"title": "Find the caravan's tracks",
			"description": "The eastern road is where they were last seen."
		},
		"talk_to_witness": {
			"title": "Talk to a witness"
		}
	},
	"stages": [
		["find_tracks", "talk_to_witness"]
	]
}
```

---

## Task fields

Each task is an entry in the `tasks` object, where the entry's key is its `taskId` (without a namespace, e.g. `find_tracks`).

| Field | Type | Required | Default | Description |
|---|---|:---:|---|---|
| `title` | Text | yes | — | The task's title. |
| `description` | Text | no | — | The task's description. |
| `condition` | object | no | — | Auto-completion conditions: `success` and/or `failure`. See [Task completion conditions](/datapack-reference/task-completion-conditions.md). |
| `on` | object | no | — | Lifecycle hooks: `load`, `tick`, `pinned_tick`, `unload`, `success`, `failure`. See [Task lifecycle hooks](/datapack-reference/task-lifecycle-hooks.md). |
| `buttons` | list of `success` \| `failure` \| `skip` | no | — | Manual-completion buttons in the book. See [Task completion conditions](/datapack-reference/task-completion-conditions.md#manual-completion-buttons-buttons). |

```json
"find_tracks": {
	"title": "Find the caravan's tracks",
	"description": "The eastern road is where they were last seen.",
	"condition": {
		"success": { "type": "predicate", "predicate": "story:at_caravan_site" }
	},
	"on": {
		"success": { "functions": ["story:caravan/found"], "tags": ["found_tracks"] }
	}
}
```

---

## Text fields and styling

The `title` and `description` fields — on both quests and tasks — accept a Minecraft JSON text component. Three forms are allowed:

```json
"title": "A plain string"
"title": {"translate": "my.key"}
"title": ["Text ", {"translate": "my.key"}, " continued"]
```

**What works:**

- Plain strings.
- Styles: `color`, `bold`, `italic`, `underlined`, `strikethrough`, `obfuscated`, `font`, `insertion`, `clickEvent`, `hoverEvent`.
- `{"translate": "..."}` — with or without a `with` field.
- `{"keybind": "..."}` — shows the player's current key.
- Nested arrays and the `extra` field.

**What doesn't work in the HUD and the book:**

| Component             | Problem        |
| --------------------- | -------------- |
| `{"score": {...}}`    | Shows up empty. |
| `{"selector": "..."}` | Shows up empty. |
| `{"nbt": "..."}`      | Shows up empty. |

These components substitute their value only in chat and commands; in the book and HUD it stays empty — a limitation of the implementation.

> To show the player **numeric progress**, don't put a `score` component in the text — use a `score` condition. It draws a progress bar directly and has none of these limitations. See [Task completion conditions](/datapack-reference/task-completion-conditions.md#score-condition-score).

---

## Unknown and extra fields

The mod doesn't choke on typos, but it doesn't guess them either. Any key not in the lists above is **ignored**, and a warning like `Unknown key "<key>" in <quest|task|…> — it will be ignored` goes to the server log — the `in …` part shows which level the extra key sits at. The mod also warns if `require` is set without `after`.

If a field you expected "did nothing", check the log for these warnings first: most often it's a typo in the field name, or nesting one level above or below where it belongs. More in [Common questions and errors](/troubleshooting/common-questions-and-errors.md).

---

## See also

- [Quick start](/getting-started/quick-start.md) — your first quest in five minutes.
- [Quest structure](/how-quests-work/quest-structure.md) — how stages and tasks add up to a quest.
- [Task completion conditions](/datapack-reference/task-completion-conditions.md) and [Task lifecycle hooks](/datapack-reference/task-lifecycle-hooks.md) — what goes inside the `condition` and `on` fields.
- [Common questions and errors](/troubleshooting/common-questions-and-errors.md) — what to do if a quest didn't load.
