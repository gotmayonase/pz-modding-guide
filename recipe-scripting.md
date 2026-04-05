# Recipe Scripting (craftRecipe)

[← Item Scripting](item-scripting.md) | [Home](index.md) | Next: [Container Mechanics →](container-mechanics.md)

---

## Full Syntax Reference

```
craftRecipe UniqueRecipeID
{
    timedAction     = Making,
    Time            = 300,
    category        = Carpentry,
    SkillRequired   = Woodwork:3;Blacksmith:1,
    xpAward         = Woodwork:60;Blacksmith:20,
    Tags            = InHandCraft,
    needToBeLearn   = false,

    inputs
    {
        item 4 [Base.Plank],
        item 8 [Base.Nails],
        item 1 tags[base:hammer] mode:keep,
        item 1 tags[base:saw] mode:keep,
    }

    outputs
    {
        item 1 MyMod.MyItem,
    }
}
```

---

## Parameters

| Parameter | Format | Notes |
|---|---|---|
| `timedAction` | `= Making` | Animation to play during craft. Common values: `Making`, `SawLog`, `Carpentry`. |
| `Time` | `= 300` | Duration in **tenths of a second**. 300 = 30 seconds. |
| `category` | `= Carpentry` | Groups the recipe in the crafting UI. |
| `SkillRequired` | `= Woodwork:3` | Skill gate. Semicolon-separated for multiple: `Woodwork:3;Metalwork:1`. Player needs ALL listed skills. |
| `xpAward` | `= Woodwork:60` | XP granted on completion. Semicolon-separated for multiple skills. |
| `Tags` | `= InHandCraft` | Flags. `InHandCraft` means recipe appears in the standard crafting menu. |
| `needToBeLearn` | `= false` | `true` = recipe requires finding a recipe book first. `false` = available by default. |
| `OnCreate` | `= Recipe.OnCreate.MyFunc` | Lua function called after craft completes. |
| `OnTest` | `= Recipe.OnTest.MyFunc` | Lua function called to test if recipe is available (return bool). |

---

## inputs Block

```
inputs
{
    // Consumed (destroyed) — default behavior
    item 4 [Base.Plank],

    // Tool — not consumed
    item 1 tags[base:hammer] mode:keep,

    // Specific item, not consumed
    item 1 [Base.Nails] mode:keep,

    // Any item matching a tag, not consumed, with flags
    item 1 tags[base:saw] mode:keep flags[MayDegradeLight],
}
```

**Critical rule:** Items are **consumed (destroyed) by default**. You must add `mode:keep` for tools and anything that shouldn't be used up.

**Tag matching** (`tags[base:tagname]`) accepts any item in the game that has that tag. This is how recipes accept "any hammer" rather than a specific hammer item ID. Use tags for tools wherever possible — it's more compatible with other mods that add new tool types.

---

## outputs Block

```
outputs
{
    item 1 MyMod.MyItem,        // one item
    item 3 Base.Plank,          // multiple of same item
}
```

Note: outputs use `Module.ItemID` **without brackets**, unlike inputs which use `[Module.ItemID]` with brackets.

---

## Upgrade Path Pattern

To create a recipe that consumes a previous tier as a base component, just list it as a normal consumed input:

```
craftRecipe UpgradeToWoodenWheelbarrow
{
    ...
    inputs
    {
        item 1 [MyMod.DragSledge],          // consumed — becomes the wheelbarrow base
        item 1 [MyMod.CarvedWoodWheel],      // consumed — the wheel
        item 2 [Base.Plank],
        item 6 [Base.Nails],
        item 1 tags[base:hammer] mode:keep,
    }

    outputs
    {
        item 1 MyMod.WoodenWheelbarrow,
    }
}
```

The player will feel like they "upgraded" their drag sledge into a wheelbarrow because the sledge is consumed in the process — their earlier work carries forward.

---

## Skill Names

These are the internal skill ID strings used in `SkillRequired` and `xpAward`. All verified against B42 vanilla script files (`recipes_stonemasonry_i.txt`, `recipes_carving.txt`, `recipes_blacksmith_*.txt`, `recipes_carpentry.txt`, `recipes_tailoring.txt`):

| Display Name | Internal ID | Notes |
|---|---|---|
| Carpentry | `Woodwork` | Used in both `category` and `SkillRequired` |
| Metalworking / Blacksmithing | `Blacksmith` | **Not** `Metalwork` — a common wrong guess |
| Masonry | `Masonry` | Matches display name exactly |
| Carving | `Carving` | Matches display name exactly |
| Tailoring | `Tailoring` | Matches display name exactly |
| Farming | `Farming` | Matches display name exactly |
| First Aid | `Doctor` | Does **not** match display name |
| Aiming | `Aiming` | Matches display name exactly |

> **Common mistake:** Using `Metalwork` instead of `Blacksmith`. The skill is displayed as "Metalworking" in-game but the internal ID is `Blacksmith`. If your metalwork recipes silently fail to gate on skill level, this is why.

---

[← Item Scripting](item-scripting.md) | [Home](index.md) | Next: [Container Mechanics →](container-mechanics.md)
