# Solaris Gambit: road to v1.0.0 (release plan)

*Brief for Claude Code. Written 22 Nov 2026 after a review of the repo `kralnu90/Solaris-Gambit` against the Kralnu Standard.*

## Goal

Take Solaris Gambit from "content-complete living project" to a **finished, release-ready v1.0.0**, one that meets every Kralnu Games house rule and could go on the Play Store as the last step.

The game's content is done and needs no new features: eight main rivals, four element chapters (Fire & Ice, Shadow & Holy, Water & Thunder, Earth & Wind), the Gauntlet, weekly tournaments, puzzles, daily challenge, Altar, signature card, 29+ achievements and safe saves. **This work is about the shell, the polish, a proper finale, and the release.** Don't redesign the core card game or rebalance anything without evidence (see Session 3).

---

## The project at a glance

- **Live:** https://kralnugames.com/Solaris-Gambit/ (GitHub Pages, custom domain via the hub repo `kralnu90.github.io`)
- **Repo files:** `index.html` (~179 KB, single file, plain JS + canvas/DOM, no build step, **minified-style long lines**: use `grep -n`/Python anchors, don't try to read it whole), `sw.js`, `manifest.webmanifest`, `version.json`, `README.md`, `LICENSE`, `icons/`
- **Save keys:** `localStorage['ninefold-save']` plus an IndexedDB backup (`readBackup()` / `safeBackup`). Dev saves use `ninefold-dev`. The old name "Ninefold" is baked into these keys. **Never rename them**, because players would lose progress. All new save fields go through `migrate()` with safe defaults.
- **Version:** `const APP_VERSION='16'` in `index.html`, `{"version": "16"}` in `version.json`, `const VERSION='sg-v16'` in `sw.js`. `checkUpdate()` compares with `!==`, so switching to `1.0.0` is safe for old clients.
- **Service worker** only clears caches that start with `sg-`, so other games on the domain keep theirs. Keep that behaviour.
- **Shared kits already in the file:** the Kralnu install kit (Install button, `beforeinstallprompt`, iPhone instructions) and `window.KralnuUpdate` (update banner + Get Latest Version). Reuse these; don't rebuild them.
- **Review form:** `https://forms.gle/MMDpbhCNALUvpBe68` (already wired to the Review button).

## House rules (Kralnu Standard: follow these, don't ask about them)

**Menus**
- No "shotgun" menus. **Max 4–6 buttons per screen**; group the rest into sub-menus or panels.
- Tile/grid buttons with **icon + label** (Kris's preferred style).
- Play/Continue is the biggest, most obvious button.
- Buttons only show when useful: Install hides once installed, update prompt only when there's an update.
- Every sub-menu has a **Back button in the same place**.

**Main menu:** Play (primary) → mode select · Settings · More panel (How to Play, What's New, Credits, Review, Support, Get Latest Version) · Install (conditional) · version number small in a corner (e.g. `v1.0.0`).

**Settings:** separate **Music** and **SFX** volume · control options where relevant · **Reset save data with a confirm pop-up** · settings remembered between visits.

**Credits:** "Ideas, design and testing by Kris. Built with Claude, an AI by Anthropic." · Kralnu Games name/logo · kralnugames.com · hello@kralnugames.com · any third-party fonts/assets with licences (the game uses Google Fonts **Cinzel** and **Alegreya Sans**, both SIL Open Font License) · "© 2026 kralnu90. All rights reserved."

**Support:** links to ko-fi.com/kralnu90. Invitation wording only, e.g. "Enjoying the game? You can support Kralnu Games on Ko-fi." No nag pop-ups, never ask outright for money.

**Saving:** auto-save (try/catch) plus a suspend save when the app is hidden **or closed**.

**Gameplay:** a pause menu (Resume, Settings, How to Play, Main Menu).

**Build defaults:** phone-first, touch, big tap targets, safe-area padding, works offline, no ads, no IAP, free forever.

**Guardrails**
- **Costs:** warn Kris before anything that could cost money (paid APIs, assets, hosting, store fees).
- **Secrets:** warn before anything that would put a password, private key or personal detail in this public repo.
- **Family:** in anything public-facing, never name his son or wife. Say only "my son" / "my wife".
- **Branding:** "Kralnu Games". Not chef-themed. Public contact is always hello@kralnugames.com.
- **README:** keep the existing style. Changelog entries go at the top of "What's new".

---

## Findings: what's missing or wrong today

### A. Gaps against the standard (must fix)

1. **Shotgun main menu.** `renderHome()` pushes ~15 buttons into one scrolling list (Learn to play, Challenge a rival, Daily, Friendly, Pass & play, Puzzles, Goals, Tournament, Gauntlet, Deck, Collection, Shop, Altar, Signature card, How to play, Backup, Review, Install, Get latest version).
2. **No Settings screen.** Only a "Sound on/off" pill (`[data-mute]`, `SAVE.sound`, `syncMute()`) in the top bar of home and game.
3. **No music.** Audio is seven synthesized SFX (`SFX.tap/place/flip/chain/coin/win/lose`) via `AudioContext`. No volume control.
4. **No Credits screen.** No credit line, no kralnugames.com, no hello@kralnugames.com anywhere.
5. **No Ko-fi / Support.**
6. **No in-game What's New.** The changelog only lives in the README.
7. **No pause menu during a match.** You can't get to How to Play, Settings or the main menu mid-game.
8. **Version is `16`, not semver, and not shown in a corner.** (It only appears in the Get Latest Version button subtext and the review panel.)
9. **Suspend save:** it listens for `visibilitychange` but not `pagehide`, so iOS can drop the last move. Add `pagehide` (and make sure saves fire on both).
10. **No `NOTES.md`** handover file (all of Kris's other games have one).
11. **Manifest:** `orientation: "any"`. The game has landscape CSS, so keeping `any` is fine; just confirm both orientations still look right after the menu rebuild.

### B. Finish and clean-up

12. **Dev mode in the live build.** Tapping the "O" in the title (`#devO`, `oTaps`) opens a password box checked against `DEV_HASH` (SHA-256, in the public repo). It sets `localStorage['ninefold-devmode']` and opens `#devPanel` (give every card, +Hearts, skip, peek, etc.). It's not a real security issue (single-player, client-side), but a release build shouldn't ship it. **Preferred fix:** gate it behind a URL flag that only works on localhost / a `?dev` param, **or** remove the tap-to-open entry point and password box while leaving the internal `DEV` code paths inert. Ask Kris which he prefers before deleting anything. Either way, the `DEV SAVE` stat label must never show for normal players.
13. **No finale.** Beating the last main rival only grants the "Champion of the Inn" achievement. Add a short **ending scene**: a few lines from Mellana / the innkeeper in the game's tone, then a **credits roll**. It plays once, the first time all eight main rivals are beaten, and can be replayed from Credits. It then points the player to Champion rematches and the Gauntlet. Kris will want to rewrite the ending text, so keep it in one clearly marked constant (e.g. `ENDING_LINES`) and say so in NOTES.md.
14. **README tone.** It currently says "living project" / "Coming next: more rivals, cards and modes". Change it to present the game as **complete**, with the option of free updates later.

### C. Balance (Session 3)

15. Do a final balance check on: rival difficulty curve across the eight main rivals and the chapter rivals, Gauntlet run length, coin income vs pack/Altar/signature costs, and Star Fragment / Heart pacing. **Measure first** with headless bot sims, then propose changes to Kris with numbers before applying them. Don't nerf anything just because it looks strong.

---

## Work plan

Do this in **three sessions**. Each session ends with a working, deployable game and a version bump.

### Session 1: menus and the standard shell → v1.0.0-rc1 (internal)

1. **Main menu rebuild** (tile grid, icon + label):
   - **Play** (big, primary; says "Learn to play" until the tutorial is done, then "Continue: next up X") → **Play panel**: Rival campaign · Daily challenge · Friendly · Pass & play · Puzzles · Tournament · Gauntlet. That's more than 6, so group it, e.g. *Story* (Rivals, Tournament, Gauntlet) / *Quick play* (Daily, Friendly, Pass & play, Puzzles). Locked modes keep their current lock text.
   - **Collection** → Deck · Collection · Shop · The Altar · Signature card
   - **Goals** (bounties + achievements, as now)
   - **Settings**
   - **More** → How to Play · What's New · Credits · Review · Support · Backup · Get Latest Version
   - **Install** tile only when installable (existing kit logic)
   - Coins top-left (as now); **`v1.0.0` small in a corner**
   - Keep every badge/hint that exists today ("New: Earth & Wind rivals are waiting", "New pack", bounty counts, etc.), just moved onto the right tile.
2. **Settings panel:** Music volume, SFX volume (sliders, 0–100, saved in `SAVE`), the existing hidden-hands option for Pass & play if it fits, **Reset save** (two-step confirm; also clears the IndexedDB backup so it doesn't auto-restore). Remove the "Sound on/off" pills or turn them into a quick mute that respects the sliders.
3. **Credits panel** (contents as in House rules above).
4. **Support** entry in More, invitation wording, opens ko-fi.com/kralnu90 in a new tab.
5. **What's New panel**, built from a `CHANGELOG` array in the code. Start with v1.0.0 plus short summaries of Updates 2.5–5.
6. **Pause menu** in matches (button in the game bar): Resume · Settings · How to Play · Main Menu. Main Menu mid-match must follow the existing forfeit/stakes rules. If leaving a rival match counts as a loss today, warn first, and never allow a free escape from a losing "for keeps" match.
7. **Suspend save** on `pagehide` as well as `visibilitychange`.
8. **Version** → `APP_VERSION='1.0.0'`, `version.json` `"1.0.0"`, `sw.js` `sg-v1.0.0`.
9. **Create `NOTES.md`** (format below).

### Session 2: music, finale, dev clean-up

1. **Music:** a gentle generated tavern/night-sky loop (Web Audio, no files, no paid assets), different moods for menu, match and finale, controlled by the Music slider. It starts only after the first user tap (autoplay rules) and pauses when the app is hidden.
2. **Finale + credits roll** (item 13).
3. **Dev mode** (item 12), per Kris's choice.
4. **README** → "complete" wording, add the v1.0.0 changelog entry, add a short Credits section.

### Session 3: balance, QA, release

1. **Bot sims** for item 15. Report the numbers, propose tweaks, apply only what Kris approves.
2. **Full QA pass** (checklist below) on a phone-sized viewport (390×844) and landscape (844×390).
3. **Release:** final version bump, cache bump, What's New entry, NOTES.md update. Tell Kris what to post on the hub (news item + move the card out of WIP in `kralnu90.github.io`). Don't push to the hub repo unless he asks.
4. **Play Store (only once Kris says it's 101% done):** wrap as a TWA / Android app the same way as To Remember (`to-remember/android/` + its GitHub Actions workflow is a working reference). **Cost heads-up:** a Google Play developer account is a one-off **$25** fee if he doesn't already have one. Store listing needs a privacy policy (the game stores data only on the device, with no accounts or tracking; a short page on kralnugames.com is enough), screenshots, a feature graphic and a content rating questionnaire.

---

## How to work on the code

- **Find code with anchors, not by reading the file.** `grep -n` / Python `str.find` around function names (`renderHome`, `checkUpdate`, `migrate`, `persist`, `SFX`, `syncMute`, `DEV_HASH`, `endButtons`). Make edits with Python replacements that **assert each anchor matches exactly once**.
- **Syntax check** after every change: extract the main `<script>` block(s) and run `node --check`.
- **Test headless** with Playwright on a local server (`python3 -m http.server`). Check that it loads with no console errors, the menu renders, and a full match plays to the end. Test with **a fresh save AND an existing save** (inject a realistic `ninefold-save` JSON from before the change) to prove migration keeps all progress.
- **Screenshot** the main menu, each panel and a match at 390×844 and 844×390, and look at them before calling a step done.
- Keep the visual style: night sky, velvet/brass, Cinzel headings, Alegreya Sans body, existing CSS variables (`--display`, `--body`, `--good`, `--bad`, etc.).
- Credit line in any new About/Credits text: "Ideas, design and testing by Kris. Built with Claude, an AI by Anthropic."

## QA checklist (all must pass before v1.0.0 ships)

- [ ] Fresh install: splash → menu → tutorial → first rival match, with no errors
- [ ] Old save from v16 loads with every card, coin, rival, achievement, signature card and Gauntlet/tournament state intact
- [ ] No screen has more than 6 buttons; every sub-panel has Back in the same place
- [ ] Music and SFX sliders work independently, persist after reload, and music stops when the app is hidden
- [ ] Reset save asks twice, really wipes (including the IndexedDB backup) and the game restarts clean
- [ ] Pause menu works in every mode (rival, daily, friendly, pass & play, puzzle, tournament, gauntlet) and respects stakes
- [ ] Finale plays once on beating all eight main rivals, is replayable from Credits, and never triggers early
- [ ] Credits show the exact credit line, contact email, site, font licences and copyright
- [ ] Support wording is an invitation; no pop-ups ask for money
- [ ] Version shows `v1.0.0` in the corner; `version.json` and `sw.js` match; the update banner appears when `version.json` is newer
- [ ] Works offline after first load (airplane-mode test in Playwright: `context.setOffline(true)` + reload)
- [ ] Install button shows on Android/desktop when installable and hides once installed; iPhone shows the Share → Add to Home Screen tip
- [ ] Dev panel can't be reached by a normal player
- [ ] Portrait and landscape both look right, safe-area padding is respected, tap targets ≥ 44px
- [ ] No passwords, private keys or personal details added to the repo; no family names anywhere public

## Delivery

- Kris often updates GitHub from his phone. **Ask him at the start** whether you should commit and push directly or hand him unzipped `index.html`, `README.md`, `sw.js`, `version.json` and `NOTES.md` to upload. If he has edited the README on GitHub, pull first and don't overwrite his changes.
- Keep him posted with short updates when a session's work is ready to try, plus the live link.

## NOTES.md format (create it in Session 1, update every session)

```
# Solaris Gambit · Handover notes
Last updated: <date> · game version <x.y.z>

## The project      (what it is, live URL, repo, files, save keys and why they say "ninefold")
## Kris's house rules (short version of the Kralnu Standard + delivery preference)
## How to work on it (anchors, node --check, Playwright, test save injection)
## Systems map      (rivals, chapters, gates, economy, Altar, signature card, Gauntlet, tournament, achievements: where each lives in the code)
## Release checklist
## Open questions / ideas for later
```
