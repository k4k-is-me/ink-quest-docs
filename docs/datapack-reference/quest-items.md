# Items in a datapack

The datapack and command side of the two InkQuest items: the quest book (`inkquest:quest_book`) and the quest scroll (`inkquest:quest_scroll`). How to override the book's recipe, how to bind a scroll to a specific quest, and how the `doQuestBookItemCheck` gamerule makes the book a required item. For what these items do for the player and how they're used, see [Items](/player-view/items.md).

---

## Identifiers

| Item | Identifier |
| --- | --- |
| Quest Book | `inkquest:quest_book` |
| Quest Scroll | `inkquest:quest_scroll` |

---

## Book recipe

The book is defined by an ordinary datapack recipe, `inkquest:quest_book` — shapeless, from three items:

```text
minecraft:book + minecraft:rotten_flesh + minecraft:string
```

### Overriding or disabling it

A datapack overrides the recipe by placing its own file at the same path:

```text
data/inkquest/recipes/quest_book.json
```

To **change the ingredients**, write your own `crafting_shapeless` or `crafting_shaped`.

To **remove the survival craft**, leave `minecraft:barrier` as the only ingredient: it can't be obtained without creative mode or commands, so the book becomes impossible to craft. You can still hand it out with `/give`, from a chest, or as a reward — only the crafting table is closed off.

```json
{
  "type": "minecraft:crafting_shapeless",
  "category": "misc",
  "ingredients": [
    { "item": "minecraft:barrier" }
  ],
  "result": { "item": "inkquest:quest_book", "count": 1 }
}
```

---

## Binding a scroll to a quest

A scroll is tied to a single quest through the `Quest` NBT field — the quest identifier in `namespace:id` form:

```mcfunction
/give <player> inkquest:quest_scroll{Quest:"namespace:quest_id"}
```

For example, a scroll for the quest `story:rescue_blacksmith`:

```mcfunction
/give @s inkquest:quest_scroll{Quest:"story:rescue_blacksmith"}
```

Without the `Quest` field the scroll is empty and only reports an error when clicked. The same field is set in loot tables and merchant trades — anywhere the item is given with NBT.

> Scrolls don't stack — each one occupies its own inventory slot.

What happens when a scroll is used — granting the quest, consumption, failure reasons — is on [Items](/player-view/items.md#quest-scroll).

---

## Gamerule `doQuestBookItemCheck`

```mcfunction
/gamerule doQuestBookItemCheck true|false
```

Default **false**. An ordinary Minecraft gamerule — the value is stored with the world.

When `true`, the quest book becomes a required item: until it's in the player's inventory (any slot, not necessarily in hand), quests can't be opened.

| Way to open | `false` (default) | `true` |
| --- | --- | --- |
| **J** key | Always works | Needs the book in the inventory |
| Quest Scroll | Grants the quest and opens the book | No book — quest isn't granted, scroll isn't consumed |
| Right-click on the book | Always works | Always works (the book is already in hand) |

The gamerule suits maps where quests are woven into survival: the book has to be earned first — crafted, found in a chest, or given as a reward — and only then can quests be read. Until then, granted quests pile up but stay hidden.

### Enabling it automatically via a datapack

To set the gamerule on every world load, hang the command on the `minecraft:load` function tag.

**1.** Create or extend `data/minecraft/tags/functions/load.json`:

```json
{
  "values": ["your_namespace:setup"]
}
```

**2.** Create the function `data/your_namespace/functions/setup.mcfunction`:

```mcfunction
gamerule doQuestBookItemCheck true
```

Replace `your_namespace` with your datapack's namespace. The function also runs on every `/reload` — that's safe: the gamerule is simply rewritten to the same value.

---

## See also

- [Items](/player-view/items.md) — what the book and scroll do for the player and how they're used.
- [Repeatable quests](/how-quests-work/repeatable-quests.md) — when a scroll works on an already-completed quest.
- [Command reference](/command-api/command-reference.md) — `/quest give` as an alternative to the scroll.
