# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Matt's Marketplace Hustle is a single-file HTML5 browser game — a text-based Facebook Marketplace flipping simulator. The entire application (HTML, CSS, JavaScript, game logic) lives in `index.html`. There is no build system, no package manager, and no dependencies beyond Google Fonts and a YouTube embed.

**To run:** Open `index.html` directly in any modern browser.

## Architecture

### Single Global State Object

All game state lives in the `G` object:
- `G.money`, `G.energy`, `G.budLights`, `G.rallyPrep` — core player resources
- `G.inventory` — array of purchased items
- `G.listings` — current marketplace listings (regenerated each browse)
- `G.pendingEvent` — blocks normal commands when a random event requires response
- `G.phase` — tracks game phase (`'game'`, `'intro'`, `'end'`)

### Game Loop

`processCommand(input)` → parses command → updates `G` → calls `afterAction()` → checks random events → updates UI.

`afterAction()` runs after every command: drains energy, fires random events (weighted probabilities), checks win/loss conditions, and auto-advances the week after 10 actions.

### Random Event System

Events trigger inside `afterAction()` with weighted coin flips:
- Griffin barking: 9% — sets `G.pendingEvent = 'griffin'`, requires `calm` command
- T-Swan visit: 3% — sets `G.pendingEvent = 'tswan'`, requires `deal` command
- Fireworks question: 7% — sets `G.pendingEvent = 'fireworks'`, requires `answer [#]`
- Rally questions: 8% — sets `G.pendingEvent = 'rally'`, requires `answer [#]`

While `G.pendingEvent` is set, all other commands are blocked.

### Items & Pricing

Ten regular items are defined in the `ITEMS` array (each with `name`, `buyMin/Max`, `sellMin/Max`). The recalled Yeti Cooler is a special rare drop handled separately with an `upgrade` command that applies a 3× multiplier.

`generateListings()` picks 4 random items, assigns random prices in their buy ranges, and stores them in `G.listings`. Lowballed-and-rejected listings are tracked in `G.blocked` to prevent re-listing.

### Output & UI

`out(text, cssClass)` — appends a line to the terminal output div. CSS classes map to colors: `system`, `event`, `griffin`, `rally`, `money`, `error`, `win`, `special`.

`updateStatus()` — refreshes the top stats bar (week, cash, Bud Lights, energy, rally prep, inventory count).

### Win/Loss Conditions

Checked inside `afterAction()`:
- **Loss:** `G.energy <= 0`, Griffin bite risk at 100%, or end of final week with `G.rallyPrep < 75`
- **Win:** End of final week with `G.rallyPrep >= 75` — profit determines tier (Poor <$12K, Decent $12K+, Good $25K+, Legendary $40K+)

### Command Registration

Commands are defined as entries in the `COMMANDS` array (name + aliases) and handled by a large `switch` inside `processCommand()`. Tab-autocomplete matches against `COMMANDS` names and aliases.
