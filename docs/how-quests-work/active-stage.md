# Active stage

**The active stage** is the first [stage](/how-quests-work/quest-structure.md) of the quest whose required task isn't done yet. The system doesn't lock this stage in — it recomputes it on the fly from what the player has already finished. So it shifts on its own: both when the player completes tasks and when the quest itself changes.

---

## Only the active stage is live

The system deals only with the active stage. Tasks of future and already-finished stages are "asleep" — nothing is counted for them. The active stage decides two things:

- **Conditions are checked automatically** — constantly, for all of the player's active quests, not just the pinned one. For details — see [Task completion conditions](/datapack-reference/task-completion-conditions.md).
- **The whole active stage is shown in the HUD — for a pinned quest.** All tasks of the active stage appear on screen: both the required one and the optional ones. But a quest reaches the HUD only through [pinning](/player-view/pinning.md), not because it's active — an unpinned quest isn't visible on screen, though its conditions are still checked.

---

## The stage shifts when the quest is edited

Since the stage is computed from progress, editing the quest takes effect immediately on those playing it right now. Add a new stage at the front, and players who haven't finished the quest will return to it: its required task isn't done, so it's now the earliest unfinished stage. Once they clear it, they end up right back where they left off — finished stages stay finished.

This doesn't touch completed quests. A completed quest is **frozen**: its status and progress no longer change, and edits to the quest itself don't reach it — for someone who has already finished it, it stays exactly as it ended. More on the states — see [Quest and task statuses](/how-quests-work/quest-and-task-statuses.md).

---

## See also

- [Quest structure](/how-quests-work/quest-structure.md) — stages, required and optional tasks, quest outcomes.
- [Pinning](/player-view/pinning.md) — how the active stage's tasks reach the HUD and who controls it.
- [Quest and task statuses](/how-quests-work/quest-and-task-statuses.md) — `active` / `complete` and the freezing of a completed quest.
