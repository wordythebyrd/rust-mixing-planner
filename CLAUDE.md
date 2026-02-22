# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-file static web app (`index.html`) — a Rust game mixing table planner with live countdown timers, server restart warnings, and a recipe picker. No build step, no dependencies, no framework.

## Development

**Run locally:**
```bash
python -m http.server 3456
# then open http://localhost:3456
```

**Deploy:** Push to `main` — GitHub Pages redeploys automatically within ~30 seconds.
Live at: https://wordythebyrd.github.io/rust-mixing-planner/

## Architecture

Everything lives in `index.html` — styles, markup, and script are all inline. The JS is organized into sections separated by `// ═══` banner comments:

| Section | What it does |
|---|---|
| `DATA` | `ITEM_IMAGES` map (name → wiki CDN URL), `RECIPES` array (all 39 mixing table recipes with ingredients and base times) |
| `STATE` | Single `state` object: `skillLevel`, `restarts[]`, `tables[]`, plus modal locals `modalContext`, `selectedRecipeId`, `timeMode` |
| `SKILL` | `getSkillMultiplier()` / `getEffectiveSeconds()` — skill level 0–10, each level = 8.5% reduction |
| `RESTARTS` | Add/remove HH:MM restart times; `getNextRestartTime()` returns the next upcoming restart as `{ restart, time }` |
| `TABLES` | CRUD for tables and craft queue items; `confirmAddCraft()` chains queue items by setting `startTime`/`endTime` |
| `RENDERING` | `renderTables()` → `renderTableCard()` → `renderCraftItem()` — full re-render on state mutation; `renderAllQueues()` updates only queue sections |
| `TICK` | `setInterval(tick, 1000)` — updates all countdowns, progress bars, warning states, restart countdown, and status bar |
| `HELPERS` | `fmtDuration()`, `fmtSeconds()`, `fmtTime()`, `escHtml()` |

## Key Data Shapes

```js
// Recipe
{ id, output, outputQty, timeSeconds, inputs: [{ item, qty }] }

// Craft queue item
{ id, name, imgUrl, inputs, batches, baseSeconds, totalSeconds, startTime, endTime, recipeId }

// Restart
{ id: Date.now(), timeStr: "HH:MM" }

// Table
{ id, name, queue: [craft, ...] }
```

## Design Conventions

- CSS custom properties for the Rust color palette are all in `:root` — use those, don't hardcode colors.
- Item images come from the Facepunch wiki CDN (`files.facepunch.com`). New items need an entry in `ITEM_IMAGES`.
- `renderTables()` does a full `innerHTML` replace of `#tablesGrid` — do not cache references to table/craft DOM nodes across renders.
- Ingredient quantities from the wiki sometimes list the same item in multiple slots; `consolidateInputs()` merges duplicates before storing.
- `tick()` reads live `Date.now()` against stored `endTime` — do not mutate `endTime` after a craft starts unless re-chaining the whole queue.
