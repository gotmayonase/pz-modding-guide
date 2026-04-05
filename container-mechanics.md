# Container Mechanics

[← Recipe Scripting](recipe-scripting.md) | [Home](index.md) | Next: [Visual Assets →](visual-assets.md)

---

## How Capacity Works

`Capacity` is the maximum total encumbrance of items stored inside the container. Each item in the game has a `Weight` value — when placed in a container with `WeightReduction = 60`, that item effectively weighs 40% of its normal value toward the container's capacity.

Example: A container with `Capacity = 50` and `WeightReduction = 65`:
- Item weighing 5.0 effectively weighs 1.75 (5.0 × 0.35)
- So you can fit ~28 of that item before hitting the capacity limit

The **100 capacity hard cap** in vanilla B42 is a known limitation. If you need more, the community workaround is to use a Lua `OnContainerUpdate` hook to manipulate the container — but this is poorly documented and needs testing.

---

## Lua Container API

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

[← Recipe Scripting](recipe-scripting.md) | [Home](index.md) | Next: [Visual Assets →](visual-assets.md)
