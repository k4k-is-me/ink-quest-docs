# Player interface

Players interact with quests through three surfaces: the **HUD** on screen, the **Quest Book** with the full list, and a pop-up **new quest notification**.

---

## HUD — quests on screen

The HUD shows pinned quests directly over the game without opening any windows. For each pinned quest you can see:

- the quest title;
- all tasks of its current stage — the required task and any optional ones;
- a progress bar for each task that has a numeric condition (such as `score` or `all`).

Multiple quests can be pinned at the same time. The HUD is animated: quests fade in and out smoothly, old tasks slide out and new ones slide in when the stage changes, and a short animation plays when a task completes.

What appears in the HUD is entirely controlled by pinning. See [Pinning](/how-quests-work/pinning.md) for details.

---

## Quest Book

The Quest Book is the player's complete quest list. Unlike the HUD, it shows **all** tracked quests, not just pinned ones.

### How to open it

- Press **J** (configurable in Controls).
- Right-click a Quest Book item in your hand.

Both methods may require having the book in your inventory — this depends on gamerule `doQuestBookItemCheck`. See [Items](/player-view/items.md).

### Quest list

The list is split into two sections: **active** and **completed**. Each quest is a card with an icon and title; order is set by the `index` field (see [Quest file format](/datapack-reference/quest-file-format.md)). Active quests that the player hasn't opened yet show a small **unviewed indicator** next to the icon.

### Quest details

Selecting a quest from the list opens its detail page: the quest description and all tasks with their progress. From here the player can **pin** a task to the HUD without leaving the book (unless gamerule `allowManualQuestPin` disables this — see [Pinning](/how-quests-work/pinning.md)). If a task has manual completion buttons (`buttons`), they appear here too. See [Task completion conditions](/datapack-reference/task-completion-conditions.md#manual-completion-buttons-buttons).

Once the player opens a quest's details, the unviewed indicator for that quest disappears.

---

## New quest notification

A quest can be given to a player without them noticing — it's not pinned in the HUD and the book is closed. To prevent the new content from going unnoticed, a notification pops up in the corner of the screen: "You have new quests — press **J**".

Multiple quests given in quick succession are grouped into a single notification over a short window (a few seconds) to avoid spam. If there is exactly one new quest, pressing **J** opens the book directly on it.

The notification disappears on its own and doesn't appear for quests the player has already seen: those pinned in the HUD or given via a Quest Scroll (the scroll opens the book on the quest automatically — see [Items](/player-view/items.md)). Opening the book resets the notification entirely.

> The notification is a purely client-side visibility hint. It has no effect on quest logic: not on progress, not on pinning.

---

## See also

- [Pinning](/how-quests-work/pinning.md) — controlling what appears in the HUD.
- [Items](/player-view/items.md) — the Quest Book and Quest Scroll as items, the open gamerule.
- [Quest and task statuses](/how-quests-work/quest-and-task-statuses.md) — what the "active" and "completed" sections mean.
