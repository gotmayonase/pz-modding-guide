# Build 42 vs Build 41 — What Changed

[← Home](index.md) | Next: [Mod Folder Structure →](mod-structure.md)

---

If you find a modding tutorial online, there's a good chance it's for Build 41. B42 introduced several **breaking changes** that will silently fail or crash your mod if you follow old guides.

## The Three Big Breaks

### 1. Mod Folder Structure

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

### 2. Item Type Declaration

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

### 3. Recipe System

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

---

## Minor Breaking Changes by Sub-Version

### 42.15
- **Translation files changed format.** Old: `.txt` files with language code in filename (`ItemName_EN.txt`). New: `.json` files without language code (`ItemName.json`), all UTF-8. Any mod with the old format will have broken translations in 42.15+.
- Fixed: mod folder detection — only one of `common/` or `42/` needs to exist.
- Added: `.glb` model file support in mod watcher.

### 42.14
- `Base.223Bullets`, `Base.BoxBullets223`, `Base.CartonBullets223` removed. Replaced with 5.56 equivalents. Only affects ammo mods.
- New thread-equivalent items added: `Base.DentalFloss`, `Base.FishingLine` now accepted by `tags[base:thread]`.

---

[← Home](index.md) | Next: [Mod Folder Structure →](mod-structure.md)
