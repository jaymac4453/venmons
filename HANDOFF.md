# Session Handoff Document

**Status:** [COMPLETE]
**Date & Time:** 2026-10-04T14:10:00-07:00
**Previous Conversation / Session ID:** 569b0573-e99b-4cc2-8f9f-9c50683bcaac
**Git Branch / Commit:** main @ GitHub `jaymac4453/venmons` (release **script 1.0.60** / **skin 4.0.14**)

---

## 1. Executive Summary & Objective

- **Completed Milestone:** NFL is a real game list (not 30 scraper clones), GameDay writes live ESPN scores on a timer so it cannot sit at 0 LIVE, the NFL slate auto-writes, and each game uses that team's ESPN logo with a weekly home/away cycle.
- **Architectural Context:** Movies/TV still The Crew + Real-Debrid. Sports: exclusive feed first, scraper backup, The Crew last. NFL UI is AntiVenom `sportshub` + Mad Titan play_video + ESPN scoreboard/art. Kids stay official YouTube.

## 2. Completed Changes & Verified Seams

### Files

- `script.antivenom/resources/lib/sportshub.py` — NFL game list, slate cache, team logo scan/cycle, play one game.
- `script.antivenom/resources/lib/gameday.py` — real ESPN refresh, write cache, never apply a days-old empty file, trigger NFL slate write.
- `script.antivenom/default.py` — `sports_play`, `refresh_gameday` writes scores + NFL slate.
- `script.antivenom/service.py` — startup + periodic GameDay/NFL slate; AlarmClock loop every 2 minutes.
- `script.antivenom/skin.antivenom/xml/Home.xml` — Home onload refreshes GameDay; larger movie/TV search buttons (from prior unpushed work).
- `script.antivenom/skin.antivenom/xml/Variables.xml` — do not force `nfl.jpg` over a set poster.
- `script.antivenom/addon.xml` — **1.0.59**
- `script.antivenom/skin.antivenom/addon.xml` — **4.0.14**
- `NOTES.md`, `HANDOFF.md`, `MISTAKES.md`, `build_repo.py` changelog.

### Issues → fixes

| Issue | Fix |
| --- | --- |
| 30 identical `NFL LIVE (TS/SC/…)` tiles | Route NFL to `render_nfl()`: RedZone + one row per real game |
| Chip `GAMEDAY: 0 LIVE` on Sunday | Stop applying stale empty cache; fetch ESPN; write `gameday_scores.json` every 2 min; keep last good file on fail |
| NFL list only existed after a click | `refresh_nfl_slate()` writes `nfl_slate.json` on the same timer |
| Every tile the same VS/NFL shield | ESPN team logos cached in `nfl_art/`; weekly home/away + style cycle |
| User asked for version auto-check | **Not built** (they said slate only) |

### Verification

- ESPN NFL board on 2026-10-04 had live games (Vikings, Raiders, 49ers, Seahawks).
- Slate write produced 15 rows; after team art, 15 unique logo files.
- GameDay empty-cache age was ~48 hours before the write fix.

### Gaps

- Fire Stick Check for Updates still needs the user (or a later boot) to pull **1.0.59**. Older sticks (report was **1.0.44**) stay behind until they update.
- Kodi service that was already running still has old Python in memory until Kodi restarts or `RunScript(refresh_gameday)` / reopen Sports.
- Windows live `Home.xml` was an older sports layout than source; `dev.py sync` publishes **source** skin 4.0.14.

## 3. Active System State & Working Directory

- Workspace: `C:\Users\xelaa\Downloads\ClaudeCode-20260920T041743Z-1-001\ClaudeCode\AntiVenom`
- Not a local git repo. Publish: `python dev.py release` (sync + build + GitHub contents API).
- Live Kodi: `%APPDATA%\Kodi\addons\...`
- Addon data: `%APPDATA%\Kodi\userdata\addon_data\script.antivenom\`
  - `gameday_scores.json` — ESPN scores
  - `nfl_slate.json` — today's NFL rows + play items
  - `nfl_art\` — ESPN team PNGs

## 4. Cold-Start Directive for Incoming Agent

- Read `NOTES.md` first.
- Never remap DaddyLive stream numbers to dlhd channel IDs.
- Never skip ESPN/slate writes by only loading cache when `apply_gameday_to_skin()` is called with no data.
- After Python edits: update **source** then `python dev.py sync`. Source wins on sync.
- Bump `script.antivenom` and `skin.antivenom` versions before any GitHub push.
- Do not add a silent version auto-installer unless the user asks. They declined it this session.
- Kids = official YouTube episode picker only.

### Key invariants

- Real scoreboards and real team logos only. If a fetch fails, keep the last real file.
- Exclusive sports HLS first; scraper rank ~40; Crew backup window; skip daddy / mytv / `/slate/`.
- NFL directory is `plugin://script.antivenom/?action=sports_league&key=nfl`, not The Crew `action=nfl`.
