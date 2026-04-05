# Project Zomboid Modding Guide (Build 42)

A practical, ground-up guide to modding Project Zomboid targeting **Build 42.15+** (unstable branch). Written as actual mod development happened — expect real examples, verified techniques, and honest notes about what still needs confirming in-game.

> **Why this guide exists:** Official documentation for B42 is sparse. The PZwiki lags behind. B42 made sweeping changes to the modding API that broke most B41 tutorials. This guide is an attempt to document the current reality.

---

## Table of Contents

1. [Build 42 vs Build 41 — What Changed](#build-42-vs-build-41--what-changed)
2. [Mod Folder Structure](#mod-folder-structure)
3. [Item Scripting](#item-scripting)
4. [Recipe Scripting (craftRecipe)](#recipe-scripting-craftrecipe)
5. [Container Mechanics](#container-mechanics)
6. [Visual Assets — Icons and Models](#visual-assets--icons-and-models)
7. [Lua Scripting](#lua-scripting)
8. [Multiplayer Architecture](#multiplayer-architecture)
9. [Key Gotchas and Verified IDs](#key-gotchas-and-verified-ids) — item IDs, skill names, tags, script properties
10. [Testing Workflow](#testing-workflow)
11. [Resources](#resources)

---

## Build 42 vs Build 41 — What Changed

If you find a modding tutorial online, there's a good chance it's for Build 41. B42 introduced several **breaking changes** that will silently fail or crash your mod if you follow old guides.

### The Three Big Breaks

#### 1. Mod Folder Structure
B41 mods had a flat `media/` folder at the mod root. B42 introduced versioned subfolders.

```
# B41 (old)
MyMod/
└── media/
    └── scripts/

# B42 (new — required)
MyMod/
└── 42/
    └── media/
        └── scripts/
```

The game now only loads files from the versioned folder matching the current build. You can support both builds simultaneously with separate `41/` and `42/` folders, but **if you only have `media/` at the root, your mod will not load in B42.**

One exception: a `common/` folder at the same level as `42/` is loaded by both builds. Use it for Lua that works on both. For any new B42-only mod, ignore `common/` and just use `42/`.

#### 2. Item Type Declaration
Every item must declare what kind of item it is. The property name and format changed completely.

```
# B41 (old — will break in B42)
Type = Container

# B42 (new — required)
ItemType = base:container
```

Common `ItemType` values:
- `base:container` — holds other items
- `base:normal` — generic item (materials, components)
- `base:food` — food item
- `base:weapon` — weapon
- `base:clothing` — wearable item

#### 3. Recipe System
The entire recipe system was replaced. Old `recipe { }` blocks no longer work.

```
# B41 (old — will not work in B42)
recipe Make Wooden Plank {
    TreeBranch=2,
    keep Saw,
    Result:WoodenPlank,
    Time:50.0,
    Category:Carpentry,
    SkillRequired:Woodwork=2,
}

# B42 (new)
craftRecipe MakeWoodenPlank
{
    timedAction   = Making,
    Time          = 50,
    category      = Carpentry,
    SkillRequired = Woodwork:2,

    inputs
    {
        item 2 [Base.TreeBranch],
        item 1 tags[base:saw] mode:keep,
    }

    outputs
    {
        item 1 Base.WoodenPlank,
    }
}
```

### Minor Breaking Changes by Sub-Version

#### 42.15
- **Translation files changed format.** Old: `.txt` files with language code in filename (`ItemName_EN.txt`). New: `.json` files without language code (`ItemName.json`), all UTF-8. Any mod with the old format will have broken translations in 42.15+.
- Fixed: mod folder detection — only one of `common/` or `42/` needs to exist.
- Added: `.glb` model file support in mod watcher.

#### 42.14
- `Base.223Bullets`, `Base.BoxBullets223`, `Base.CartonBullets223` removed. Replaced with 5.56 equivalents. Only affects ammo mods.
- New thread-equivalent items added: `Base.DentalFloss`, `Base.FishingLine` now accepted by `tags[base:thread]`.

---

## Mod Folder Structure

### Minimal Structure (B42 only)

```
MyMod/
├── workshop.txt                          ← Steam Workshop metadata
├── preview.png                           ← Workshop thumbnail
└── Contents/
    └── mods/
        └── MyMod/
            └── 42/
                ├── mod.info              ← Required. Mod identity.
                ├── poster.png            ← Optional thumbnail inside mod
                └── media/
                    ├── scripts/          ← Item + recipe .txt files
                    ├── lua/
                    │   ├── client/       ← Lua that runs on the client
                    │   └── server/       ← Lua that runs on the server
                    ├── textures/         ← Icons: item_YourIcon.png
                    └── models/           ← 3D models (.fbx, .b3d, .glb)
```

For local testing (non-Workshop), your mod goes in `~/Zomboid/mods/MyMod/` with the same internal structure.

### mod.info

```
name=My Mod Display Name
id=MyModID
author=YourName
description=What your mod does, shown in the mod list.
poster=poster.png
```

- `name` and `id` are required. Everything else is optional but recommended.
- `id` is used to reference this mod in dependencies: `require=SomeOtherModID`
- As of 42.15, mod name and description can be translated — see the Translation section.

### workshop.txt

```
version=1
workshopid=0
title=My Mod Display Name
description=Workshop description (can be longer than mod.info).
visibility=public
tags=Build 42;Items;Recipes
```

Set `workshopid=0` until you've published to Workshop — it gets assigned on first publish.

### Module Naming

When you define items and recipes, you declare a `module`. Items are then referenced as `ModuleName.ItemID` everywhere in the game (recipes, Lua, loot tables, etc.).

Two approaches:

**Use `module Base`** — your items join the vanilla Base namespace. Simpler references (`Base.MyItem`), but higher risk of ID collision with other mods. Fine for small mods.

**Use a custom module** — `module MyMod { }` means items are `MyMod.MyItem`. Safer, recommended for anything you plan to publish. Add `imports { Base }` inside your module block to reference vanilla items in recipes.

```
module MyMod
{
    imports
    {
        Base
    }

    item MyItem { ... }

    craftRecipe MakeMyItem
    {
        inputs { item 1 [Base.Plank], }   // vanilla item via import
        outputs { item 1 MyMod.MyItem, }  // our item via module name
    }
}
```

---

## Item Scripting

Scripts go in `42/media/scripts/` as `.txt` files. You can have multiple files — the game loads all of them. Splitting items and recipes into separate files (`items.txt`, `recipes.txt`) is a common convention.

### Basic Item Template

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

### Container Item Properties

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

### Two-Handed Hauling Item Pattern

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

### RunSpeedModifier Reference (this mod)

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

## Recipe Scripting (craftRecipe)

### Full Syntax Reference

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

### Parameters

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

### inputs Block

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

### outputs Block

```
outputs
{
    item 1 MyMod.MyItem,        // one item
    item 3 Base.Plank,          // multiple of same item
}
```

Note: outputs use `Module.ItemID` **without brackets**, unlike inputs which use `[Module.ItemID]` with brackets.

### Upgrade Path Pattern

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

### Skill Names

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

## Container Mechanics

### How Capacity Works

`Capacity` is the maximum total encumbrance of items stored inside the container. Each item in the game has a `Weight` value — when placed in a container with `WeightReduction = 60`, that item effectively weighs 40% of its normal value toward the container's capacity.

Example: A container with `Capacity = 50` and `WeightReduction = 65`:
- Item weighing 5.0 effectively weighs 1.75 (5.0 × 0.35)
- So you can fit ~28 of that item before hitting the capacity limit

The **100 capacity hard cap** in vanilla B42 is a known limitation. If you need more, the community workaround is to use a Lua `OnContainerUpdate` hook to manipulate the container — but this is poorly documented and needs testing.

### Lua Container API

```lua
local item = player:getPrimaryHandItem()
if item then
    local container = item:getContainer()
    if container then
        local capacity  = container:getCapacity()     -- max capacity
        local weight    = container:getWeight()       -- current contents weight
        local fillRatio = weight / capacity           -- 0.0 to 1.0

        -- Iterate contents
        local items = container:getItems()            -- Java ArrayList
        for i = 0, items:size() - 1 do
            local storedItem = items:get(i)
            -- do something with storedItem
        end

        -- Add/remove items
        container:AddItem("Base.SomeItem")            -- add by string ID
        container:Remove(someInventoryItem)           -- remove by InventoryItem reference
        container:contains(someInventoryItem)         -- boolean check
    end
end
```

---

## Visual Assets — Icons and Models

PZ items have two distinct visual components: a 2D **icon** shown in the inventory UI, and an optional **3D model** shown in the player's hands and on the ground. Neither is required for an item to function — items with no model simply show nothing in-hand, and items with no icon show a blank square.

### Icons

**Specs:**
- Format: PNG, **8-bit color only** (16-bit is rejected silently)
- Size: **32×32 pixels** (standard); 64×64 works but 32×32 is the norm
- Location: `42/media/textures/Item_YourName.png`
- Naming: filename **must** start with `Item_` (capital I)
- Script reference: `Icon = YourName,` — omit the `Item_` prefix and extension

```
// File: 42/media/textures/Item_Travois.png
// Script:
item StickTravois
{
    Icon = Travois,    // engine finds Item_Travois.png automatically
    ...
}
```

**Common mistake:** Putting `Item_` in the script value (`Icon = Item_Travois`) or forgetting it on the filename. Either breaks icon display with no error.

**Using vanilla icons as placeholders:** You can reference any vanilla icon name directly — the engine searches vanilla texture packs first. No file needed in your mod. To browse available vanilla icons, use [ProjectZomboidPackManager](https://github.com/cdaragorn/ProjectZomboidPackManager) to extract `UI2.pack` from the game install.

For food items, the engine also looks for suffixed variants: `Item_MyFood_Cooked.png`, `Item_MyFood_Rotten.png`, `Item_MyFood_Burnt.png`.

---

### 3D Models

Each item can have three model references:

| Property | Where it shows | Script location |
|---|---|---|
| `ReplaceInPrimaryHand` | Right hand when equipped | Item script block |
| `ReplaceInSecondHand` | Left hand when equipped | Item script block |
| `WorldStaticModel` | Dropped on the ground | Item script block |
| `StaticModel` | Default held appearance (single-hand items) | Item script block |

All of these reference a **model script name** — a name you define in a `model {}` script block, which points to the actual `.fbx` file. They do not reference the file directly.

#### Model Script Block

Defined in any `.txt` file in `42/media/scripts/`:

```
model MyTravois
{
    mesh    = MyTravois,        // references MyTravois.fbx in 42/media/models_X/
    texture = MyTravoisTex,     // references MyTravoisTex.png in 42/media/textures/
    attachment Bip01_Prop1
    {
        offset = 0.0 0.0 0.0,
        rotate = 0.0 0.0 0.0,
        scale  = 1.0,
    }
}
```

- `Bip01_Prop1` is the right-hand prop bone; `Bip01_Prop2` is the left-hand bone
- `offset`, `rotate`, `scale` position the model relative to the bone — use the in-game **Attachment Editor** to adjust these visually without re-exporting

Then reference it in your item script:

```
item StickTravois
{
    ...
    ReplaceInPrimaryHand    = MyTravois holdingbagright,
    ReplaceInSecondHand     = MyTravois holdingbagleft,
    WorldStaticModel        = MyTravois,
}
```

The second word after the model name (`holdingbagright`, `holdingbagleft`) is the **animation name** — it controls the character's arm and hand pose while holding the item. Common values:
- `holdingbagright` / `holdingbagleft` — bag/pack holding pose (arms at sides)
- Browse `[PZ install]/media/scripts/items_weapons.txt` for more animation names used by vanilla items

**Referencing vanilla models:** You can use vanilla model script names directly in any of these properties. To find them, search vanilla script files for `StaticModel =`. This is the fastest way to get something visible in-game while custom art is pending.

#### Model File Requirements

- **Format:** `.fbx` — community standard, works reliably. Stick with FBX for item meshes; `.glb` support in B42 is still being established for item models specifically.
- **Location:** `42/media/models_X/YourModel.fbx`
- **Scale:** Export at **0.01 scale** in Blender. Zomboid's unit scale means a 1m Blender object exports at 100x too large — the 0.01 corrects this.
- **Transforms:** Apply all transforms before export (Object > Apply > All Transforms)
- **Poly count:** No hard limit; match vanilla items (low-poly, a few hundred to low thousands of triangles)
- **Texture:** A separate PNG (typically 128×128) placed in `42/media/textures/`

#### Blender Workflow (Custom Models)

1. Model your item in Blender (any modern version)
2. Apply all transforms
3. Export as FBX: File > Export > FBX
   - Scale: `0.01`
   - Forward: `-Z Forward`, Up: `Y Up` (standard)
4. Place `.fbx` in `42/media/models_X/`
5. Place texture PNG in `42/media/textures/`
6. Write a `model {}` script block referencing both
7. Launch game with your mod, open **Attachment Editor** to tweak grip position

**Useful tools:**
- [ZomboidAssetConverter](https://pzwiki.net/wiki/ZomboidAssetConverter) — converts vanilla `.x` model files to `.gltf` for import into Blender as reference
- [ZomboidImportExport Blender addon](https://github.com/rundaaa/ZomboidImportExport) — old (Blender 2.79) but useful for importing vanilla assets
- [Paddlefruit Community Rig](https://github.com/Paddlefruit/ProjectZomboid_CommunityRig) — character animation rig; useful for understanding prop bone positions
- **Attachment Editor** — built into the game's debug/modding tools; adjust model positioning live

#### Workshop and Mod Poster Images

| Image | Size | Location | Purpose |
|---|---|---|---|
| `preview.png` | 256×256, under 1MB | Workshop root (next to `workshop.txt`) | Steam Workshop thumbnail |
| `poster.png` | 256×256, under 1MB | `42/poster.png` | Shown in the in-game mod list |

---

## Lua Scripting

### File Location

Client Lua: `42/media/lua/client/`
Server Lua: `42/media/lua/server/`
Shared Lua: `42/media/lua/shared/` (loaded by both)

The game loads all `.lua` files in these directories automatically — no registration needed.

### Event System

PZ uses a global `Events` object. You register a Lua function to be called when an event fires:

```lua
local function myHandler(player)
    -- your logic
end

Events.OnPlayerUpdate.Add(myHandler)

-- Remove when no longer needed
Events.OnPlayerUpdate.Remove(myHandler)
```

### Key Events for Item/Container Mods

| Event | When it fires | Parameters |
|---|---|---|
| `OnPlayerUpdate` | Every game tick per player | `(IsoPlayer player)` |
| `OnPlayerMove` | Every tick the local player moves | `(IsoPlayer player)` |
| `OnCreatePlayer` | Player spawns into world | `(int playerIndex, IsoPlayer player)` |
| `OnFillInventoryObjectContextMenu` | Player right-clicks an item | `(int playerNum, ISContextMenu menu, ArrayList items)` |
| `OnContainerUpdate` | Container contents change | varies |
| `EveryOneMinute` | Every in-game minute | none |

### OnPlayerUpdate Performance

`OnPlayerUpdate` fires **every game tick** for every player. Keep your handlers fast:

```lua
local MY_ITEM = "MyMod.MyItem"

local function onPlayerUpdate(player)
    -- Always guard against nil first
    if not player then return end

    -- Exit early if player isn't holding our item
    local primary = player:getPrimaryHandItem()
    if not primary or primary:getFullType() ~= MY_ITEM then return end

    -- Only now do more expensive operations
    local container = primary:getContainer()
    -- ...
end

Events.OnPlayerUpdate.Add(onPlayerUpdate)
```

Avoid creating tables or objects inside `OnPlayerUpdate`. Cache anything reusable at module level.

### Drop-on-Aim Pattern

For two-handed hauling items (containers with `RequiresEquippedBothHands = true`), you might want the item to drop automatically if the player tries to aim a weapon. In practice, **`RequiresEquippedBothHands` already prevents this** — with both hands occupied by a container, the player has no weapon equipped, so `player:isAiming()` will never return true. The pattern is effectively redundant for `RequiresEquippedBothHands` container items.

However, if you are using `TwoHandWeapon = TRUE` on a weapon-type item, or if you need a failsafe, here is the pattern. It is safe to include even when redundant — it simply never triggers:

```lua
local HAUL_ITEMS = {
    ["MyMod.DragSledge"]        = true,
    ["MyMod.WoodenWheelbarrow"] = true,
    -- add all haul items here
}

local aimUnequipSent = false  -- throttle: only send once per aim event

local function onPlayerUpdate(player)
    if not player then return end
    local primary = player:getPrimaryHandItem()
    if not primary or not HAUL_ITEMS[primary:getFullType()] then
        aimUnequipSent = false
        return
    end

    if player:isAiming() and not aimUnequipSent then
        aimUnequipSent = true
        -- Request server to unequip (never modify state on client)
        sendClientCommand(player, "MyMod", "dropHaulItem", {})
    elseif not player:isAiming() then
        aimUnequipSent = false
    end
end

Events.OnPlayerUpdate.Add(onPlayerUpdate)
```

**Unequip methods (confirmed in B42):** `player:setPrimaryHandItem(nil)` and `player:setSecondaryHandItem(nil)` are the correct server-side calls to clear hand slots. Verified against `ISUnequipAction.lua` and `STrapGlobalObject.lua` in vanilla source.
```

### IsoPlayer Common Methods

```lua
player:getPrimaryHandItem()         -- InventoryItem or nil
player:getSecondaryHandItem()       -- InventoryItem or nil
player:setPrimaryHandItem(item)     -- equip item (nil to unequip)
player:setSecondaryHandItem(item)   -- equip item (nil to unequip)
player:getInventory()               -- ItemContainer (main inventory)
player:isAiming()                   -- bool
player:getVehicle()                 -- IsoVehicle or nil (nil if on foot)
player:getX(), player:getY(), player:getZ()  -- world coordinates
```

### InventoryItem Common Methods

```lua
item:getFullType()      -- "Module.ItemID" e.g. "Base.Plank"
item:getType()          -- "ItemID" without module e.g. "Plank"
item:getContainer()     -- ItemContainer if this is a container item, else nil
item:getWeight()        -- item's weight value
item:getCondition()     -- durability (0-100)
item:getDisplayName()   -- localized display name string
```

---

## Multiplayer Architecture

> **Design principle:** Build for multiplayer from the start. Retrofitting MP support into a mod that was designed singleplayer-only is painful. Designing for MP upfront costs very little.

### What PZ Handles Automatically

The good news: most item and container behavior is already MP-safe without any extra work from you.

| Mechanic | MP-safe by default? | Notes |
|---|---|---|
| Container contents (items in/out) | ✓ Yes | Engine syncs inventory state to all clients |
| `RunSpeedModifier` on items | ✓ Yes | Script property applied by engine |
| `RequiresEquippedBothHands = true` | ✓ Yes | Engine enforces both hand slots for non-weapon items |
| `TwoHandWeapon = TRUE` | ✓ Yes | Engine enforces both hand slots, but for weapons only |
| Item pickup / drop | ✓ Yes | Engine handles |
| Crafting recipes | ✓ Yes | Engine handles |

The concern is **custom Lua that modifies game state**. Any state change that happens only on the client will desync in multiplayer — other clients and the server won't know about it.

### Execution Contexts

PZ Lua runs in three contexts:

| Context | Lua folder | When it runs | What it can do |
|---|---|---|---|
| **Client** | `lua/client/` | On each player's machine | Detect local player state, render UI, send commands to server |
| **Server** | `lua/server/` | On the dedicated server or listen-server host | Authoritative state changes, receive client commands |
| **Shared** | `lua/shared/` | On both client and server | Constants, utility functions, data tables used by both |

On a **listen server** (one player hosts, others join), the host machine runs both client and server Lua. This means client Lua files will run twice for the host — once as client, once as server. This is expected and normal, but be aware of it when reasoning about execution counts.

### Guard Functions

```lua
isClient()       -- true on a game client (including listen-server host's client)
isServer()       -- true on the server (dedicated or listen-server host)
isMultiplayer()  -- true when connected to or hosting a multiplayer session
```

Use these when behavior genuinely differs between contexts:

```lua
local function onPlayerUpdate(player)
    if not isClient() then return end  -- only run on client
    -- ...
end
```

However, **prefer file placement over guards** — putting code in `client/` vs `server/` is cleaner and more intentional than sprinkling `isClient()` checks throughout shared code.

### The Client → Server Command Pattern

Any time a client needs to trigger a state change, use `sendClientCommand` rather than modifying state directly. This is the core MP safety pattern.

```
Client detects event
    → sendClientCommand(player, module, command, args)
        → Server receives via OnClientCommand
            → Server validates request
                → Server modifies state
                    → PZ engine syncs to all clients
```

**Client side** (`lua/client/`):
```lua
-- Detect, then request
if player:isAiming() and holdingHaulItem then
    sendClientCommand(player, "TheLongHaul", "dropHaulItem", {})
end
```

**Server side** (`lua/server/`):
```lua
local function onClientCommand(module, command, player, args)
    if module ~= "TheLongHaul" then return end

    if command == "dropHaulItem" then
        -- ALWAYS re-validate on the server — never trust the client alone
        if isHaulItem(player:getPrimaryHandItem()) then
            player:setPrimaryHandItem(nil)
            player:setSecondaryHandItem(nil)
        end
    end
end

Events.OnClientCommand.Add(onClientCommand)
```

**Critical:** Always re-validate the precondition on the server. A malicious client could send any command with any arguments — the server must check that the action is actually valid before acting.

### Server → Client Commands

When the server needs to notify clients of something (e.g., a visual effect, a sound, a UI update):

```lua
-- Server: broadcast to all clients
sendServerCommand("TheLongHaul", "someEvent", { x = 10, y = 20 })

-- Server: send to one specific player
sendServerCommand(player, "TheLongHaul", "someEvent", { x = 10, y = 20 })

-- Client: receive
local function onServerCommand(module, command, args)
    if module ~= "TheLongHaul" then return end
    if command == "someEvent" then
        -- do client-side visual/audio/UI response
    end
end

Events.OnServerCommand.Add(onServerCommand)
```

### What Needs Custom MP Handling (vs What Doesn't)

For a **container item mod** like The Long Haul, most things are free:

**Free (engine handles):**
- Items going into and out of the container
- The speed penalty (`RunSpeedModifier`)
- The two-hand equip restriction (`RequiresEquippedBothHands`)
- Item weight and encumbrance calculations

**Needs custom handling (Lua state changes):**
- Any forced unequip triggered by game logic (e.g., drop-on-aim)
- Any custom condition checks that result in state modifications
- Anything that reads server-side state and makes decisions based on it

### Item Duplication Concern

The classic MP bug with container mods: if a player disconnects while holding a container item, the item can appear in two places — still in the "ghost" player's hands and also in their inventory. PZ's engine handles most of this automatically for standard inventory items, but it's worth testing by:

1. Having a player pick up a haul item with items inside
2. Force-disconnecting (kill the process, not a clean exit)
3. Checking if the haul item or its contents are duplicated on rejoin

If duplication occurs, the fix is server-side tracking of who holds which haul item and cleanup on `OnPlayerDisconnect`.

### The `OnPlayerUpdate` Event in MP

`OnPlayerUpdate` behaves differently depending on context:

- **On the client:** fires only for the **local player**
- **On the server:** fires for **every connected player**

This is important: if you put an `OnPlayerUpdate` handler in `lua/server/`, it will run for all players simultaneously. In `lua/client/`, it only runs for the local player. For The Long Haul, detection logic belongs in `client/` (local player only), state changes belong in `server/` (authoritative, for whoever triggered it).

---

## Key Gotchas and Verified IDs

All entries below were verified directly against B42 game script files in `media/scripts/generated/` and `media/lua/`. Source files noted where relevant.

### Verified Vanilla Item IDs

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

### Verified Skill Names

| Display name | `SkillRequired` / `xpAward` ID |
|---|---|
| Carpentry | `Woodwork` |
| Metalworking | `Blacksmith` (**not** `Metalwork`) |
| Masonry | `Masonry` |
| Carving | `Carving` |
| Tailoring | `Tailoring` |

Source: `recipes_blacksmith_armor.txt`, `recipes_stonemasonry_i.txt`, `recipes_carving.txt`, `recipes_carpentry.txt`, `recipes_tailoring.txt`.

### Verified Recipe Tags (Tools)

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

### Verified Script Properties

**`RequiresEquippedBothHands` vs `TwoHandWeapon`** — the most important distinction for container mods:

| Property | Works on | Effect |
|---|---|---|
| `RequiresEquippedBothHands = true` | Any item type (containers, normal items) | Occupies both hand slots; player cannot hold anything else |
| `TwoHandWeapon = TRUE` | Weapons only (`ItemType = base:weapon`) | Same effect, but only applies to weapon-type items |

Using `TwoHandWeapon` on a container will not work as expected — use `RequiresEquippedBothHands` for any non-weapon item.

Both are checked together in the unequip Lua: `item:isTwoHandWeapon() or item:isRequiresEquippedBothHands()` (from `ISUnequipAction.lua`).

**`setPrimaryHandItem(nil)` and `setSecondaryHandItem(nil)`** — confirmed correct server-side unequip methods in B42. Used in vanilla: `ISUnequipAction.lua` lines 119–127, `STrapGlobalObject.lua` lines 256–257.

**`player:isAiming()`** — confirmed B42 method. Returns true when the player is in aim mode (right-click held with a firearm). Will never return true when a `RequiresEquippedBothHands` container is equipped, since there's no weapon in hand to aim.

### Common Mistakes

**Item not appearing in game:** Check that your module declaration is correct and the file is in `42/media/scripts/`. Also check the game's console log — script parse errors are printed there.

**Recipe not appearing:** The `category` value must match a valid crafting category. Common values: `Carpentry`, `Tailoring`, `Blacksmith`, `Masonry`, `Carving`, `Cooking`, `Farming`. Wrong category = recipe still exists but may be hard to find or not show at all.

**Metalwork recipes silently not gating:** You almost certainly wrote `Metalwork` instead of `Blacksmith`. The in-game skill is called "Metalworking" but the internal ID is `Blacksmith`.

**Items being destroyed when they should be kept:** Forgot `mode:keep` on a tool input. All inputs are consumed by default.

**Container not blocking both hands:** Used `TwoHandWeapon = TRUE` on a container. Switch to `RequiresEquippedBothHands = true`.

**Twine always running out mid-recipe:** `Base.Twine` is a drainable item — it has a `UseDelta` that depletes it. If your recipe asks for `item 3 [Base.Twine]`, it expects 3 separate Twine items, not 3 uses from one. Consider using `tags[base:twine]` if you want any twine-tagged item instead.

**Mod not loading at all:** Check `mod.info` is inside `42/` (not at the root). Check `id` field has no spaces.

**Old B41 mod not working in B42:** The three big breaks — folder structure, `Type =` vs `ItemType =`, and `recipe {}` vs `craftRecipe {}` — cover 95% of cases.

---

## Testing Workflow

### Setup

1. Clone/copy your mod into `~/Zomboid/mods/YourModID/` (the folder name should match the `id` in `mod.info`)
2. Launch PZ, go to **Mods**, enable your mod
3. Start a new save in **Sandbox** mode — use a custom preset with high XP gain and all skills maxed so you can test any recipe immediately

### Console / Debug Output

Enable the debug console: launch PZ with `-debug` flag, or press `F11` in-game to open the debug panel.

Lua errors print to:
- The in-game console (open with `` ` `` key by default)
- `~/Zomboid/Logs/` — check `*.txt` files here for script parse errors on startup

### Quick Iteration

You **do not** need to restart the game to reload Lua changes. In the debug console:

```
/reloadlua
```

However, **item script and recipe changes do require a full restart** — those are parsed at game load, not at runtime.

### Checking Item IDs In-Game

With debug mode enabled:
- Open your inventory and hover over any item
- The debug tooltip shows the full item ID (`Module.ItemID`)
- Use this to verify any `TODO:VERIFY` IDs in your scripts

---

## Resources

### Official / Semi-Official

- [PZwiki — Mod Structure](https://pzwiki.net/wiki/Mod_structure)
- [PZwiki — Item Scripts](https://pzwiki.net/wiki/Item_(scripts))
- [PZwiki — craftRecipe Scripts](https://pzwiki.net/wiki/CraftRecipe_(scripts))
- [PZwiki — inputs block](https://pzwiki.net/wiki/Inputs)
- [PZwiki — Lua Events](https://pzwiki.net/wiki/Lua_Events)
- [PZwiki — Getting Started with Modding](https://pzwiki.net/wiki/Getting_started_with_modding)
- [Official JavaDocs](https://projectzomboid.com/modding/) — Java class reference, the closest thing to an official API doc
- [Unofficial JavaDocs (B42)](https://pzwiki.net/wiki/Unofficial_JavaDocs_(Build_42)) — community-maintained, more complete

### Community Tools

- [PZ Lua Event Docs](https://demiurgequantified.github.io/ProjectZomboidLuaDocs/md_Events.html) — best reference for event signatures
- [PZ Event Stubs (GitHub)](https://github.com/demiurgeQuantified/PZEventStubs) — IDE intellisense stubs for PZ events
- [B42 Mod Template (GitHub)](https://github.com/LabX1/ProjectZomboid-Build42-ModTemplate) — working B42 starter template
- [Awesome B42 Resources (GitHub)](https://github.com/JBD-Mods/awesome-project-zomboid-build42-resources) — curated tool list

### Reference Mods (Study These)

- [Hydrocraft Wheelbarrow (Workshop)](https://steamcommunity.com/sharedfiles/filedetails/?id=2926995676) — simple B42 container item, good starting reference
- [SaucedCarts (Workshop)](https://steamcommunity.com/sharedfiles/filedetails/?id=3651954650) — advanced ground-object push system, B42-native

### Migration Guides

- [B42 Modding Migration Guide (42.15) — The Indie Stone Forums](https://theindiestone.com/forums/index.php?/topic/92433-modding-migration-guide-4215/) — covers the translation format change
- [How to Update Your Mod for Build 42 (Steam Guide)](https://steamcommunity.com/sharedfiles/filedetails/?id=3391657438)

---

*This guide is maintained alongside [The Long Haul mod](https://github.com/gotmayonase/the-long-haul). Last updated: April 2026. All item IDs, skill names, tags, and API calls in the Key Gotchas section verified directly against B42.15 game script files.*
