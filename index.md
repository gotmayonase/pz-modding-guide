# Project Zomboid Modding Guide (Build 42)

A practical, ground-up guide to modding Project Zomboid targeting **Build 42.15+** (unstable branch). Written as actual mod development happened — expect real examples, verified techniques, and honest notes about what still needs confirming in-game.

> **Why this guide exists:** Official documentation for B42 is sparse. The PZwiki lags behind. B42 made sweeping changes to the modding API that broke most B41 tutorials. This guide is an attempt to document the current reality.

---

## Table of Contents

1. [Build 42 vs Build 41 — What Changed](b42-changes.md) — breaking changes, migration checklist
2. [Mod Folder Structure](mod-structure.md) — folder layout, mod.info, modules
3. [Item Scripting](item-scripting.md) — item properties, container items, templates
4. [Recipe Scripting (craftRecipe)](recipe-scripting.md) — full syntax, inputs/outputs, skill names
5. [Container Mechanics](container-mechanics.md) — capacity, weight reduction, Lua container API
6. [Visual Assets — Icons and Models](visual-assets.md) — icons, 3D models, Blender workflow
7. [Lua Scripting](lua-scripting.md) — events, player API, common patterns
8. [Multiplayer Architecture](multiplayer.md) — client/server split, command pattern, MP safety
9. [Key Gotchas and Verified IDs](gotchas.md) — item IDs, skill names, tags, script properties
10. [Testing Workflow](testing.md) — setup, debug console, quick iteration
11. [Resources](resources.md) — official docs, community tools, reference mods
12. [Modding API Changes by Version](version-changelog.md) — per-patch changelog of modding-relevant changes

---

## Quick Start

If you're starting a new B42 mod from scratch:

1. Read [Build 42 vs Build 41](b42-changes.md) to understand what's different
2. Follow [Mod Folder Structure](mod-structure.md) to set up your mod directory
3. Use [Item Scripting](item-scripting.md) to define your items
4. Use [Recipe Scripting](recipe-scripting.md) to add crafting recipes
5. Check [Key Gotchas and Verified IDs](gotchas.md) before wondering why something doesn't work

---

## About This Guide

This guide targets **Build 42.15+** of Project Zomboid (unstable branch). All code examples and item IDs have been verified against the actual game files or tested in-game. Where something still needs in-game confirmation, it's marked.

B41 tutorials are everywhere. This guide documents what actually works in B42.
