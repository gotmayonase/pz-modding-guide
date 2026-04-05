# Testing Workflow

[← Key Gotchas](gotchas.md) | [Home](index.md) | Next: [Resources →](resources.md)

---

## Setup

1. Clone/copy your mod into `~/Zomboid/mods/YourModID/` (the folder name should match the `id` in `mod.info`)
2. Launch PZ, go to **Mods**, enable your mod
3. Start a new save in **Sandbox** mode — use a custom preset with high XP gain and all skills maxed so you can test any recipe immediately

---

## Console / Debug Output

Enable the debug console: launch PZ with `-debug` flag, or press `F11` in-game to open the debug panel.

Lua errors print to:
- The in-game console (open with `` ` `` key by default)
- `~/Zomboid/Logs/` — check `*.txt` files here for script parse errors on startup

---

## Quick Iteration

You **do not** need to restart the game to reload Lua changes. In the debug console:

```
/reloadlua
```

However, **item script and recipe changes do require a full restart** — those are parsed at game load, not at runtime.

---

## Checking Item IDs In-Game

With debug mode enabled:
- Open your inventory and hover over any item
- The debug tooltip shows the full item ID (`Module.ItemID`)
- Use this to verify any unknown IDs in your scripts

---

[← Key Gotchas](gotchas.md) | [Home](index.md) | Next: [Resources →](resources.md)
