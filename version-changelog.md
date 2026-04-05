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
