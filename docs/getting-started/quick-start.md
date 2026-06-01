# Quick start

Let's build our first quest — "Escape": a required task "Find and pull the lever" and an optional one "Find out what became of the survivors". A quest here isn't just a checkbox in a list: it moves through stages and can end in success, failure, or be skipped.

There are two ways to do it — both produce the same quest:

- **With commands** — to try it out in-game right away, without creating files. This kind of quest (a dynamic quest) is saved inside the world and exists only as part of it.
- **Through a datapack** — when the quest becomes part of your content. A datapack can live in the world and, when needed, be detached and distributed on its own — as a standalone set of quests.

Start with commands if you just want to get a feel for the system.

---

## Path 1. A quest from commands

> InkQuest commands require operator level 2+. In a single-player world, turn on cheats for that.

We create the quest and its two tasks — one `required`, one `optional`:

```mcfunction
/quest new example:escape "Escape" "There's no staying here any longer — but how do I get out?"
/quest modify example:escape tasks add required push_the_lever "Find and pull the lever that opens the hatch" "When one of the guards stepped out, I caught a click somewhere at the far end of the room"
/quest modify example:escape tasks add optional look_for_survivors "Find out what became of the survivors" "I should search the cells — maybe I'll find someone else who made it out alive"
```

We give the quest to the player and pin it to the HUD:

```mcfunction
/quest give PlayerName example:escape
/quest pin PlayerName example:escape
```

After the quest is granted, the player sees a hint about the new quest, and the pinned quest appears in the HUD on screen. The full quest list opens with the `J` key — more in [Player interface](/player-view/player-interface.md).

To complete the tasks, you can hide command blocks around the map with commands like these:

```mcfunction
/quest complete PlayerName example:escape task push_the_lever success
/quest complete PlayerName example:escape task look_for_survivors success
```

> Tasks added by command are completed only by the `quest complete` command. For a task to complete on its own — say, by a scoreboard score or a Minecraft predicate — you need a datapack quest (see below and [Task completion conditions](/datapack-reference/task-completion-conditions.md)).

---

## Path 2. The same quest in a datapack

A static quest is described in JSON and lives in a datapack — you can keep it together with the world or distribute it separately. Unlike the command version, its tasks can complete on their own — by the conditions you set. Put the files in the world's datapack:

```text
<world>/datapacks/example/
├── pack.mcmeta
└── data/example/quests/escape.json
```

`pack.mcmeta` — the datapack's metadata file (`pack_format: 15` matches Minecraft 1.20.1):

```json
{
	"pack": {
		"pack_format": 15,
		"description": "Example quests"
	}
}
```

`escape.json` — the quest itself. The file name (`escape`) and the folder (`example`) together give the identifier `example:escape`:

```json
{
	"version": 3,
	"variant": 1,
	"title": "Escape",
	"description": "There's no staying here any longer — but how do I get out?",
	"tasks": {
		"push_the_lever": {
			"title": "Find and pull the lever that opens the hatch",
			"description": "When one of the guards stepped out, I caught a click somewhere at the far end of the room"
		},
		"look_for_survivors": {
			"title": "Find out what became of the survivors",
			"description": "I should search the cells — maybe I'll find someone else who made it out alive"
		}
	},
	"stages": [
		["push_the_lever", "look_for_survivors"]
	]
}
```

A few things about the structure:

- `version` and `variant` are required. They guard against loading in an incompatible mod version; the current values are `version: 3`, `variant: 1`. The breakdown is in [Quest file format](/datapack-reference/quest-file-format.md).
- `stages` is a list of stages, and each stage is a list of task IDs. **The first task in a stage is required, the rest are optional.** Here there's one stage: `push_the_lever` drives progress, `look_for_survivors` runs alongside.

Load the datapack with `/reload` and check that the quest was read:

```mcfunction
/quest list static
```

If `example:escape` is in the list — you're all set; from here the quest is granted with the same `quest give` command. If it isn't — check the server log and [Common questions and errors](/troubleshooting/common-questions-and-errors.md).

> Not every quest has to be a "task with progress". You can make a note-quest with no tasks — a journal page, a rumor, a piece of lore — or an event with shared progress. Ready-made recipes are in [Quest patterns](/quest-patterns/quest-patterns.md).

---

## What's next

The example quest is the simplest case. From here the documentation branches by interest:

- **Understand the model.** How stages and tasks add up to a playthrough — [Quest structure](/how-quests-work/quest-structure.md), [Active stage](/how-quests-work/active-stage.md), [Quest and task statuses](/how-quests-work/quest-and-task-statuses.md).
- **Automate tasks.** So tasks complete on their own — [Task completion conditions](/datapack-reference/task-completion-conditions.md) and [Task lifecycle hooks](/datapack-reference/task-lifecycle-hooks.md).
- **Tie quests into a story.** One unlocks the next — [Dependencies between quests](/how-quests-work/quest-dependencies.md) and [Quest chain](/quest-patterns/quest-chain.md).
- **Reference at hand.** All file fields — [Quest file format](/datapack-reference/quest-file-format.md), all commands — [Command reference](/command-api/command-reference.md).
- **Ready-made recipes.** Whole mechanics off the shelf — [Quest patterns](/quest-patterns/quest-patterns.md).
