# Modding API Changes by Version

[Home](index.md)

---

A running log of Project Zomboid updates that affect modders. Only changes relevant to the modding API, script format, Lua, file structure, or item/recipe behavior are recorded here. Gameplay-only changes are omitted.

Entries are in reverse chronological order — newest at the top.

**Sources used:**
- The Indie Stone official patch notes (forums + Steam)
- Direct inspection of game script files between versions
- Community-reported mod breakages

---

## Build 42.16 / 42.16.1

**Release date:** March 31, 2026 (42.16.0 unstable); 42.16.1 hotfix ~April 2, 2026

> **Save incompatibility:** 42.16 saves are not compatible with 42.15. The primary cause is Echo Creek's removal from vanilla map data. Players on 42.15 saves must use the `outdatedunstable` branch.

### Breaking Changes

- **Echo Creek removed as a vanilla spawn location.** Mods that add custom spawn points or reference the spawn location list by position may be affected. Community mods to restore Echo Creek as a separate mod already exist on the Workshop.
- **Occupation and trait display names changed** — internal script IDs unconfirmed. The following were renamed; if the internal IDs also changed, any mod referencing them by ID will break. **Test explicitly.**

  | Old name | New name |
  |---|---|
  | Fire Officer | Firefighter |
  | Angler | Fishing Guide |
  | Wilderness Knowledge (trait) | Bushcrafter |
  | Outdoorsman / Nature Lover | Survivor |

- **Character creation screen reworked** (new horizontal layout, mode descriptions UI). Mods that hook into or modify the character creation screen will likely be broken.
- **`FirearmsUseDamageChance` sandbox option changed from boolean toggle to dropdown.** Mods that read or set this option by `true`/`false` need to handle the new discrete values.
- **Firearm attachment scripts changed.** Recoil Pad removed from Sawn-off Shotgun and M16. Scopes removed from JS-2000 attachment options. Mods overriding these attachment definitions should recheck.
- **42.16.0 mod loading regression.** A bug caused mods not to load at all in the initial 42.16.0 release. Fixed in 42.16.1. If testing on 42.16.0, upgrade to 42.16.1.

### New / Added

- **New vanilla traits:** `Tinkerer` (+1 Maintenance) and `Target Shooter` (+1 Aiming). Mods that enumerate all vanilla traits or add traits with these names will be affected.
- **New `roomDef` entries** for crafting Points of Interest. Custom POI or room definition scripts should check for conflicts.
- **VHS skill categories expanded** — new B42 skill categories added to the VHS system "for future support and modding utilization." Mods creating custom VHS skill tapes should check whether `MediaCategory` entries need updating.
- **`kill animal` tag added to vanilla Handiknife** item script. Mods that override the Handiknife script block should incorporate this tag.

### Bug Fixes (Modding-Relevant)

- **Drainable items in `craftRecipe` fixed.** If your mod had workarounds for the pre-fix broken behavior of drainable item consumption, those workarounds may now conflict with the corrected behavior. Test and remove any workarounds.

### Notes for Modders

- The tailoring XP system was reworked — XP is now granted for crafting items rather than grinding patches, with complexity-based rewards. Mods that modify tailoring `craftRecipe` entries or XP multipliers should re-test.
- Firearm attachments received new 3D models/textures. Mods using vanilla attachment models as a base should verify visual compatibility.
- The translation rework from 42.15 is still actively breaking mods in this build. If your mod uses the old `.txt` translation format, update to `.json` (see [Build 42 vs Build 41](b42-changes.md)).

**Sources:** [Official blog (42.16.0)](https://projectzomboid.com/blog/news/2026/03/balancing-time/) · [Steam 42.16.0](https://store.steampowered.com/news/app/108600/view/533253197651772556) · [Steam 42.16.1](https://store.steampowered.com/news/app/108600/view/533253650845270940) · [PZwiki Build 42.16.0](https://pzwiki.net/wiki/Build_42.16.0) · [patched.gg full changelog](https://patched.gg/games/project-zomboid/build-42160-unstable-released)

---

## Build 42.15

**Release date:** TBD (unstable branch)

### Breaking Changes

- **Translation file format changed.** Old format: `.txt` files with language code in the filename (e.g., `ItemName_EN.txt`). New format: `.json` files without language code (`ItemName.json`), UTF-8 encoding. Any mod using the old format will have broken translations in 42.15+.

### New / Added

- `.glb` model file support added to the mod watcher. FBX remains the safer choice for item models until `.glb` support is more established.
- Mod folder detection improved — only one of `common/` or `42/` needs to exist (previously both may have been required).
- Mod `name` and `description` in `mod.info` can now be translated.

### Removed / Deprecated

Nothing removed in this version.

---

## Build 42.14

**Release date:** TBD (unstable branch)

### Breaking Changes

- **Ammo item IDs removed:** `Base.223Bullets`, `Base.BoxBullets223`, `Base.CartonBullets223` no longer exist. Replaced with 5.56 equivalents. Only affects mods that reference these specific item IDs directly.

### New / Added

- `Base.DentalFloss` and `Base.FishingLine` now accepted by `tags[base:thread]`.

### Removed / Deprecated

- `Base.223Bullets` — use 5.56 equivalent
- `Base.BoxBullets223` — use 5.56 equivalent
- `Base.CartonBullets223` — use 5.56 equivalent

---

## Build 42.0 — Initial B42 Release

**Release date:** TBD

This version introduced the foundational B42 modding changes. For the full B41 → B42 migration guide, see [Build 42 vs Build 41 — What Changed](b42-changes.md).

### Breaking Changes (summary)

- **Versioned folder structure required.** Mods must use `42/media/` instead of `media/` at root.
- **`Type =` replaced by `ItemType =`** with new namespaced values (e.g., `base:container` instead of `Container`).
- **Recipe system replaced.** `recipe { }` blocks no longer work. All recipes must use `craftRecipe { }` syntax.

### New / Added

- `craftRecipe` system with `inputs` / `outputs` blocks, tag-based tool matching, `mode:keep`, and `flags[]`.
- `common/` folder support — loaded by both B41 and B42 simultaneously.
- Versioned subfolder system (`41/`, `42/`) for multi-build mod support.

---

*Have a correction or addition? Open an issue or PR on [GitHub](https://github.com/gotmayonase/pz-modding-guide).*
