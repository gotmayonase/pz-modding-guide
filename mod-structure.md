# Mod Folder Structure

[← B42 Changes](b42-changes.md) | [Home](index.md) | Next: [Item Scripting →](item-scripting.md)

---

## Minimal Structure (B42 only)

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

---

## mod.info

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

---

## workshop.txt

```
version=1
workshopid=0
title=My Mod Display Name
description=Workshop description (can be longer than mod.info).
visibility=public
tags=Build 42;Items;Recipes
```

Set `workshopid=0` until you've published to Workshop — it gets assigned on first publish.

---

## Module Naming

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

[← B42 Changes](b42-changes.md) | [Home](index.md) | Next: [Item Scripting →](item-scripting.md)
