# Выдача квестов новому игроку

> Игрок зашёл на сервер впервые — и сразу получил стартовый квест (или несколько). Без admin-команд, без участия оператора, без модов: чистая ваниль.

Подход универсальный: помимо квестов в той же функции можно выдать стартовые предметы, выставить теги, завести scoreboard'ы, привязать к стартовой локации — всё, что нужно подготовить для нового игрока.

---

## Зачем нужен этот паттерн

- **Самозапуск сюжета.** Игрок зашёл — и уже в истории, без «вызовите оператора, чтобы получить пролог».
- **Универсальная точка входа.** Одна функция готовит игрока к игре: квесты, инвентарь, теги, точка спавна, ачивки. Менять условия старта — править одну функцию.
- **Полностью ванильный механизм.** Никаких модов, никаких зависимостей от InkQuest при первом запуске — только датапак, который и так нужен под квесты.

---

## Как это работает в двух предложениях

Создаётся advancement с тривиальным trigger'ом `minecraft:tick` — он выдаётся каждому игроку на первом же тике после входа. В награду advancement вызывает mcfunction, в которой ты делаешь что хочешь: `quest give @s ...`, `give @s ...`, `tag @s add ...` и т.д.

Поскольку advancement выдаётся **один раз на игрока навсегда**, функция сработает ровно один раз — при первом подключении.

---

## Шаг 1. Пишем функцию подготовки игрока

`data/story/functions/on_player_setup.mcfunction`:

```mcfunction
quest give @s story:prologue
quest give @s story:field_guide
quest give @s story:daily_hunt
```

Это и есть «приветственный набор» — три квеста, которые получит каждый новичок. Сюда же можно дописать что угодно:

```mcfunction
quest give @s story:prologue
give @s minecraft:bread 5
give @s minecraft:wooden_sword
tag @s add story.newcomer
scoreboard players set @s story.reputation 0
spawnpoint @s -120 64 340
```

Функция вызывается **от имени игрока** — `@s` внутри неё — это именно тот игрок, который только что зашёл.

---

## Шаг 2. Создаём advancement-триггер

`data/story/advancements/on_join.json`:

```json
{
  "criteria": {
    "tick": {
      "trigger": "minecraft:tick"
    }
  },
  "rewards": {
    "function": "story:on_player_setup"
  }
}
```

Разберём по строкам:

- `criteria.tick.trigger: "minecraft:tick"` — единственный критерий, без условий. Сработает на первом же тике после захода игрока.
- `rewards.function: "story:on_player_setup"` — путь к функции из шага 1 (namespace + имя без `.mcfunction`).

Готово. Этот advancement никак не отображается игроку (нет `display`-блока), он чисто служебный.

---

## Шаг 3. Проверка

1. Зайди на сервер новым аккаунтом (или после `/advancement revoke @s only story:on_join` — об этом ниже).
2. На первом же тике в книге квестов должны появиться `story:prologue`, `story:field_guide`, `story:daily_hunt`.
3. Если что-то не появилось:
   - Файл advancement лежит в `data/story/advancements/on_join.json`? ID будет `story:on_join` — namespace = имя папки в `data/`.
   - Файл функции лежит в `data/story/functions/on_player_setup.mcfunction`? Путь в `rewards.function` должен совпадать.
   - Датапак вообще загружен? `/datapack list` должен показать `trail-of-elara` (или твой).

---

## Как перезапустить setup для уже существующих игроков

Сценарий: ты обновил функцию `on_player_setup` (например, добавил новый стартовый квест), и хочешь, чтобы старые игроки тоже его получили.

Поскольку advancement выдаётся один раз навсегда, его нужно сначала отозвать:

```
/advancement revoke @a only story:on_join
```

На следующем тике advancement снова выдастся (trigger срабатывает каждый тик у тех, у кого его нет), и функция запустится у всех онлайн-игроков. Оффлайн-игроки получат его при следующем входе.

> ⚠️ Это вызовет функцию **повторно** — если в ней есть `quest give`, повторная выдача активного квеста выдаст ошибку `already_tracked`. Чтобы повтор был безопасным — либо проверяй состояние перед выдачей, либо делай новые квесты [repeatable](/quest-patterns/repeatable-quest.md).

### Защита от повторного запуска внутри функции

Если хочешь явный «однократный» setup, добавь scoreboard-флаг:

```mcfunction
# data/story/functions/on_player_setup.mcfunction
scoreboard objectives add story.setup_done dummy
execute if score @s story.setup_done matches 1 run return 0

# реальный setup ниже
quest give @s story:prologue
give @s minecraft:bread 5
scoreboard players set @s story.setup_done 1
```

Теперь функция начинается с проверки: если флаг уже стоит — возврат без действий. Полезно, если ты собираешься периодически revoke'ать advancement для регрессии тестов или ивентов.

---

## Расширения

### Раздать разные стартовые наборы по выбору игрока

Если перед стартом игрок должен выбрать класс/фракцию — раздели setup на два advancement'а:

1. `story:on_join` — выдаёт «нейтральный» стартовый набор и [квест выбора](/quest-patterns/branch-and-choice-memory.md).
2. `story:on_class_chosen` — без trigger'а; выдаётся **функцией** из `on.success` задачи выбора (`advancement grant @s only story:on_class_chosen`). В его `rewards.function` — раздача классового снаряжения и стартового квеста класса.

`data/story/advancements/on_class_chosen.json`:

```json
{
  "criteria": {
    "impossible": {
      "trigger": "minecraft:impossible"
    }
  },
  "rewards": {
    "function": "story:on_class_chosen"
  }
}
```

Trigger `minecraft:impossible` никогда не срабатывает сам — advancement можно выдать только командой `advancement grant`. Это даёт ручной контроль над моментом запуска `rewards.function`.

`data/story/functions/on_class_chosen.mcfunction`:

```mcfunction
execute if entity @s[tag=class.warrior] run give @s minecraft:iron_sword
execute if entity @s[tag=class.warrior] run quest give @s story:warrior_prologue
execute if entity @s[tag=class.mage] run give @s minecraft:enchanted_book
execute if entity @s[tag=class.mage] run quest give @s story:mage_prologue
```

Так старт делится на два этапа: общий setup и классовый, оба запускаются автоматически.

### Только для определённого мира

Если стартовый набор должен выдаваться только в одном измерении (например, в стартовой локации, а не в Незере) — добавь в advancement player-predicate:

```json
{
  "criteria": {
    "tick": {
      "trigger": "minecraft:tick",
      "conditions": {
        "player": [
          {
            "condition": "minecraft:entity_properties",
            "entity": "this",
            "predicate": {
              "location": {
                "dimension": "minecraft:overworld"
              }
            }
          }
        ]
      }
    }
  },
  "rewards": {
    "function": "story:on_player_setup"
  }
}
```

Теперь advancement выдастся только когда игрок впервые окажется в Overworld.

### Сезонные стартовые квесты

Хочешь, чтобы во время серверного «зимнего ивента» новые игроки получали «зимний» пролог, а в остальное время — обычный? Заведи серверный флаг сезона и проверяй его в setup-функции.

`data/story/functions/on_player_setup.mcfunction`:

```mcfunction
execute if predicate story:winter_event_active run quest give @s story:winter_prologue
execute unless predicate story:winter_event_active run quest give @s story:prologue
```

`data/story/predicates/winter_event_active.json`:

```json
{
  "condition": "minecraft:value_check",
  "value": {
    "type": "minecraft:score",
    "target": { "type": "minecraft:fixed", "name": "SERVER" },
    "score": "story.season",
    "scale": 1.0
  },
  "range": 1
}
```

Админ переключает сезон вручную: `scoreboard players set SERVER story.season 1` (зима) или `scoreboard players set SERVER story.season 0` (обычно). Vanilla Minecraft не умеет проверять реальную дату — для этого нужна привязка к серверному флагу или внешнему скрипту.

### «Догон» для постоянных игроков

Когда добавляешь новый стартовый квест в `on_player_setup`, постоянным игрокам он сам не выдастся. Чтобы автоматически «догнать» их при следующем входе — заведи **второй** advancement с уникальным ID для каждого обновления:

`data/story/advancements/setup_v2.json`:

```json
{
  "criteria": { "tick": { "trigger": "minecraft:tick" } },
  "rewards": { "function": "story:setup_v2" }
}
```

`data/story/functions/setup_v2.mcfunction`:

```mcfunction
quest give @s story:new_chapter
tag @s add story.got_chapter_v2
```

Все, кто уже играл, получат `setup_v2` при следующем заходе. Новые игроки получат и `on_join`, и `setup_v2` подряд.

### Стартовый ачив как видимая «глава»

Если хочешь, чтобы игрок видел: «История началась» — добавь display-блок:

```json
{
  "display": {
    "icon": { "item": "minecraft:writable_book" },
    "title": { "translate": "advancement.story.welcome" },
    "description": { "translate": "advancement.story.welcome.desc" },
    "background": "minecraft:textures/gui/advancements/backgrounds/stone.png",
    "show_toast": true,
    "announce_to_chat": false
  },
  "criteria": { "tick": { "trigger": "minecraft:tick" } },
  "rewards": { "function": "story:on_player_setup" }
}
```

Игрок получит ванильный тост «Достижение получено: …» — приятная отметка начала истории.

---

## Подводные камни

- **Если функция падает с ошибкой — advancement всё равно засчитан.** Minecraft не делает «отката»: даже если в функции была опечатка и она вылетела на полпути, advancement выдан, и при следующем заходе функция не запустится. Тестируй функцию руками (`/function story:on_player_setup`) перед тем, как ставить её в setup.
- **Trigger `minecraft:tick` срабатывает у admin'ов и зрителей тоже.** Spectator и creative тоже получают advancement. Если стартовый набор не нужен админам — фильтруй внутри функции через тег: `execute unless entity @s[tag=admin] run quest give @s story:prologue`.
- **Перезагрузка датапака (`/reload`) не «пересдаёт» advancement.** Если ты только что добавил advancement, а игрок уже на сервере — ему нужно либо релог, либо `/advancement grant @s only story:on_join`.
- **Квесты, которые ещё не загружены, выдать нельзя.** Если `on_player_setup` запускается в момент, когда InkQuest ещё не подгрузил квесты (теоретически возможно на самой первой загрузке мира), `quest give` выдаст ошибку. На практике это не встречается, потому что первый тик игрока приходит уже после загрузки сервера и датапаков.
- **`on_player_setup` — не для тяжёлых операций.** Это однократный setup, но если ты решишь «заодно» считать что-то про всех онлайн-игроков, помни: функция выполняется от имени `@s` (новичка), а не админа. `@e[type=player]` сработает только у тех, кого новичок «видит» в зоне прогрузки.

---

## См. также

- [ЦЕПОЧКА КВЕСТОВ](/quest-patterns/quest-chain.md) — после стартового квеста зависимые откроются автоматически
- [РАЗВИЛКА И ПАМЯТЬ ВЫБОРА](/quest-patterns/branch-and-choice-memory.md) — если стартовый квест предлагает выбор класса/фракции
- [ГЛОБАЛЬНЫЙ КВЕСТ](/quest-patterns/global-quest.md) — если стартовый квест включает игрока в текущий серверный ивент
- [ХУКИ ЖИЗНЕННОГО ЦИКЛА ЗАДАЧИ](/datapack-reference/task-lifecycle-hooks.md) — функции и теги в `on.load`/`on.success` тоже могут готовить игрока
