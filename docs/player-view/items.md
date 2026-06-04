# Items

The two InkQuest items the player uses to work with quests straight from the inventory: the Quest Book — to read granted quests and track progress — and the Quest Scroll — to receive a specific quest on right-click. This page is about how the player uses them. Their datapack and command side (recipe, scroll NBT, gamerule) is in [Items in a datapack](/datapack-reference/quest-items.md).

---

## Quest Book

Opens the quest book — the full list of granted quests with their tasks and progress. The item itself is not consumed: it's a reusable "key" to the interface. For what's inside, see [Player interface](/player-view/player-interface.md).

### How to get it

Crafted shapeless from three items:

```text
minecraft:book + minecraft:rotten_flesh + minecraft:string
```

A map author can change the recipe or remove the craft entirely — see [Items in a datapack](/datapack-reference/quest-items.md#book-recipe).

### How to open it

- With the **J** key (rebindable in controls).
- By right-clicking while holding the book.

On some maps the book is a required item: until it's in your inventory, quests can't be opened — neither with the key nor with a scroll. This is turned on by the `doQuestBookItemCheck` gamerule — see [Items in a datapack](/datapack-reference/quest-items.md#gamerule-doquestbookitemcheck).

---

## Quest Scroll

A consumable item bound to a single quest. Right-clicking grants that quest to the player and opens the book on it right away — handy as a reward for another quest, a merchant's wares, or a chest find.

### How to get it

There is no crafting recipe. A scroll only reaches the player through a datapack or commands: loot tables, chests, merchants, the `/give` command. How to bind a scroll to a specific quest is in [Items in a datapack](/datapack-reference/quest-items.md#binding-a-scroll-to-a-quest).

Scrolls don't stack — each one occupies its own inventory slot.

### How to use it

Right-click while holding the scroll. If the quest is granted, the scroll is consumed (in creative mode it isn't consumed) and the book opens on the new quest.

If the quest can't be granted, the scroll stays in hand and a hint with the reason appears on the action bar. That happens when the quest is already active, already completed (except [repeatable](/how-quests-work/repeatable-quests.md) ones), or when the map requires a book the player doesn't have.

---

## See also

- [Player interface](/player-view/player-interface.md) — what the book and HUD look like inside.
- [Pinning](/how-quests-work/pinning.md) — what shows in the HUD.
- [Items in a datapack](/datapack-reference/quest-items.md) — the book recipe, the scroll's NBT, the `doQuestBookItemCheck` gamerule.
