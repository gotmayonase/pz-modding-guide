# Visual Assets — Icons and Models

[← Container Mechanics](container-mechanics.md) | [Home](index.md) | Next: [Lua Scripting →](lua-scripting.md)

---

PZ items have two distinct visual components: a 2D **icon** shown in the inventory UI, and an optional **3D model** shown in the player's hands and on the ground. Neither is required for an item to function — items with no model simply show nothing in-hand, and items with no icon show a blank square.

---

## Icons

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

## 3D Models

Each item can have three model references:

| Property | Where it shows | Script location |
|---|---|---|
| `ReplaceInPrimaryHand` | Right hand when equipped | Item script block |
| `ReplaceInSecondHand` | Left hand when equipped | Item script block |
| `WorldStaticModel` | Dropped on the ground | Item script block |
| `StaticModel` | Default held appearance (single-hand items) | Item script block |

All of these reference a **model script name** — a name you define in a `model {}` script block, which points to the actual `.fbx` file. They do not reference the file directly.

### Model Script Block

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

---

### Model File Requirements

- **Format:** `.fbx` — community standard, works reliably. Stick with FBX for item meshes; `.glb` support in B42 is still being established for item models specifically.
- **Location:** `42/media/models_X/YourModel.fbx`
- **Scale:** Export at **0.01 scale** in Blender. Zomboid's unit scale means a 1m Blender object exports at 100x too large — the 0.01 corrects this.
- **Transforms:** Apply all transforms before export (Object > Apply > All Transforms)
- **Poly count:** No hard limit; match vanilla items (low-poly, a few hundred to low thousands of triangles)
- **Texture:** A separate PNG (typically 128×128) placed in `42/media/textures/`

---

### Blender Workflow (Custom Models)

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

---

## Workshop and Mod Poster Images

| Image | Size | Location | Purpose |
|---|---|---|---|
| `preview.png` | 256×256, under 1MB | Workshop root (next to `workshop.txt`) | Steam Workshop thumbnail |
| `poster.png` | 256×256, under 1MB | `42/poster.png` | Shown in the in-game mod list |

---

[← Container Mechanics](container-mechanics.md) | [Home](index.md) | Next: [Lua Scripting →](lua-scripting.md)
