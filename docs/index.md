# InkQuest — Narrative Quest Engine

InkQuest is a quest system for Minecraft 1.20.1 (Fabric) built around narrative gameplay. Here a quest is a story event that unfolds: it moves through stages and can end in success, failure, or be skipped. Everything is configured through JSON datapacks and commands — no GUI editor.

Unlike most quest systems, InkQuest is not limited to task types like "kill X mobs". Any mechanic is expressed through scoreboard conditions, Minecraft predicates, and datapack functions. Quests are versioned alongside your project, shipped inside a datapack, and integrate deeply with command blocks.

---

## Where to start

First time here? Open [Quick start](/getting-started/quick-start.md) and build your first quest in five minutes. Beyond that, the docs are not meant to be read front to back: each section answers its own kind of question. Find your column below.

---

## Contents

### Getting started

- [Quick start](/getting-started/quick-start.md) — your first quest, via commands and via datapack.

### How quests work

*Understand the model — what a quest actually is and the rules it lives by.*

- [Quest structure](/how-quests-work/quest-structure.md) — stages, tasks, outcomes.
- [Active stage](/how-quests-work/active-stage.md) — which stage is "now" and how it advances.
- [Quest and task statuses](/how-quests-work/quest-and-task-statuses.md) — active, complete, success/failure/skipped.
- [Quest dependencies](/how-quests-work/quest-dependencies.md) — how one quest unlocks another (`after`, `require`).
- [Repeatable quests](/how-quests-work/repeatable-quests.md) — a quest you can run again.
- [Pinning](/how-quests-work/pinning.md) — how a quest reaches the HUD and who controls it.

### Datapack reference

*Exact JSON fields — to look up "how to write it precisely".*

- [Quest file format](/datapack-reference/quest-file-format.md) — full list of quest and task fields, format versions, text and styles.
- [Task completion conditions](/datapack-reference/task-completion-conditions.md) — auto-completion: `score`, `predicate`, `all`/`any`/`none`, `tasks`, buttons.
- [Task lifecycle hooks](/datapack-reference/task-lifecycle-hooks.md) — running functions and granting tags on task events.
- [Items in a datapack](/datapack-reference/quest-items.md) — book recipe, binding a scroll to a quest, the `doQuestBookItemCheck` gamerule.

### Player view

- [Player interface](/player-view/player-interface.md) — HUD, quest book, notifications.
- [Items](/player-view/items.md) — the quest book and quest scroll, opening gamerule.

### Command API

- [Command reference](/command-api/command-reference.md) — every `/quest` and `/execute if quest` command.
- [Integration with commands](/command-api/integration-with-commands.md) — a quest as readable state; wiring with command blocks and scoreboard.

### Quest patterns

- [Quest patterns](/quest-patterns/quest-patterns.md) — a journal note, a chain, a branch, a shared-progress event, and more.

### Troubleshooting

- [Common questions and errors](/troubleshooting/common-questions-and-errors.md) — what to do if a quest fails to load or a condition doesn't fire.
