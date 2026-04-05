# Key Gotchas and Verified IDs

[← Multiplayer Architecture](multiplayer.md) | [Home](index.md) | Next: [Testing Workflow →](testing.md)

---

All entries below were verified directly against B42 game script files in `media/scripts/generated/` and `media/lua/`. Source files noted where relevant.

---

## Verified Vanilla Item IDs

| In-game name | Correct ID | Notes |
|---|---|---|
| Tree Branch | `Base.TreeBranch2` | B42 renamed from `Base.TreeBranch`. Foraged via Foraging; used as input in all vanilla crafting recipes. |
| Wooden Rod | `Base.WoodenStick2` | Display name is "Wooden Rod". Carved from a branch via `craftRecipe CarveStick` (Carving:1). Not to be confused with `Base.SmallHandle`. |
| Twigs | `Base.Twigs` | Small twig bundle; fire tinder |
| Twine | `Base.Twine` | `ItemType = base:drainable` — a spool with uses, not a count. Tag: `base:twine` |
| Sheet | `Base.Sheet` | The torn bedsheet / fabric sheet |
| Ripped cloth scraps | `Base.RippedSheets` | **Not** `Base.Rag` — that ID does not exist in B42 |
| Log | `Base.Log` | Standard chopped log |
| Plank | `Base.Plank` | Standard wooden plank |
| Nails | `Base.Nails` | |
| Scrap Metal | `Base.ScrapMetal` | |
| Metal Pipe | `Base.Pipe` | **Not** `Base.MetalPipe` — the item is just `Pipe` |
| Stone Wheel | `Base.StoneWheel` | Masonry-crafted large stone wheel. Also: `Base.StoneWheelSmall`. Both have `RequiresEquippedBothHands = true`. |
| Small Handle | `Base.SmallHandle` | Short wooden rod/handle. 4 produced from one `WoodenStick2` via `SawWoodenRod`. Different from Wooden Rod. |

**B42 Renaming Pattern:** Several B41 item IDs gained a `2` suffix in B42 as the item was reworked into a weapon-type item:
- `Base.TreeBranch` → `Base.TreeBranch2`
- `Base.WoodenStick` → `Base.WoodenStick2`

If a B41 item ID returns nothing in B42, try appending `2`.

---

## Verified Skill Names

| Display name | `SkillRequired` / `xpAward` ID |
|---|---|
| Carpentry | `Woodwork` |
| Metalworking | `Blacksmith` (**not** `Metalwork`) |
| Masonry | `Masonry` |
| Carving | `Carving` |
| Tailoring | `Tailoring` |

Source: `recipes_blacksmith_armor.txt`, `recipes_stonemasonry_i.txt`, `recipes_carving.txt`, `recipes_carpentry.txt`, `recipes_tailoring.txt`.

---

## Verified Recipe Tags (Tools)

Tags are used in `inputs` to accept any item matching a capability, rather than a specific item ID. All confirmed against vanilla recipes:

| Tool needed | Tag to use |
|---|---|
| Any hammer | `tags[base:hammer]` |
| Any saw | `tags[base:saw]` |
| Sharp knife / cleaver | `tags[base:sharpknife]` (**not** `base:knifesharp`) |
| Sewing needle | `tags[base:sewingneedle]` (**not** `base:needle`) |
| Any thread/binding | `tags[base:binding]` or `tags[base:twine]` |
| Fishing line | `tags[base:fishingline]` |

Source: `recipes_carving.txt`, `recipes_tailoring.txt`.

---

## Verified Script Properties

**`RequiresEquippedBothHands` vs `TwoHandWeapon`** — the most important distinction for container mods:

| Property | Works on | Effect |
|---|---|---|
| `RequiresEquippedBothHands = true` | Any item type (containers, normal items) | Occupies both hand slots; player cannot hold anything else |
| `TwoHandWeapon = TRUE` | Weapons only (`ItemType = base:weapon`) | Same effect, but only applies to weapon-type items |

Using `TwoHandWeapon` on a container will not work as expected — use `RequiresEquippedBothHands` for any non-weapon item.

Both are checked together in the unequip Lua: `item:isTwoHandWeapon() or item:isRequiresEquippedBothHands()` (from `ISUnequipAction.lua`).

**`setPrimaryHandItem(nil)` and `setSecondaryHandItem(nil)`** — confirmed correct server-side unequip methods in B42. Used in vanilla: `ISUnequipAction.lua` lines 119–127, `STrapGlobalObject.lua` lines 256–257.

**`player:isAiming()`** — confirmed B42 method. Returns true when the player is in aim mode (right-click held with a firearm). Will never return true when a `RequiresEquippedBothHands` container is equipped, since there's no weapon in hand to aim.

---

## Common Mistakes

**Item not appearing in game:** Check that your module declaration is correct and the file is in `42/media/scripts/`. Also check the game's console log — script parse errors are printed there.

**Recipe not appearing:** The `category` value must match a valid crafting category. Common values: `Carpentry`, `Tailoring`, `Blacksmith`, `Masonry`, `Carving`, `Cooking`, `Farming`. Wrong category = recipe still exists but may be hard to find or not show at all.

**Metalwork recipes silently not gating:** You almost certainly wrote `Metalwork` instead of `Blacksmith`. The in-game skill is called "Metalworking" but the internal ID is `Blacksmith`.

**Items being destroyed when they should be kept:** Forgot `mode:keep` on a tool input. All inputs are consumed by default.

**Container not blocking both hands:** Used `TwoHandWeapon = TRUE` on a container. Switch to `RequiresEquippedBothHands = true`.

**Twine always running out mid-recipe:** `Base.Twine` is a drainable item — it has a `UseDelta` that depletes it. If your recipe asks for `item 3 [Base.Twine]`, it expects 3 separate Twine items, not 3 uses from one. Consider using `tags[base:twine]` if you want any twine-tagged item instead.

**Mod not loading at all:** Check `mod.info` is inside `42/` (not at the root). Check `id` field has no spaces.

**Old B41 mod not working in B42:** The three big breaks — folder structure, `Type =` vs `ItemType =`, and `recipe {}` vs `craftRecipe {}` — cover 95% of cases.

---

[← Multiplayer Architecture](multiplayer.md) | [Home](index.md) | Next: [Testing Workflow →](testing.md)
