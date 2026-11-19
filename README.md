# Delve — handoff notes (current: v0.28, 19 Nov 2026)

The files in this project are the latest build (`delve/Index.html` should be v0.28 — check `GAME_VERSION='0.28'`). **If the project copy still says 0.27.2, the v0.28 upload didn't land — ask Beau for his latest index.html.** (19 Nov: `project_write` with `local_path` was refused because the 1.2 MB file can't be Read in full; Beau was asked to upload v0.28 to the project himself.) **Start every new chat from these:**

- `delve/Index.html` — the whole game (HTML/CSS/JS, one file). Same file is `index.html` on GitHub and `Index` in Apps Script. `GAME_VERSION` and the `VERSIONS` changelog live near the "versions" section.
- `delve/Code.gs` — Apps Script back end (leaderboard, keep, accounts, recordings). Unchanged since v0.23. Note: its `topRuns()` is capped at `TOP_N = 50` — the page no longer uses it for the board (see `allRuns` below); it's only the fallback and the return value of `submitRun`.
- `delve/Api.gs` — (v0.25) second Apps Script file: `doPost` JSON API so the GitHub site can call the same functions (allow-list `API_ALLOWED`). v0.26.1 added `allRuns()` — the whole board, every player, no cap.
- `delve/config.js` — (v0.25) GitHub-only: `window.DELVE_API_URL` (the /exec URL, already filled in) and `window.DELVE_GOOGLE_CLIENT_ID`. Updates to the game never overwrite it.
- `delve/manifest.webmanifest` — PWA manifest.

To pick up work: `project_read` the files, make changes, test, deliver, then `project_write` back (replace in place) and update this README. (Large files: `project_read` of the 1.2 MB game lands in a local temp file; `cp` it into the working directory, edit/test there, and `project_write` with `local_path` — worked on 16 Nov, refused on 19 Nov ("read the whole file first"). If refused, ask Beau to upload the file to the project instead.) Editing tip: the file is too big for the Read/Edit tools; patch it with small Python scripts that replace exact unique strings (watch for `·`/`—` escapes in the source vs literal characters), then `node --check` each `<script>` block.

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

## v0.28 (19 Nov 2026) — gods, and the Village Idiot in numbers
(Built on 0.27.2, then merged with 0.27.3 from GitHub, which was never saved to this project: 0.27.3 (18 Nov) only added the song 'Absolution Apparatus' to the Book III list (`3:[...]` in the soundtrack table, between Red Silhouette and Heartstrings) plus its VERSIONS entry. Check GitHub's latest index.html against the project copy before each round; commit list: api.github.com/repos/beaustapley-ai/delve/commits?path=index.html, readable from the browser pane.)
Beau: the Keeper of Count punished him for "rushing" every time he went down/up a floor to rest. His call: **what each god likes/hates is fixed per god; temperament (humour) and gift tastes stay random per run.** Wants a codex place showing who your god is and what you've learned this run.
- **Fixed likes/hates**: `GODS[i].dl` / `.dh`. count: likes greed, patience / hates thirst · rule: haste, blood / patience · choir: working, thirst / greed · lamp: distance, working / blood · knot: blood, pain / distance, haste · ash: working, pain / thirst, patience. `godMind()` applies them when `f.fixedFor!==P.god` (migrates old saves; drops `godKnow` like/hate/felt clues and `deedSign` entries that are no longer true). Humour still rolled once per run.
- **Haste rework**: `enterLevel` now calls `hasteCheck()` (going down to a fresh floor) and sets `P.arrT=turns` on every arrival. Liked haste → old rule (under 230 turns since `P.floorIn`). Hated haste → only if under 120 turns since arriving (`max(arrT,floorIn)`), `seenShare()<0.34` (seen share of non-wall tiles), HP ≥ 60%, and no awake monster within 8. DEEDS.haste `as` text → "As you hurry on down the stair".
- **Patience**: `rest()` only calls `godDeed('patience')` when at least one turn was rested (refused rests used to count).
- **Cross-run god lore**: `localStorage['delve.godlore']` = `{godId:{like:[],hate:[],runs}}` (per device, not synced). `godLoreAdd(k)` from `godNote` (like:/hate:/felt: keys); `godLoreRun()` from `godTruth()` (morgue reveals all).
- **Codex → "Your god" tab** (`godCodex(live,GL)`): during a run — what the god is, every clue/feeling this run with its floor, and everything known over all runs; plus other gods you know about. From the title-screen codex it's "The gods" (all known lore). Character sheet + prayer text updated ("same every run").
- **Village Idiot text** now mechanical: class blurb (70% more XP, bows/staves −40%, 5% rarity bump), Keep Going, Sit Down, Still Here (one-time +18% max HP, min +20), Lucky (+10 drop / +8 furniture odds), Slow Learner (XP penalty 70%→15%), Picked It Up (removes the clumsy −40% for bows/staves), Odd Jobs. `prayLevelUp` now uses the Slow Learner rate when taken.
- Tested: each god's fixed likes, haste cases (hurt / rushing / explored / Old Rule liking it), old-save migration, save/load, codex in-run and from lore, 7-class random bot — no page errors.

## v0.27.2 (16 Nov 2026)
- Beau: his own traps must never affect him. Player trap trigger in `arrive` now skips `mine` traps; `disarm()` on an own trap lifts it straight back (no roll, no XP, no disarm count). Native traps unchanged.

## v0.27.1 (16 Nov 2026) — reasons to dive, and to come back up
Beau's design calls: hidden quick-descent gifts (yes; no "wanderer" pressure), keys-from-below vaults with loot two floors better, errands + rescues (escort / report back) + trader orders + restocks. Traders before floor 4 are fine — the 4–5 rule is only the maximum gap.
- Most code is one block `/* ═══ v0.27.1 — reasons to go down, and to come back up` just before `$('goBtn').onclick=titleScreen`; `floorExtras27(d)` runs after `floorExtras26(d)` at the end of `genLevel`.
- **Stacked traps**: own (`mine`) traps can share a square (native traps still block). `monTraps(m,x,y,K)` (from `step()` and the ghost-through-door path) fires all of them and marks every one spent even if the monster is already dead/gone. (v0.27.2: the player never sets off own traps.)
- **Arrows**: `arrowsN(n)` = ×2/3 on floor/chest/vault/armoury drops; monster arrow drop chance 11%→7%. Shop arrows unchanged.
- **Vaults**: `makeVault(d)` (36%, floors 2..LAST-4) carves a 3–5×3–4 room into solid `T_WALL` beside reachable ground, ring of `T_HARD`, one `T_LOCKED` door. Stored in `FL.sv` {dx,dy,ox,oy,rx,ry,rw,rh,n,open} and `P.vaults[d]` {n,kf,got,open}. Key = SUPPLY `vaultkey` with `vf:d,en:name`, placed on floor `kf=d+2..d+4` by `placeVaultKeys` (or `injectStored` if that floor already exists from a chart). `isVaultDoor` blocks tryDoor/picks/bust, the unlock spell, break stone, the engine-house lever, and wall-breaking monsters; `vaultGuard()` (top of `endTurn`) puts anyone found inside a shut vault back outside. `vaultDoor()` opens it: loot made at `L=max(kf, deepestWalked())+2` — one gear piece a rarity up, two `makeItem(L)`, maybe a potion, gold.
- **Jobs** (`P.jobs`, `jobById`, `jobLines` on the character sheet): `JOB_ROLES` get `n.offer` 55% in `floorExtras27`. `talk()` → `jobTalk(n)` first, else `talkBase(n)`. `offerFor` picks fetch (55%) or rescue; target floor = deepest walked +1..3. `placeJob` puts the errand item (SUPPLY `errand`), or for a rescue a `strayed` NPC (alive 55%) or a keepsake token. Strayed NPCs follow (`n.follow`, `followStep`), take stairs if within 2 squares (`enterLevel`), `escortArrive()` on the giver's floor. Rewards (`jobReward`): gear / permanent stat (`JOB_STATS`) / skill pick.
- **Orders/restock**: `ORDER_WORDS[kind]`, `orderMenu`, `orderReady(n)`, `restock27(n,kind)`. `deepestWalked()` uses `P.st.fl` (charts don't count — `deepestFloor()` does).
- **Quick-descent gift (hidden)**: `quickStart(d)` stores `P.qk[d]={t,par}`; `quickCheck(from)` on the stair down. Never shown in the UI.
- **Map**: NPCs get `found` when in view; map book draws found people; tap shows `npcInfo(n)`. Legend adds `vaultNote(d)`.
- **Score**: parts `jobs` (150), `vaults` (100), `quick` (60); record fields `r.jobs/r.vaults/r.quick`.

## v0.27 (16 Nov 2026) — playtest round
- **Immunities on direct damage** (`flyTo0` → `flyElem(rider)` → `SPELL_ELEM`; `immuneHit`). `IMM_RX`, `monImmune`, `m.imm`.
- **Player-set traps** (`monTrap`) handle every trap kind; only `mine:1` traps catch monsters.
- **FP food/drink**: `PROV_EFF` bright/brimming/fogging/emptying; 15 SUPPLY entries after `stairkey`.
- **See invisible by depth**: `seeInvOdds(d)=clamp((d-2)*3.2,0,75)`.
- **Merchants**: `marketPlan()` → `P.mkt={gap,odds}`; forced seller when `d-(P.shopD||0)>=gap`; markets.
- **Paid naming**: `NAME_TRADE[kind]`, `nameMenu(n)`, `namePrice`; buying names the item.
- **Village Idiot**: `luck:0.05` → `dumbLuck(i)` in `rollRarity`.
- **Cutpurse Vanish**: 5 FP, 4 turns; Second Shadow 10.

## Brightness (v0.26.3)
**Settings → Display** (`displayPane`): brightness 80–180% (`BRIGHT.b`). CSS `filter` on `<html>` via `DELVE_BRIGHT_FILTER`. Stored per device in `localStorage['delve.bright']`.

## Code conventions worth knowing
- Map tiles packed one char per tile via `tileCh`/`chTile`; ids ≥36 use `TILE_EXTRA`.
- `SUPPLY`, `TRAPS`, `POTIONS`, `SPELLS`, `MONS`, `WANDS` (and other item tables) are indexed in saves — only ever **append**. `progressCompat()` pads `P.potMap`/`pLook`, `bLook`/`sLook`; the load path gives new SUPPLY entries an `fLook`. `BOOK_LOOKS`/`SCROLL_LOOKS` must have at least as many entries as `SPELLS`.
- Monsters are re-skinned per place by `themedKind`; flags carry through. Immunity follows the re-skinned name.
- Weapon slots: `P.wep` = melee, `P.off` = ranged/wand. Class weapon affinity: `hand` (+15%), `poor` (−40%) via `affinity`/`wepBonus`.
- Server calls: `window.DELVE_ACC(method,args,cb)`; GitHub fetch shim fakes `google.script.run`.
- HTML attributes don't interpret JS escapes — use entities.
- Settings tabs: `settingsScreen(tab,back)`. Testing a run: `begin(CLASSES[i], QUESTS[j])`; move with `act(dx,dy)`; `enterLevel(d,'down',0)`; `doSave(0)` needs `setCharName(..)` first; `doLoad(0)`. Codex: `codexHome='game';showCodex('god')`.

## v0.26 systems
- **Prayer (Nethack-like, hidden).** `P.god` per run from 6 `GODS` (`rollGod`: class god 35%, else random); `godMind()` fills likes/hates (fixed per god since v0.28) and humour; hidden `P.standing`, `P.prayT`, `P.godAnger`. `godDeed(id,amt)` hooks; clues from altars and prayer scraps (`godClue`, `godNote`); `pray()`/`prayAsk(Pr)`; morgue shows `godTruth()`.
- **See invisible**: `seesInvis(m)`.
- **Monster statuses**: `m.slowT`, `m.weakT`; `MON_STATUS`/`monMarks`, `drawMonAuras`.
- **Shot FX**: `flyTo` wraps `flyTo0`; `shotFx`/`burstFx`.
- **Spells** ids 24–31 via `castNew()`.
- **Maps**: `placeRiddleGate(d)`, `rewardNooks(d)`, `placeSpellbook` from `floorExtras26(d)`.

## Recent versions
- 0.28 — each god's likes/hates fixed (temper and gifts still random); haste only counts genuinely skipping a floor; Codex "Your god" tab with cross-run lore; Village Idiot text in numbers.
- 0.27.2 — own traps never hurt the player; lifting your own trap back is free.
- 0.27.1 — stacked traps; fewer arrows; vaults; errands, rescues, orders, restocks; quick-descent gifts; people on the map.
- 0.27 — immunities stop direct elemental damage; player traps on monsters; FP food/drink; see-invisible; traders every 4–5 floors; paid naming; Village Idiot dumb luck; Vanish change.
- 0.26.3 — brightness. 0.26.2 — catch-up sync. 0.26.1 — leaderboard fetches every run. 0.26 — prayer rebuilt etc. 0.25 — soundtrack, GitHub Pages. 0.24 — levelling rebuilt.
