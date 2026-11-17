# Delve — handoff notes (current: v0.27.2, 16 Nov 2026)

The files in this project are the latest build (`delve/Index.html` is v0.27.2 — check `GAME_VERSION='0.27.2'`). **Start every new chat from these:**

- `delve/Index.html` — the whole game (HTML/CSS/JS, one file). Same file is `index.html` on GitHub and `Index` in Apps Script. `GAME_VERSION` and the `VERSIONS` changelog live near the "versions" section.
- `delve/Code.gs` — Apps Script back end (leaderboard, keep, accounts, recordings). Unchanged since v0.23. Note: its `topRuns()` is capped at `TOP_N = 50` — the page no longer uses it for the board (see `allRuns` below); it's only the fallback and the return value of `submitRun`.
- `delve/Api.gs` — (v0.25) second Apps Script file: `doPost` JSON API so the GitHub site can call the same functions (allow-list `API_ALLOWED`). v0.26.1 added `allRuns()` — the whole board, every player, no cap.
- `delve/config.js` — (v0.25) GitHub-only: `window.DELVE_API_URL` (the /exec URL, already filled in) and `window.DELVE_GOOGLE_CLIENT_ID`. Updates to the game never overwrite it.
- `delve/manifest.webmanifest` — PWA manifest.

To pick up work: `project_read` the files, make changes, test, deliver, then `project_write` back (replace in place) and update this README. (Large files: `project_read` of the 1.2 MB game lands in a local temp file; `cp` it into the working directory, edit/test there, and `project_write` with `local_path` — worked again on 16 Nov for v0.27. If it's ever refused, ask Beau to upload the file to the project instead.) Editing tip: the file is too big for the Read/Edit tools; patch it with small Python scripts that replace exact unique strings (watch for `·`/`—` escapes in the source vs literal characters), then `node --check` each `<script>` block.

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
**Every run by every player is fetched and visible to everyone.** Never filter or hide board entries by player or account. The table draws the top `REC_SHOW = 100` rows *in the current sort order* (Beau's call), but the whole board is always fetched so each run's # is its place out of all runs and tab counts are right; sorting by another column (Player, When…) reaches every run.
- Page: `DELVE_FETCH_BOARD` calls `allRuns` (falls back to `topRuns` if the back end is older). `recordScreen` re-fetches every time it opens. `boardRows` ranks against the full list, draws the first `REC_SHOW`, and adds a "Showing the first 100 of N" note. `mergeRuns` dedupes only on a non-zero `when`.
- v0.26.2 catch-up sync: `syncRunsUp(board)` sends local `delve.records` the shared board lacks.

## v0.27.2 (16 Nov 2026)
- Beau: his own traps must never affect him. Player trap trigger in `arrive` now skips `mine` traps; `disarm()` on an own trap lifts it straight back (no roll, no XP, no disarm count). Native traps unchanged.

## v0.27.1 (16 Nov 2026) — reasons to dive, and to come back up
Beau's design calls: hidden quick-descent gifts (yes; no "wanderer" pressure), keys-from-below vaults with loot two floors better, errands + rescues (escort / report back) + trader orders + restocks. Traders before floor 4 are fine — the 4–5 rule is only the maximum gap.
- Most code is one block `/* ═══ v0.27.1 — reasons to go down, and to come back up` just before `$('goBtn').onclick=titleScreen`; `floorExtras27(d)` runs after `floorExtras26(d)` at the end of `genLevel`.
- **Stacked traps**: own (`mine`) traps can share a square (native traps still block). `monTraps(m,x,y,K)` (from `step()` and the ghost-through-door path) fires all of them and marks every one spent even if the monster is already dead/gone. (v0.27.2: the player never sets off own traps.)
- **Arrows**: `arrowsN(n)` = ×2/3 on floor/chest/vault/armoury drops; monster arrow drop chance 11%→7%. Shop arrows unchanged.
- **Vaults**: `makeVault(d)` (36%, floors 2..LAST-4) carves a 3–5×3–4 room into solid `T_WALL` beside reachable ground, ring of `T_HARD`, one `T_LOCKED` door. Stored in `FL.sv` {dx,dy,ox,oy,rx,ry,rw,rh,n,open} and `P.vaults[d]` {n,kf,got,open}. Key = SUPPLY `vaultkey` with `vf:d,en:name`, placed on floor `kf=d+2..d+4` by `placeVaultKeys` (or `injectStored` if that floor already exists from a chart). `isVaultDoor` blocks tryDoor/picks/bust, the unlock spell, break stone, the engine-house lever, and wall-breaking monsters; `vaultGuard()` (top of `endTurn`) puts anyone found inside a shut vault back outside. `vaultDoor()` opens it: loot made at `L=max(kf, deepestWalked())+2` — one gear piece a rarity up, two `makeItem(L)`, maybe a potion, gold. Sweep: 112 vaults / 345 floors, never cuts off a stair, key always placed.
- **Jobs** (`P.jobs`, `jobById`, `jobLines` on the character sheet): `JOB_ROLES` (scholar, survivor, wounded, lamp, digger, pilgrim, deserter, chronicler) get `n.offer` 55% in `floorExtras27`. `talk()` → `jobTalk(n)` first, else `talkBase(n)` (the old talk). `offerFor` picks fetch (55%) or rescue; target floor = deepest walked +1..3. `placeJob` puts the errand item (SUPPLY `errand`, `job:id,en:name`), or for a rescue either a `strayed` NPC (alive 55%, role in NPCS with floors [99,99], `nm` name) or a keepsake token (`tok:1`). Strayed NPCs follow (`n.follow`, `followStep` in `moveNpcs`), take stairs if within 2 squares (handled in `enterLevel`), and on the giver's floor `escortArrive()` walks them to the giver (`J.st='back'`). If the strayed dies → token dropped (`jobFalls`); giver dies → failed. Rewards (`jobReward`): big = choose named/better gear, a random permanent stat (`JOB_STATS`), or a skill pick (`P.picks++`, `offerSkills`); token = gear or gold. Errand items/keys can't be sold.
- **Orders/restock**: `ORDER_WORDS[kind]` → "Place a special order" in `openShop`; `orderMenu` quotes `orderItems(kind, deepestWalked()+4)` at 80% price, `n.order={items,at:deepestWalked()+2}`; `orderReady(n)` hands it over (named) when you open the shop after reaching `at`. `restock27(n,kind)` adds 2–4 stock items each time you come back having gone 2 floors deeper than last time. `deepestWalked()` uses `P.st.fl` (charts don't count — `deepestFloor()` does).
- **Quick-descent gift (hidden)**: `quickStart(d)` on a fresh floor stores `P.qk[d]={t,par}` with par = 70 + 5×(walking distance from arrival to a down stair) (+90 if the stairwell is locked); `quickCheck(from)` when you take the stair down. Under par → gold + FP/health/XP; under 60% of par → plus an item. Never shown in the UI; pars measured 140–560 turns.
- **Map**: NPCs get `found` when in view (`fov`); the map book draws only found people (legacy saves: undefined = shown) on a ringed plate; tapping within 1 square shows `npcInfo(n)` (trader trade, order, job) above the mark editor. Legend adds `vaultNote(d)`.
- **Score**: new parts `jobs` (150), `vaults` (100), `quick` (60, "Floors left in good time"); record fields `r.jobs/r.vaults/r.quick`.
- Tested: stacked traps (monster + player), vault sweep + open/no-key/spell/guard, fetch + rescue alive (escort two floors) + rescue dead (token), order + restock, map tap, save/load mid-escort, par values, 7-class bot.

## v0.27 (16 Nov 2026) — playtest round
- **Immunities on direct damage.** Before, immunity only blocked the *status* (burning/poisoned), not the hit itself. Now `flyTo0` checks `flyElem(rider)` → `SPELL_ELEM` (`ember`,`kindle` → burn; `venom`,`miasma` → poison, read from `CUR_SPELL` while `FLY_KIND==='spell'`, so wands/scrolls count) and an immune monster takes nothing (`immuneHit` logs it). Also: miasma skips poison-immune, the branding (`P.sk.burnMark`) fire spread skips burn-immune. `IMM_RX.burn` gained pyromancer/obsidian/caldera. Immunity is still name-based per floor theme (`monImmune` → `IMM_RX` on `monKind(m).n`) plus `m.imm`.
- **Player-set traps** (`monTrap`, called from `step()` — and now from the ghost-through-door path too) handle every trap: pit/crumble remove the monster (bosses just get held), snare/noose/webshot/grasp hold, hush makes it lose you, vent burns (nothing if burn-immune), gas poisons (nothing if poison-immune), dart adds a little poison, collapse hits harder, sluice/undertow knock down. Only traps with `mine:1` (lifted and re-set) catch monsters — native traps don't, by design.
- **FP food/drink**: new `PROV_EFF` `bright` (+5+d/3 FP), `brimming` (fills FP, rare), `fogging` (−4+d/4 FP + 3 muddle), `emptying` (all FP). 15 SUPPLY entries appended after `stairkey` (lamptea…clearspring), spread across books 0–4 and "anywhere". Older `focusing`/`dulling` still exist. More FOOD_LOOKS/DRINK_LOOKS added at the front of the lists.
- **See invisible by depth**: `seeInvOdds(d)=clamp((d-2)*3.2,0,75)` → 3% f3, 10% f5, 26% f10, 42% f15, 58% f20, 74% f25 (was capped at 40%).
- **Merchants**: `marketPlan()` rolls `P.mkt={gap:4|5, odds:12–51}` per run. In `genLevel`, a seller is forced when `d-(P.shopD||0)>=gap` (or `P.shopD===d`, for regen retries); `P.shopD` = last floor with a trader. With chance `odds` (twice for a third) other trader roles join the same shop room (`shopR.market=1`). Sweep of 40 runs: max gap always ≤ gap, 0–6 markets a run.
- **Paid naming**: shop shelves are no longer labelled for free (`traderTells` now only does `learnGear` for steel; old body kept as `traderTellsOld`). Shop cards use `describeItem(it,0)`. Buying names the item (`learnItem`). `NAME_TRADE[kind]` decides who names what — food: supply; relic: potion/ring/wand; paper: all workings + gear; iron: gear; general (pack-trader, Waykeeper): anything ×1.5; keys: nothing. `nameMenu(n)` lists your pack + their shelf (deduped by `identKey`), priced by `namePrice` (food max(12+depth, worth×0.6); else max(30, worth×0.3)). `identifiable2` adds unknown food; `learnItem` handles supply.
- **Village Idiot**: class field `luck:0.05`; `rollRarity` → `dumbLuck(i)` bumps one tier 5% of the time (avg tier +0.05 measured).
- **Cutpurse Vanish**: 5 FP (was 3), 4 turns (was 6); Second Shadow 10 (was 12; old saves capped via `Math.min(P.sk.vanish,10)`).
- Tested: ember/kindle/venom/wand vs `m.imm` and themed names (brimstone rat, marsh rat), every trap kind on a monster, a trap lifted + set via `useCarry`, all new foods, shop + naming menus for six roles, buy-learning, 40-run shop sweep, idiot rarity stats, save/load, 7-class random bot to ~floor 10 — no page errors.

## Brightness (v0.26.3)
**Settings → Display** (`displayPane`): brightness 80–180% (`BRIGHT.b`), quick set, Shadows lift, defaults. CSS `filter` on `<html>` via `DELVE_BRIGHT_FILTER` in a `<script>` at the top of `<head>`. Stored per device in `localStorage['delve.bright']` — not synced to the account.

## Code conventions worth knowing
- Map tiles packed one char per tile via `tileCh`/`chTile`; ids ≥36 use `TILE_EXTRA`.
- `SUPPLY`, `TRAPS`, `POTIONS`, `SPELLS`, `MONS`, `WANDS` (and other item tables) are indexed in saves — only ever **append**. `progressCompat()` pads `P.potMap`/`pLook`, `bLook`/`sLook`; the load path gives new SUPPLY entries an `fLook`. `BOOK_LOOKS`/`SCROLL_LOOKS` must have at least as many entries as `SPELLS`.
- Monsters are re-skinned per place by `themedKind` (name/glyph/colour by "part"); flags like `sap`, `seeInv` carry through. Immunity follows the re-skinned name.
- Weapon slots: `P.wep` = melee, `P.off` = ranged/wand.
- Server calls: `window.DELVE_ACC(method,args,cb)`; GitHub fetch shim fakes `google.script.run`.
- HTML attributes don't interpret JS escapes — use entities.
- Most v0.26 code lives in two blocks: the prayer block (`/* ═══ v0.26 — gods who keep their own counsel`) and a block just before `$('goBtn').onclick=titleScreen` (see-invisible, statuses, FX, new spells, map rewards). Immunity helpers (`IMM_RX`, `monImmune`, `SPELL_ELEM`, `flyElem`, `immuneHit`) sit in the "what a thing cannot be hurt by" block.
- Settings tabs: `settingsScreen(tab,back)` with panes `keysPane`, `buttonsPane`, `layoutPane`, `displayPane`, `soundPane`, `accountPane`. Testing a run: `begin(CLASSES[i], QUESTS[j])`; `doSave(0)` needs `setCharName(..)` first; `doLoad(0)`.

## v0.26 systems
- **Prayer (Nethack-like, hidden).** `P.god` per run from 6 `GODS`; `godMind()` fills likes/hates (`DEEDS`) and humour; hidden `P.standing`, `P.prayT`, `P.godAnger`, `P.pester`, `P.greedN`. `godDeed(id,amt)` hooks; clues from altars and prayer scraps; `pray()`/`prayAsk(Pr)`; morgue shows `godTruth()`.
- **See invisible**: `seesInvis(m)` = `m.seeInv` (rolled in `makeMon` with `seeInvOdds(d)`), bosses, minis from floor 8, ghost/invis kinds from book II.
- **Monster statuses**: `m.slowT`, `m.weakT`; rings via `MON_STATUS`/`monMarks`, particles via `drawMonAuras`.
- **Shot FX**: `flyTo` wraps `flyTo0` and records `FLY_TRACE`; `shotFx`/`burstFx` drawn by `drawCombatFx`.
- **Spells** appended (ids 24–31): rime, venom, kindle, rend, sleet, miasma, sap, arc — `castNew()` before `castSpell0`. Riders via `flyRider`.
- **Focus drinkers**: MONS with `sap:1`.
- **Maps**: `placeRiddleGate(d)`, `rewardNooks(d)`, `placeSpellbook`; all from `floorExtras26(d)` at the end of `genLevel`.

## Recent versions
- 0.27.2 — own traps never hurt the player; lifting your own trap back is free.
- 0.27.1 — stacked traps; a third fewer arrows; sealed vaults with keys from below; errands, rescues (escort / report back), trader special orders and restocks; hidden quick-descent gifts; people on the map with tap-to-identify.
- 0.27 — immunities stop direct elemental damage; all player-set traps work on monsters and respect immunity; FP food/drink; more see-invisible deeper; a trader every 4–5 floors, markets, paid naming (shelves no longer free-labelled); Village Idiot dumb luck; Vanish 5 FP / 4 turns.
- 0.26.3 — brightness: Settings → Display.
- 0.26.2 — catch-up sync of unsent runs.
- 0.26.1 — leaderboard fetches every run by everyone.
- 0.26 — prayer rebuilt; 8 new spells; status rings; drawn shots; spellbooks; FP-draining enemies; see-invisible enemies; gates; side-room rewards; item plates.
- 0.25 — soundtrack; GitHub Pages home + Api.gs; phone layouts; thrown flasks; immunities; invisibility flask; blessed/cursed flasks.
- 0.24 — levelling rebuilt; trader prices; 30 places; lurkers, new traps; named/chief/dread enemies.
