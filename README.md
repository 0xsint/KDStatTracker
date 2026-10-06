# KDStatTracker (BGStatsTracker)

A combat statistics add-on for **The Elder Scrolls Online** that tracks your Kill/Death ratio, kill streaks, multi-kills, duels, and Battleground match history in real time — with an animated, movable on-screen HUD and a full set of chat commands for digging into your history.

Works in **Battlegrounds**, **Cyrodiil / Imperial City (Cyrodiil-style PvP)**, and **1v1 Duels**.

---

## Table of Contents

- [Features](#features)
- [The HUD Panel](#the-hud-panel)
- [Kill Streaks](#kill-streaks)
- [Multi-Kills](#multi-kills)
- [Duel Tracking](#duel-tracking)
- [Battleground Match History](#battleground-match-history)
- [Slash Commands](#slash-commands)
- [Settings Menu](#settings-menu)
- [Saved Data](#saved-data)
- [Installation](#installation)
- [Requirements / Dependencies](#requirements--dependencies)
- [File Overview](#file-overview)
- [Known Behavior Notes](#known-behavior-notes)
- [Credits](#credits)

---

## Features

- 📊 **Live K/D HUD** — a draggable, scalable, themed overlay showing kills, deaths, K/D ratio, kill streaks, multi-kills, and your top kill/death abilities.
- 🔥 **Animated kill streak banners** with 16 named tiers, from *Double Kill* up to *Unreal* (100 kills).
- ⚡ **Animated multi-kill banners** (kills bunched within a 10-second window), from *Double Kill* up to *Deca Kill*.
- ⚔️ **Duel win/loss tracking**, including a per-opponent head-to-head record (by `@DisplayName`).
- 🏆 **Battleground match history** — automatically records kills/deaths and game mode for your last 20 matches.
- 🎯 **Ability-level kill/death source tracking** — see which abilities get you the most kills, and which ones kill you the most.
- 💾 **Per-character and account-wide stat tracking**, persisted across sessions via `SavedVariables`.
- ⌨️ **Twelve chat (`/`) commands** for querying stats without opening any menu.
- ⚙️ **LibAddonMenu-2.0 settings panel** for display toggles, scale, font size/color, panel position, and data reset.

---

## The HUD Panel

`KDStatTrackerUI.CreateUI()` builds a small themed overlay panel (default `320x198`, draggable and clamped to screen) displaying:

| Row | Label | Description |
|---|---|---|
| Header | `KDStatTracker` | Title bar with a pink accent stripe and a subtle 5-stripe translucent background theme |
| Row 1 | `KILLS` / `DEATHS` / `K/D` | Total kills, deaths, and computed ratio (`N/A` if 0 deaths and 0 kills) |
| Row 2 | `STREAKS` / `CURRENT` | Total number of recorded kill streaks (3+ kills without dying) and the size of your currently active streak |
| Row 3 | `MULTI` | Your current multi-kill chain count and tier name (e.g. `3 (Triple Kill)`) |
| Row 4 | `TOP KILL` | The ability that has landed you the most kills, with its count and last victim |
| Row 5 | `TOP DEATH` | The ability that has killed you the most, with its count |
| Row 6 | `LAST KILL` | The last player you killed |

The panel is:
- **Movable** — drag it anywhere; position is saved automatically on release (`OnMoveStop`).
- **Scalable** — 50%–200% via the settings slider.
- **Auto-refreshing** — a lightweight `OnUpdate` ticker (every 250ms) expires stale multi-kill chains and saves them to history even if you don't land another kill.
- **Toggleable** — can be hidden entirely via the settings panel without unloading the add-on.

## Kill Streaks

A kill streak increases every time you land a kill (`EVENT_COMBAT_EVENT`) and resets whenever you die (`EVENT_UNIT_DEATH_STATE_CHANGED`). Streaks of **3 or more kills** are saved to history (keeping the last 50 entries) along with the list of victims and a timestamp.

Each streak threshold has a name and a themed banner color, shown in a large animated banner (scale + fade in → settle → hold → fade out) in the center of the screen:

| Kills | Name |
|---|---|
| 2 | Double Kill |
| 3 | Triple Kill |
| 4 | Mega Kill |
| 5 | Ultra Kill |
| 7 | Rampage |
| 10 | Demigod |
| 12 | Wicked Sick |
| 15 | Monster Kill |
| 18 | Ludicrous Kill |
| 25 | Unstoppable |
| 30 | Godlike |
| 40 | Beyond Godlike |
| 50 | Legendary |
| 70 | Mythical |
| 90 | Immortal |
| 100 | Unreal |

The banner's "hold" time scales with streak size (longer for bigger streaks, capped at 4 seconds).

## Multi-Kills

Independent from kill streaks, a **multi-kill** counts kills landed within a rolling **10-second window** (`KDStatTracker.MultiKill.window`). Multi-kills of 2+ are saved to history (last 50 entries) and shown via their own animated banner:

| Kills | Name |
|---|---|
| 2 | Double Kill |
| 3 | Triple Kill |
| 4 | Quadra Kill |
| 5 | Penta Kill |
| 6 | Hexa Kill |
| 7 | Hepta Kill |
| 8 | Octa Kill |
| 9 | Nona Kill |
| 10 | Deca Kill |

Multi-kill chains that go stale (no kill for >10s) are automatically expired and archived by the HUD's periodic update tick, even if you don't land a new kill to trigger the reset.

## Duel Tracking

Listens for `EVENT_DUEL_FINISHED` and records, per character:
- Total duels, wins, and losses (forfeits count as a loss for the forfeiting side).
- A **per-opponent** breakdown keyed by the opponent's `@DisplayName`, so you can check your record against a specific player.

## Battleground Match History

Listens for `EVENT_BATTLEGROUND_STATE_CHANGED`:
- On match start, resets the current kill streak and begins tracking kills/deaths for that match, and identifies the game mode (Deathmatch, Domination, Chaosball, Capture the Relic, King of the Hill, Murderball, Crazy King).
- On match end, stores the match's kills, deaths, game type, and character name into a rolling history of the **last 20 matches**, and prints a summary to chat.

## Slash Commands

| Command | Description |
|---|---|
| `/kd` | Shows your current account-wide K/D (kills, deaths, ratio). |
| `/kdchar` | Shows K/D for your **current character** specifically. |
| `/bgstats` | Lists your last 10 Battleground matches (game mode + K/D). |
| `/duelstats` | Shows total duel wins/losses for your current character. |
| `/killstats` | Lists kill counts broken down by ability. |
| `/deathstats` | Lists death counts broken down by ability. |
| `/mostkills` | Shows the single ability that has gotten you the most kills. |
| `/mostdeaths` | Shows the single ability that has killed you the most. |
| `/duels` | Shows your overall duel win/loss record and W/L ratio. |
| `/duels @DisplayName` | Shows your duel record against a specific player. |
| `/killstreak` (alias `/killstreaks`) | Shows your killstreak history (last 10) and your highest-ever tier. |
| `/killstreak current` | Shows only your currently active kill streak and its victims. |
| `/multikills` | Shows your multi-kill history (last 10) and your highest-ever tier. |
| `/multikills current` | Shows only your currently active multi-kill chain and its victims. |
| `/kdhelp` | Prints the full command list in chat. |

## Settings Menu

Requires **LibAddonMenu-2.0**. Available under *Settings → Add-Ons → KDStatTracker*:

- **Display Settings**
  - `Show KD Display` — toggle the HUD panel on/off.
  - `UI Scale` — 50%–200% (5% steps).
  - `Font Size` — 12–36pt base size for the panel.
  - `Font Color (Hex)` — hex color used for streak/chat message accents (e.g. `FFFFFF`).
- **Position**
  - `X Position` / `Y Position` — sliders bound to screen width/height, mirrors manual drag-to-move.
  - `Reset Position` — snaps the panel back to the default `(500, 500)`.
- **Data**
  - `Reset Killstreak History` — permanently clears saved kill streak history and streak totals (confirmation warning shown).

## Saved Data

Data persists account-wide via `ZO_SavedVars`, under three saved variable tables declared in the manifest:

| SavedVariables key | Purpose |
|---|---|
| `KDStatTrackerVars` | Core stats: total kills/deaths, per-character K/D, match history, duel records, kill/death sources, kill streak & multi-kill history. |
| `KDStatTrackerKDA` | The account-wide running kill/death counters shown by `/kd`. |
| `KDStatTrackerSettings` | HUD display preferences (position, scale, font, visibility). |

All saved tables are versioned (`version 1`) and self-heal missing fields on load, so upgrading between add-on versions won't wipe existing history.

## Installation

1. Download/copy this add-on folder into your ESO add-ons directory:
   ```
   <Documents>\Elder Scrolls Online\live\AddOns\BGStatsTracker\
   ```
   (or the equivalent `AddOnsManaged` cache path if installed via an add-on manager).
2. Make sure the folder contains:
   - `KDStatTracker.addon` (manifest)
   - `KDStatTracker.lua`
   - `KDStatTrackerSettings.lua`
3. Launch ESO, open the **Add-Ons** menu at the character select screen, and enable **KDStatTracker**.
4. In-game, type `/kdhelp` to confirm it loaded and see available commands.

## Requirements / Dependencies

- **ESO API version:** `101046` (set in the manifest; update if the game's API version changes).
- **LibAddonMenu-2.0** — required for the settings panel. Install separately if you don't already have it (commonly bundled with other add-ons or available from ESOUI/Minion).

## File Overview

| File | Responsibility |
|---|---|
| [KDStatTracker.addon](/c:/Users/Admin/AppData/Local/Elder Scrolls Online/pccert/CachedData/AddOnsManaged/BGStatsTracker/KDStatTracker.addon) | Add-on manifest: title, API version, author, dependencies, saved variable declarations, and load order. |
| [KDStatTracker.lua](/c:/Users/Admin/AppData/Local/Elder Scrolls Online/pccert/CachedData/AddOnsManaged/BGStatsTracker/KDStatTracker.lua) | Core logic — HUD construction/theming, combat/duel/battleground event handlers, kill streak & multi-kill engines, banner animations, slash commands, and saved variable initialization. |
| [KDStatTrackerSettings.lua](/c:/Users/Admin/AppData/Local/Elder Scrolls Online/pccert/CachedData/AddOnsManaged/BGStatsTracker/KDStatTrackerSettings.lua) | LibAddonMenu-2.0 settings panel definition (display, position, and data-reset controls). |

## Known Behavior Notes

- Kill/death source tracking (`/killstats`, `/deathstats`, `/mostkills`, `/mostdeaths`) is scoped **per-character** (keyed by `GetUnitName("player")`), while `/kd` reflects the account-wide running total.
- A kill streak only counts if it is **not interrupted by your own death** — dying always resets both the kill streak and the active multi-kill chain.
- Multi-kill and kill streak counters are tracked independently: a multi-kill requires kills within a 10-second window and is also reset by death (`ResetMultiKill`), while a kill streak only resets on death, regardless of how much time passes between kills.
- Battleground match history, duel stats, and kill-streak/multi-kill history each retain a bounded number of most-recent entries (20 matches, 50 streaks, 50 multi-kills) to keep saved variable files small.

## Credits

- **Author:** Vixen Hunny
- **Version:** 3.0
- **Inspired by Himiga** — the original idea and motivation to build KDStatTracker came from Himiga.
- **Base code by Synkronist** — this add-on started from a base/foundation written by Synkronist, which was then extended with the HUD, kill streak/multi-kill engines, duel tracking, battleground history, and settings panel.
