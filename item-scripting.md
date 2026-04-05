# Item Scripting

[← Mod Structure](mod-structure.md) | [Home](index.md) | Next: [Recipe Scripting →](recipe-scripting.md)

---

Scripts go in `42/media/scripts/` as `.txt` files. You can have multiple files — the game loads all of them. Splitting items and recipes into separate files (`items.txt`, `recipes.txt`) is a common convention.

## Basic Item Template

```
module MyMod
{
    item MyItemID
    {
        DisplayName         = My Item Display Name,
        DisplayCategory     = MyCategory,
        ItemType            = base:normal,
        Weight              = 1.0,
        Icon                = MyIcon,
    }
}
```

---

## Container Item Properties

The full set of properties relevant to bags, packs, and hauling items:

| Property | Type | Description |
|---|---|---|
| `ItemType` | string | `base:container` for anything that holds items |
| `Capacity` | int | Maximum encumbrance the container can hold. Hard-capped at 100 in vanilla B42. |
| `WeightReduction` | int | Percentage reduction applied to contents weight. `60` means contents weigh 40% of normal. |
| `RunSpeedModifier` | float | Speed multiplier when equipped. `0.5` = half speed. The game **stacks** all modifiers from all equipped/worn items additively (all differences from 1.0 are summed). |
| `RequiresEquippedBothHands` | bool | **Correct property for non-weapon items** (containers, normal items). Forces both hand slots to be occupied. The player cannot hold anything else while this is equipped. |
| `TwoHandWeapon` | bool | **Weapons only** (`ItemType = base:weapon`). Do NOT use on containers — it is silently ignored or behaves unexpectedly. Use `RequiresEquippedBothHands` instead. |
| `Weight` | float | The item's own weight when carried. |
| `Tags` | string | Semicolon-separated tags controlling what can go inside and item behavior. `base:holdgeneral` allows general items. |
| `CloseSound` | string | Sound played when closing. `CloseSack` is a safe vanilla default. |
| `OpenSound` | string | Sound played when opening. `OpenSack` is a safe vanilla default. |
| `PutInSound` | string | Sound played when placing items inside. `StoreItemSack` is a safe default. |
| `ReplaceInPrimaryHand` | string | `ModelName AnimationName` — model and animation when held in right hand. |
| `ReplaceInSecondHand` | string | Model and animation when held in left hand. |
| `WorldStaticModel` | string | Model name when the item is dropped on the ground. |
| `Icon` | string | Filename of the icon (without extension). File must be in `media/textures/item_YourIcon.png`. |

---

## Two-Handed Hauling Item Pattern

This is the standard pattern for any item meant to simulate dragging/pushing:

```
item DragSledge
{
    DisplayName               = Drag Sledge,
    DisplayCategory           = Container,
    ItemType                  = base:container,
    Weight                    = 5.0,
    Icon                      = WoodenPlank,          // placeholder until custom art
    Capacity                  = 50,
    WeightReduction           = 65,
    RunSpeedModifier          = 0.45,                 // 45% of normal speed
    RequiresEquippedBothHands = true,                 // blocks both hand slots; use this, NOT TwoHandWeapon
    CloseSound                = CloseSack,
    OpenSound                 = OpenSack,
    PutInSound                = StoreItemSack,
    Tags                      = base:holdgeneral,
}
```

**Key design levers:**
- `RunSpeedModifier` is your primary balance tool. Lower = slower = more primitive.
- `RequiresEquippedBothHands = true` is what prevents combat while hauling — the player can't hold a weapon in either hand. This is intentional game balance, not a bug.
- `Capacity` cap: vanilla B42 caps at 100. Items with very high carry needs require a Lua workaround.
- `WeightReduction` at 65–80% means the weight of contents is dramatically reduced, letting you move more than you could carry in a backpack.

---

## RunSpeedModifier Reference (The Long Haul mod)

| Item | Modifier | Effective Speed |
|---|---|---|
| Stick Travois | 0.35 | 35% — barely walking |
| Lashed Travois | 0.40 | 40% |
| Drag Sledge | 0.45 | 45% |
| Stone Wheelbarrow | 0.50 | 50% — slow jog |
| Metal-Axled Stone Wheelbarrow | 0.55 | 55% |
| Wooden Wheelbarrow | 0.65 | 65% |
| Reinforced Wheelbarrow | 0.70 | 70% |
| Shoulder Yoke | 0.85 | 85% — near-normal |
| Padded Yoke | 0.90 | 90% |

---

[← Mod Structure](mod-structure.md) | [Home](index.md) | Next: [Recipe Scripting →](recipe-scripting.md)
