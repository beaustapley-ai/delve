# Delve — handoff notes (current: v0.26.1, 16 Nov 2026)

The files in this project are the latest build (`delve/Index.html` is v0.26.1 — check `GAME_VERSION='0.26.1'`). **Start every new chat from these:**

- `delve/Index.html` — the whole game (HTML/CSS/JS, one file). Same file is `index.html` on GitHub and `Index` in Apps Script. `GAME_VERSION` and the `VERSIONS` changelog live near the "versions" section.
- `delve/Code.gs` — Apps Script back end (leaderboard, keep, accounts, recordings). Unchanged since v0.23. Note: its `topRuns()` is capped at `TOP_N = 50` — the page no longer uses it for the board (see `allRuns` below); it's only the fallback and the return value of `submitRun`.
- `delve/Api.gs` — (v0.25) second Apps Script file: `doPost` JSON API so the GitHub site can call the same functions (allow-list `API_ALLOWED`). v0.26.1 added `allRuns()` — the whole board, every player, no cap.
- `delve/config.js` — (v0.25) GitHub-only: `window.DELVE_API_URL` (the /exec URL, already filled in) and `window.DELVE_GOOGLE_CLIENT_ID`. Updates to the game never overwrite it.
- `delve/manifest.webmanifest` — PWA manifest.

To pick up work: `project_read` the files, make changes, test, deliver, then `project_write` back (replace in place) and update this README. (Large files: `project_read` of the 1.2 MB game lands in a local temp file; `cp` it into the working directory, edit/test there, and `project_write` with `local_path` — that worked on 16 Nov. If it's ever refused, ask Beau to upload the file to the project instead.)

## Hosting (v0.25)
- **Main home: GitHub Pages.** Repo https://github.com/beaustapley-ai/delve (personal, beaustapley@hotmail.com). Game: https://beaustapley-ai.github.io/delve/
  - Repo root: `index.html`, `config.js`, `manifest.webmanifest`, `icon-192.png`, `icon-512.png`, `music/album1…album5/*.mp3` (Beau uploads songs himself).
  - To update: upload the new `index.html` only (config.js stays).
- **Back end: Apps Script** on the personal account — `Code.gs`, `Api.gs`, HTML file `Index`. Deployment: Execute as Me, access **Anyone**. Sheet `16TZ_HZ0aMCk-kCZYvbEhEDOj1tN2vF84WASOHxgCjQQ` (tabs Leaderboard, Saves, Accounts, Replays).
- Claude can read the repo through the built-in browser pane (github.com and beaustapley-ai.github.io allowed), not from the cloud sandbox. Claude can't push — Beau uploads. The cloud sandbox can't reach script.google.com either (proxy refuses), so the live board can't be queried from there.

## Working agreement
- Beau sends lists of playtest feedback; implement every item, test, then deliver.
- Deliver built files as attachments. **No published artifact.**
- Say which files changed and remind him: upload index.html to GitHub; paste into Apps Script + New version if back end changed.
- Bump `GAME_VERSION` and add a dated `VERSIONS` entry each round.

## Leaderboard rule (Beau, 16 Nov 2026)
**Every run by every player is fetched and visible to everyone.** Never filter or hide board entries by player or account. The table draws the top `REC_SHOW = 100` rows *in the current sort order* (Beau's call), but the whole board is always fetched so each run's # is its place out of all runs and tab counts are right; sorting by another column (Player, When…) reaches every run. (v0.26.1 fixed the top-50 cap that hid lower-scoring players like Zaxaro.)
- Page: `DELVE_FETCH_BOARD` calls `allRuns` (falls back to `topRuns` if the back end is older). `recordScreen` re-fetches every time it opens (cached rows drawn first, then refreshed, scroll kept). `boardRows` ranks against the full list, draws the first `REC_SHOW`, and adds a "Showing the first 100 of N" note. `mergeRuns` dedupes only on a non-zero `when`, so rows with no time never swallow each other.

## Code conventions worth knowing
- Map tiles packed one char per tile via `tileCh`/`chTile`; ids ≥36 use `TILE_EXTRA`.
- `SUPPLY`, `TRAPS`, `POTIONS`, `SPELLS`, `MONS`, `WANDS` (and other item tables) are indexed in saves — only ever **append**. `progressCompat()` pads `P.potMap`/`pLook`, and (v0.26) `bLook`/`sLook` for appended spells; `BOOK_LOOKS`/`SCROLL_LOOKS` must have at least as many entries as `SPELLS`.
- Monsters are re-skinned per place by `themedKind` (name/glyph/colour by "part"); flags like `sap`, `seeInv` carry through.
- Weapon slots: `P.wep` = melee, `P.off` = ranged/wand.
- Server calls: `window.DELVE_ACC(method,args,cb)`; GitHub fetch shim fakes `google.script.run`.
- HTML attributes don't interpret JS escapes — use entities.
- Most v0.26 code lives in two blocks: the prayer block (where `/* ═══ v0.26 — gods who keep their own counsel` starts, replacing the old favour code) and a block just before `$('goBtn').onclick=titleScreen` (see-invisible, statuses, FX, new spells, map rewards).

## v0.26 systems
- **Prayer (Nethack-like, hidden).** `P.god` is rolled per run by `rollGod(c)` from 6 `GODS` (count, rule, choir, lamp, knot, ash; each has `about`, `boon`, `wrath`). `godMind()` lazily fills `P.favour.likes[2]`, `.hates[1–2]` (ids in `DEEDS`: blood, distance, working, haste, patience, greed, thirst, pain), `.humour` (`HUMOURS`: proud, generous, fickle, stern, merciful), plus the old gift tastes (`good`, `bad`, `read`). Hidden numbers: `P.standing` (−100..100), `P.prayT` (turn it is ready again), `P.godAnger`, `P.pester`, `P.greedN`.
  - `godDeed(id,amt)` from hooks (slay → blood/distance; castSpell → working; enterLevel → haste if <230 turns on floor; search/rest → patience; gold pickup → greed; drinkFlask → thirst; hurt below 30% → pain). Per-floor diminishing (`P.deedF`). First time a cared-about deed happens: 55% chance of a sign line + journal note.
  - Clues: `godClue()` → `readGodClue('altar'|'scrap')`. Altars (`altarClue()` on stepping, once per altar, 70%), prayer scraps = loot `{t:'page',gc:1}` (50%/floor + in secret areas). Journal `P.godKnow` shown in the prayer modal and character sheet (`godKnowLines()`).
  - `pray()` shows requests (`PRAYERS`, tier 0 trust / 1 minor / 2 major / 3 great) with no costs; on an altar it shows mood (`godMood`, `godWait`) and offering cards (`godOffer` → `godOfferValue`, capped 16 standing/floor, eases anger first).
  - `prayAsk(Pr)`: anger/standing<−30 → `godSmite`; too soon → displeased (+wait, −standing), twice → anger + smite; merciful/stern change the grace window when in major trouble. Then: fixes all major troubles (`godTroubles`/`godFix`), then grants / swaps for an insight (`godInsights`: names a bad flask, lifts an unknown curse, reveals the worst unseen monster + ward, drops food/water, arrows, shows traps/stairs). Great asks need high standing (50, proud 75, generous 40, +10 per previous great grant).
  - Morgue shows the truth (`godTruth()`).
- **See invisible**: `seesInvis(m)` = `m.seeInv` (rolled in `makeMon`, `seeInvOdds(d)` up to 40%), bosses, minis from floor 8, ghost/invis kinds from book II. `senses`, `hunted`, the invis tick and `moveMons` respect it; `seeInvisTurn()` at the top of `endTurn` keeps them on you and logs it. Drawn: blind monsters go grey while you're unseen, seers keep colour + small eye (`monMarks`).
- **Monster statuses**: `m.slowT` (acts every other turn, no double steps), `m.weakT` (×0.65 attack); rings via `MON_STATUS`/`monMarks`, moving particles via `drawMonAuras` (FX canvas), look text `monStatusWords`.
- **Shot FX**: `flyTo` wraps `flyTo0` and records `FLY_TRACE`; `shotFx(...)` / `burstFx(...)` queue animations drawn by `drawCombatFx` inside `fxLoop` (muzzle ring, projectile, burst on monster, X-mark + grit on walls). Enemy shots (`monShoot`) and casts (`monCast`) too.
- **Spells** appended (ids 24–31): rime, venom, kindle, rend, sleet, miasma, sap, arc — handled in `castNew()` before `castSpell0`. Riders via `flyRider`. Wands `rimew`, `venomw`. `key` also opens gates/stair grates (`unlockGateNear`); `shatter` breaks gates.
- **Focus drinkers**: MONS appended with `sap:1` (wick-eater, thought-leech, hush-sister, pale mothkin, unmaker of words); melee/ranged drain FP and do 35–40% damage. `MON_WORKS` `siphon` for sap/drain (and mind casters vs FP-heavy players).
- **Maps**: `placeRiddleGate(d)` puts a gate only where it cuts off a region (≥6 floor tiles) that holds no stair, with an item inside. `rewardNooks(d)` — every region behind a seam/crack gets loot; rooms off the up→down path get something 72% (bare desc re-rolled). `placeSpellbook` (gap ≥3 floors, or 15% at 2). Shops: scrivener 1–2 books, relic 35%, general 25%. All run from `floorExtras26(d)` at the end of `genLevel`.
- **Item visibility**: items drawn on a dark rounded plate with a rim in the item's colour.

## Testing
Playwright + headless Chromium against `file:///…/Index.html`. v0.26: 24-floor generation sweep (no gate blocks a stair, every secret region has loot), every prayer request under several states, clue/deed/offer paths, all new spells against dummy monsters, FX screenshots mid-flight, see-invisible display, v0.25 save loaded into v0.26, 7-class random bot (shallow and deep) with save/load.
v0.26.1: stubbed `google.script.run` (addInitScript Proxy) with 152 runs across 4 players — `allRuns` path shows all 152 incl. every low-scoring player and two `when:0` rows; old `topRuns` fallback reproduces the 50-row cap; re-opening the record re-fetches.

## Recent versions
- 0.26.1 — leaderboard fetches every run by everyone (server `allRuns` in Api.gs), ranks against all of them, shows top 100 in the current sort; record re-fetches each time it opens.
- 0.26 — prayer rebuilt (per-run god and hidden wants, clues, needs over asks, anger); 8 new spells; status rings/particles on enemies; drawn shots and impacts; spellbooks every ~3 floors and in shops; FP-draining enemies; see-invisible enemies; gates only guard side areas, unlock/break stone open them; rewards in side rooms and secret areas; item plates.
- 0.25 — soundtrack; GitHub Pages home + Api.gs; phone layouts; prayer level/HP/gamble; thrown flasks; immunities; invisibility flask; blessed/cursed flasks.
- 0.24 — levelling rebuilt; trader prices; 30 places; lurkers, new traps; named/chief/dread enemies.
