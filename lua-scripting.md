# Lua Scripting

[← Visual Assets](visual-assets.md) | [Home](index.md) | Next: [Multiplayer Architecture →](multiplayer.md)

---

## File Location

```
Client Lua:  42/media/lua/client/
Server Lua:  42/media/lua/server/
Shared Lua:  42/media/lua/shared/   (loaded by both)
```

The game loads all `.lua` files in these directories automatically — no registration needed.

---

## Event System

PZ uses a global `Events` object. You register a Lua function to be called when an event fires:

```lua
local function myHandler(player)
    -- your logic
end

Events.OnPlayerUpdate.Add(myHandler)

-- Remove when no longer needed
Events.OnPlayerUpdate.Remove(myHandler)
```

---

## Key Events for Item/Container Mods

| Event | When it fires | Parameters |
|---|---|---|
| `OnPlayerUpdate` | Every game tick per player | `(IsoPlayer player)` |
| `OnPlayerMove` | Every tick the local player moves | `(IsoPlayer player)` |
| `OnCreatePlayer` | Player spawns into world | `(int playerIndex, IsoPlayer player)` |
| `OnFillInventoryObjectContextMenu` | Player right-clicks an item | `(int playerNum, ISContextMenu menu, ArrayList items)` |
| `OnContainerUpdate` | Container contents change | varies |
| `EveryOneMinute` | Every in-game minute | none |

---

## OnPlayerUpdate Performance

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

---

## Drop-on-Aim Pattern

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

---

## IsoPlayer Common Methods

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

---

## InventoryItem Common Methods

```lua
item:getFullType()      -- "Module.ItemID" e.g. "Base.Plank"
item:getType()          -- "ItemID" without module e.g. "Plank"
item:getContainer()     -- ItemContainer if this is a container item, else nil
item:getWeight()        -- item's weight value
item:getCondition()     -- durability (0-100)
item:getDisplayName()   -- localized display name string
```

---

[← Visual Assets](visual-assets.md) | [Home](index.md) | Next: [Multiplayer Architecture →](multiplayer.md)
