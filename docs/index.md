# InkQuest — Narrative Quest Engine

InkQuest is a quest system for Minecraft 1.20.1 (Fabric) built around narrative gameplay. Here a quest is a story event that unfolds: it moves through stages and can end in success, failure, or be skipped. Everything is configured through JSON datapacks and commands — no GUI editor.

Unlike most quest systems, InkQuest is not limited to task types like "kill X mobs". Any mechanic is expressed through scoreboard conditions, Minecraft predicates, and datapack functions. Quests are versioned alongside your project, shipped inside a datapack, and integrate deeply with command blocks.

---

## Where to start

First time here? Open [QUICK START](/getting-started/quick-start.md) and build your first quest in five minutes. Beyond that, the docs are not meant to be read front to back: each section answers its own kind of question. Find your column below.

---

## Contents

### Getting started

- [QUICK START](/getting-started/quick-start.md) — your first quest, via commands and via datapack.

### How quests work

*Understand the model — what a quest actually is and the rules it lives by.*

- [QUEST STRUCTURE](/how-quests-work/quest-structure.md) — stages, tasks, outcomes.
- [ACTIVE STAGE](/how-quests-work/active-stage.md) — which stage is "now" and how it advances.
- [QUEST AND TASK STATUSES](/how-quests-work/quest-and-task-statuses.md) — active, complete, success/failure/skipped.
- [DEPENDENCIES BETWEEN QUESTS](/how-quests-work/quest-dependencies.md) — how one quest unlocks another (`after`, `require`).
- [REPEATABLE QUESTS](/how-quests-work/repeatable-quests.md) — a quest you can run again.

### Datapack reference

*Exact JSON fields — to look up "how to write it precisely".*

- [QUEST FILE FORMAT](/datapack-reference/quest-file-format.md) — full list of quest and task fields, format versions, text and styles.
- [TASK COMPLETION CONDITIONS](/datapack-reference/task-completion-conditions.md) — auto-completion: `score`, `predicate`, `all`/`any`/`none`, `tasks`, buttons.
- [TASK LIFECYCLE HOOKS](/datapack-reference/task-lifecycle-hooks.md) — running functions and granting tags on task events.

### Player view

- [PLAYER INTERFACE](/player-view/player-interface.md) — HUD, quest book, notifications.
- [PINNING](/player-view/pinning.md) — how a quest reaches the HUD and who controls it.
- [ITEMS](/player-view/items.md) — the quest book and quest scroll, opening gamerule.

### Command API

- [COMMAND REFERENCE](/command-api/command-reference.md) — every `/quest` and `/execute if quest` command.
- [INTEGRATION WITH COMMANDS](/command-api/integration-with-commands.md) — a quest as readable state; wiring with command blocks and scoreboard.

### Ready-made recipes

- [QUEST PATTERNS](/quest-patterns/quest-patterns.md) — a journal note, a chain, a branch, a shared-progress event, and more.

### Troubleshooting

- [COMMON QUESTIONS AND ERRORS](/troubleshooting/common-questions-and-errors.md) — what to do if a quest fails to load or a condition doesn't fire.
