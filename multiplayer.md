# Multiplayer Architecture

[← Lua Scripting](lua-scripting.md) | [Home](index.md) | Next: [Key Gotchas and Verified IDs →](gotchas.md)

---

> **Design principle:** Build for multiplayer from the start. Retrofitting MP support into a mod that was designed singleplayer-only is painful. Designing for MP upfront costs very little.

---

## What PZ Handles Automatically

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

---

## Execution Contexts

PZ Lua runs in three contexts:

| Context | Lua folder | When it runs | What it can do |
|---|---|---|---|
| **Client** | `lua/client/` | On each player's machine | Detect local player state, render UI, send commands to server |
| **Server** | `lua/server/` | On the dedicated server or listen-server host | Authoritative state changes, receive client commands |
| **Shared** | `lua/shared/` | On both client and server | Constants, utility functions, data tables used by both |

On a **listen server** (one player hosts, others join), the host machine runs both client and server Lua. This means client Lua files will run twice for the host — once as client, once as server. This is expected and normal, but be aware of it when reasoning about execution counts.

---

## Guard Functions

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

---

## The Client → Server Command Pattern

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

---

## Server → Client Commands

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

---

## What Needs Custom MP Handling (vs What Doesn't)

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

---

## Item Duplication Concern

The classic MP bug with container mods: if a player disconnects while holding a container item, the item can appear in two places — still in the "ghost" player's hands and also in their inventory. PZ's engine handles most of this automatically for standard inventory items, but it's worth testing by:

1. Having a player pick up a haul item with items inside
2. Force-disconnecting (kill the process, not a clean exit)
3. Checking if the haul item or its contents are duplicated on rejoin

If duplication occurs, the fix is server-side tracking of who holds which haul item and cleanup on `OnPlayerDisconnect`.

---

## The `OnPlayerUpdate` Event in MP

`OnPlayerUpdate` behaves differently depending on context:

- **On the client:** fires only for the **local player**
- **On the server:** fires for **every connected player**

This is important: if you put an `OnPlayerUpdate` handler in `lua/server/`, it will run for all players simultaneously. In `lua/client/`, it only runs for the local player. For The Long Haul, detection logic belongs in `client/` (local player only), state changes belong in `server/` (authoritative, for whoever triggered it).

---

[← Lua Scripting](lua-scripting.md) | [Home](index.md) | Next: [Key Gotchas and Verified IDs →](gotchas.md)
