# Dependencies between quests

A quest can be gated behind others: until its dependencies are met it is **not granted** to the player — it's in neither the book nor the HUD. The moment they come together, the quest is **granted on its own**, without a command. This is how a progression is built from separate quests: the next chapter opens up as the previous ones are completed.

Dependencies are declared right in the quest, through two fields: `after` — which quests it depends on — and `require` — extra conditions on the player. The types and obligation of these fields are in [Quest file format](/datapack-reference/quest-file-format.md#quest-fields); here is how they work.

---

## Dependency groups — the `after` field

`after` is a list of **groups**, where each group is a list of quests:

```json
{
	"after": [
		["story:cleared_mine", "story:met_elder"],
		["story:paid_toll"]
	]
}
```

The quest opens when **all quests in at least one group** have been completed successfully. Within a group it's AND (all are needed); between groups it's OR (any one is enough):

- `[["a", "b"]]` — both `a` and `b` are needed.
- `[["a"], ["b"]]` — either `a` or `b` is enough.
- `[["a", "b"], ["c"]]` — (`a` and `b`) or `c`.

(`a`, `b`, `c` here are the identifiers of the dependency quests.)

**Only success counts.** A dependency opens a group only if it finished with the `success` outcome. A quest that finished with a failure (`failure`) or a skip (`skipped`) does not open the group. What these outcomes are — see [Quest and task statuses](/how-quests-work/quest-and-task-statuses.md#the-three-outcomes).

---

## "Unlocked" means "granted"

A quest opened by a dependency isn't merely flagged as available — the system **grants it to the player right away**, as if by `quest give`. So it appears in the book and, if so configured, is pinned to the HUD; whether to pin it on the automatic grant is decided by the `pin_mode` field (see [Pinning](/how-quests-work/pinning.md)).

The check is **reactive**: it fires the moment one of the quest's dependencies finishes. The system doesn't poll the conditions continuously — it re-evaluates the dependent quest only when one of its dependency quests has reached success.

---

## Extra conditions — the `require` field

Beyond the completed quests, the unlock can be narrowed by conditions on the player themselves, through `require`. This is for when a quest should open not to everyone who went through the backstory, but only under special circumstances: the player carries a particular tag, or a predicate passes.

```json
{
	"after": [
		["story:prologue"]
	],
	"require": {
		"tags": ["found_secret_passage"],
		"predicate": "story:is_nighttime"
	}
}
```

- `tags` — the player's scoreboard tags. **All** of the listed ones must be present (this is AND, like within an `after` group).
- `predicate` — the identifier of a datapack predicate; tested against the player.

Both fields are optional, and all the conditions are combined with AND: the quest opens only if all the tags are present **and** the predicate passes.

`require` is checked at the same moment as `after` — when a dependency finishes. **If the conditions aren't met at that moment, there is no unlock after the fact:** the quest waits until its next dependency finishes and checks the conditions again. For a quest with a single dependency this means one single chance — whether the player manages to get the tag by the instant the dependency reaches success.

**`require` without `after` is ignored**, and the mod warns about it in the log: with no unlock by dependency there's nothing to hang the check on — `require` has no other moment to be evaluated at.

An example — open a chapter only to those who chose the dark path in the previous one:

```json
{
	"after": [["story:ch1"]],
	"require": {
		"tags": ["chose_dark_path"]
	}
}
```

Only players with the `chose_dark_path` tag will get the quest — and only if the tag is on them at the moment `story:ch1` finishes.

---

## Manual granting bypasses dependencies

`after` and `require` govern **automatic** unlocking only. A manual grant — with the [`quest give`](/command-api/command-reference.md#quest-give) command or a [quest scroll](/player-view/items.md) — hands over the quest directly, checking neither the dependencies nor the conditions. An operator can always grant a quest by hand, whether its dependencies are met or not.

---

## See also

- [Quest file format](/datapack-reference/quest-file-format.md#quest-fields) — the types and obligation of the `after` and `require` fields.
- [Quest and task statuses](/how-quests-work/quest-and-task-statuses.md#the-three-outcomes) — what "completed successfully" means and why a failure or skip doesn't count.
- [Pinning](/how-quests-work/pinning.md) — the `pin_mode` field and auto-pinning on a grant by dependency.
- [Command reference](/command-api/command-reference.md#quest-give) — granting a quest by hand, bypassing dependencies.
