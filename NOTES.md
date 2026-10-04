# AntiVenom Notes — 2026-10-04 (v1.0.59 / skin 4.0.14)

Last GitHub push: **1.0.60** / skin **4.0.14**. Repo: `jaymac4453/venmons` (old `jjmandog12` account was suspended).

**1.0.60:** Live NFL games are green **LIVE** and sort to the top. The NFL folder refreshes every 2 minutes. Finals stay below.

## Issues (what was wrong)

1. **Too many NFL choices.** The Crew `action=nfl` list showed ~30 identical NFL shield tiles (`NFL LIVE (TS)`, `(SC)`, `(RS)`, `(FOXY)`, etc.). Same picture, same label family, no games.
2. **GameDay said 0 LIVE.** Sports chip showed `GAMEDAY: 0 LIVE` during Sunday football. Cache file `gameday_scores.json` was from **2026-10-02** with empty leagues. `apply_gameday_to_skin()` re-applied that file forever and never fetched ESPN again.
3. **NFL slate went stale.** Today's games only loaded when the user opened NFL. Nothing wrote the list in the background.
4. **Same picture on every NFL tile.** Mad Titan thumbs were a generic VS image. Skin `Variables.xml` also forced `nfl.jpg` on any `action=nfl` path even when a poster was set.
5. **Do not invent data.** User asked for real scoreboards and real team art only. No mock live counts. No fake version auto-updater (they declined that).

## Fixes (what we shipped)

1. **Simplified NFL list.** `sportshub.render_nfl()` shows RedZone + one row per game from the real Mad Titan NFL JSON. Click plays that game. The Crew 30-scraper page is not used for NFL.
2. **GameDay auto-write.** `gameday.refresh_all_scores()` pulls ESPN, writes `addon_data/script.antivenom/gameday_scores.json`, updates the chip. Failed fetches keep the last good file. Home + service run this every 2 minutes (`AlarmClock` + service loop).
3. **NFL slate auto-write.** `refresh_nfl_slate()` writes `nfl_slate.json` on the same timer. Rows get LIVE / Final / kickoff from ESPN. Opening NFL reads that file.
4. **Team pictures.** Scan away/home names, download that team's ESPN logo into `addon_data/script.antivenom/nfl_art/`. Even weeks feature home, odd weeks feature away; logo style also rotates by week. Skin no longer overwrites a set poster with the generic shield.
5. **Search buttons** (already in this unpushed batch): larger SEARCH MOVIES / SEARCH SHOWS on Movies and TV.

## Do not regress

- Do **not** remap DaddyLive `stream-N` numbers onto `dlhd.pk` channel IDs. That played the wrong game (SmackDown → Phillies).
- Skip DaddyLive numbered slots, `mytv.fun`, and Turner `/slate/` playlists.
- Sports play order stays exclusive HLS first, then scraper, then The Crew backup.
- Kids titles stay on official YouTube episode lists, never debrid.
- `python dev.py sync` / `release` copies **source → live**. Put working Python in `script.antivenom/` before sync or you wipe the Windows Kodi copy.

## Verify after update

- Sports chip is not stuck at 0 LIVE on Sunday afternoon.
- NFL opens ~15 game tiles, not 30 `NFL LIVE (xx)` scrapers.
- Each game has a different team logo.
- Fire Stick: old `jjmandog12` repo is dead. Sideload **1.0.60** once from `jaymac4453/venmons`, then Check for Updates works again.
