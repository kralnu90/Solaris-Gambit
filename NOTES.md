# Solaris Gambit · Handover notes
Last updated: 23 Nov 2026 · game version 1.0.0-rc2

## The project
- A nine-square card duel against tavern rivals, by Kralnu Games. Content-complete: eight main rivals, four element chapters (Fire & Ice, Shadow & Holy, Water & Thunder, Earth & Wind), Gauntlet, weekly tournaments, puzzles, daily challenge, the Altar, the signature card and 29 achievements.
- Live: https://kralnugames.com/Solaris-Gambit/ (GitHub Pages from `kralnu90/Solaris-Gambit`, custom domain through the hub repo `kralnu90.github.io`).
- Files: `index.html` (the whole game: plain JS, DOM and canvas, no build step, very long minified-style lines), `sw.js` (offline cache), `manifest.webmanifest`, `version.json` (update check), `icons/`, `README.md`, `LICENSE`, `SOLARIS-RELEASE-PLAN.md` (the v1.0.0 brief), this file.
- Save: `localStorage['ninefold-save']` plus an IndexedDB copy (`solaris-gambit` db, `kv` store, keys `save` and `at`) written by `safeBackup()`. On load, `readBackup()` puts the copy back if it's clearly further along than localStorage. The old dev save key `ninefold-dev` is no longer read (dev mode was removed in 1.0.0-rc2).
- **Why "ninefold":** the game's working title was Ninefold. The keys keep that name on purpose. **Never rename them**, or every player loses their progress. New save fields go through `migrate()` with a safe default.

## Kris's house rules (short version)
- The Kralnu Standard: tile menus with icon + label, at most 4–6 buttons a screen, Play is the biggest button, Back always top-left, Install and update prompts only when useful, version small in a corner.
- Main menu: Play · Collection · Goals · Settings · More (+ Install when installable).
- Settings: separate music and SFX volume, reset with confirmation, remembered between visits. Pause menu in matches: Resume, Settings, How to Play, Main Menu.
- Credits line: "Ideas, design and testing by Kris. Built with Claude, an AI by Anthropic." Contact is always hello@kralnugames.com. Support wording is an invitation to Ko-fi, never a request.
- Free forever, no ads, no IAP, works offline, phone first.
- Guardrails: warn before anything that costs money or puts secrets or personal details in this public repo. Never name Kris's son or wife in anything public.
- Don't change balance, rules or content without showing Kris the numbers first.
- **Delivery:** commit on the work branch, then wait. When Kris says **"push"**, that means **go live**: merge into `main` and push it, so GitHub Pages deploys to everyone. Never push before he says so. Check `origin/main` first, in case he edited files (especially the README) on GitHub.

## How to work on it
- Don't read `index.html` whole. Find code with `grep -o ".\{0,200\}ANCHOR.\{0,300\}" index.html` and edit with a Python script that asserts each anchor matches exactly once before replacing.
- **Don't put `//` comments in code you inject onto one line.** They comment out everything after them. Use `/* */`.
- Syntax check: pull out every inline `<script>` block and run `node --check` on each (and on `sw.js`).
- Test headless with Playwright against `python3 -m http.server`. Run each test twice: once with a fresh save and once with an old save injected into `localStorage['ninefold-save']` before load. Compare every field of the old save afterwards; nothing should change except new fields. (An inconsistent hand-made save can earn catch-up achievements on load. v16 does this too, so let a test save settle once before using it as a fixture.)
- Play a whole match in the test (pick the first playable card, first open square, and handle the coin toss, rock-paper-scissors and pass screens). Screenshot at 390×844 and 844×390 and look at them.
- Releasing: change `APP_VERSION` in `index.html`, `version.json` and `VERSION` in `sw.js` together, and add an entry at the top of `CHANGELOG` in `index.html` (it feeds the in-game What's New).

## Systems map (anchors to grep for in index.html)
- **Menus (v1.0 shell):** `ICONS`, `icon()`, `tile()`, `homeInfo()`, `renderHome`, `renderPlay`, `renderQuick`, `renderCards`, `renderMore`, `renderSettings`, `renderCredits`, `renderNews`, `CHANGELOG`. Screens are listed in `SCREENS` and opened with `show(id)`. Tiles use `data-go="<screen>"`. The CSS is in the `v1.0 shell` style block.
- **Back buttons:** every screen's Back is the pill top-left of its `.bar`. Guide and Settings remember where they were opened from in `dataset.back` (including `'game'` from the pause menu).
- **Pause menu:** `openPause`, `closePause`, `leaveMatch`, `leaveQuiet`, `pauseNote`. The stakes are the same as v16: Gauntlet adds a loss, Tournament knocks you out, leaving a rival match costs nothing (to revisit), and puzzles and finished matches leave on one tap. Everything else asks you to tap twice.
- **Settings and sound:** `SAVE.sound` (sound effects on/off, the Sound pill `[data-mute]` in Settings and in matches, `syncMute()`), `SAVE.sfxVol` (0–100), `sfxBus()` (master gain for all effects), `SFX`, `tone()`, `noise()`, `ac()`.
- **Music:** `MOODS` (night = menus, tavern = matches, finale), `Music` (scheduler: `refresh`, `set(mood)`, `start`, `stop`, `beat`, synth voices `pad`/`pluck`/`bell`/`hit`). `SAVE.music` (on/off, the Music pill `[data-music]` in Settings, `syncMusic()`) and `SAVE.musicVol` (0–100, default 50). Music starts on the first tap, stops when the app is hidden, and `show()` picks the mood. Old saves with sound off start with music off too.
- **Finale:** `ENDING_LINES` (**Kris: rewrite the ending text here**; `who:''` lines are narration), `maybeFinale` (runs after every screen change except the game: all eight main rivals beaten, `SAVE.finaleSeen` false, no panel open), `playFinale`, `finLine`, `finRoll` (credits roll), `finEnd` (end card: Champion rematches / Gauntlet / Menu), `finClose`. Replay button `#replayEnd` in Credits once seen.
- **Saving:** `store`, `persist()` (skipped while `WIPING`), suspend save on `pagehide` and `visibilitychange` (when hidden), `safeBackup`, `readBackup`, `progressOf`, `migrate`, `newSave`.
- **Reset:** `resetStep()`, `wipeSave()`, `wipeBackup()`. It overwrites the IndexedDB copy with a fresh save so it can't restore itself, keeps sound settings, then reloads.
- **Rivals and chapters:** `RIVALS` (chapter rivals have a `chapter` field), `CHAPTERS`, `openChapters`, `gate` (unlock rules: `gate.tourney`, `gate.gauntlet`, `gate.champ`, chapter gates), `unlocked(i)`, `startRival`, champion rematches via `data-champ`.
- **Cards:** `DEFS`, `def(k)`, `BASE_KEYS`, `gain()`, shiny cards in `SAVE.shiny`.
- **Economy:** `PACKS` and the shop (`renderShop`, `data-buy`), coins from rivals (`pay` on each rival), daily (25), friendlies, bounties (`BOUNTY_POOL`, `BOUNTY_PAY`, `BOUNTY_BONUS`, `ensureBounty`).
- **Altar and signature card:** `renderAltar` (scrap, trade up, sacrifice for Star Fragments, fuse four into a Heart), `renderForge`, `SAVE.sig`, `makeSigDef`, `SIG_MAX`, `lvlOfSum`.
- **Gauntlet:** `G_FEE`, `G_PAY`, `gRoll`, `gauntletMatch`, `gEnd`, `SAVE.gauntlet`.
- **Tournament:** `weekRules`, `tourneyMatch`, `renderTourney`, `SAVE.tourney`.
- **Puzzles and daily:** `PUZZLES`, `renderPuzzles`, `startDaily`, `SAVE.daily`.
- **Achievements:** `ACH`, `unlockAch`, `checkState` (catches up achievements on load).
- **Match engine:** `setupMatch`, `play`, `aiMove`, `finish`, `endButtons`, `M` (current match).
- **Updates and install:** `checkUpdate()` (compares `version.json` with `APP_VERSION` using `!==`, shows `#updateBar`), `window.KralnuUpdate` (Get Latest Version, `data-get-latest`), the Kralnu install kit (`data-install`, shown only when the `kg-can-install` class is on `<html>`).
- **Dev mode:** removed in 1.0.0-rc2 (no password box, panel or DEV checks). The game clears any old `ninefold-devmode` flag on load. To test late-game content, inject a save in Playwright instead.

## Release checklist
1. Bump `APP_VERSION` (index.html), `version.json` and `sw.js` `VERSION` together.
2. Add a What's New entry at the top of `CHANGELOG` (and in the README's "What's new" from v1.0.0 on).
3. Run `node --check` and the Playwright tests (fresh and old save, both orientations, a full match, no console errors).
4. Update this file.
5. Post a news item on the hub and refresh the game's card (Kris does this, or ask before touching `kralnu90.github.io`).
6. Play Store only when the game is 101% done (Google Play account is a one-off $25 if Kris doesn't have one).

## Session log
- **Session 1 (1.0.0-rc1):**
  - New tile main menu with Play, Collection, Goals, Settings, More and a conditional Install.
  - Play panel (Rival campaign, Daily, Quick play, Tournament, Gauntlet) with a Quick play sub-panel (Friendly, Pass & play, Puzzles).
  - Collection panel and More panel (How to play, What's new, Credits, Review, Support, Get latest version).
  - Settings with SFX on/off and volume, house rules, save backup and a two-step reset.
  - Credits, What's New from `CHANGELOG`, Support link to Ko-fi.
  - Pause menu replaces the Leave button.
  - Suspend save on `pagehide`.
  - Version shown in the bottom corner of the home screen.
- **Session 2 (1.0.0-rc2):**
  - Generated music with three moods, and separate Sound and Music on/off pills, each with its own volume slider, in Settings.
  - Finale scene (placeholder text in `ENDING_LINES`), credits roll and end card. Plays once, including for players who had already beaten all eight. Replay from Credits.
  - Dev mode removed completely.
  - README now presents the game as complete, with a v1.0.0 entry and a Credits section.
  - Levels at the defaults (music 50%, effects 100%): music peaks around 0.05, effects 0.12–0.28, so music sits underneath.
- **Session 3 (planned):** balance sims and proposals, full QA, 1.0.0 release.

## Open questions / ideas for later
- The ending text is a placeholder for Kris to rewrite (`ENDING_LINES`).
- Leaving a rival match halfway is still free, as in v16, even in a losing "for keeps" match. Reloading the page mid-match also gets you out. Revisit in Session 3 with numbers (Kris chose to keep it for now).
- Before the tutorial is finished, the big Play tile starts the tutorial, so the Play panel isn't reachable until it's done. A brand-new player always starts with the tutorial; restored saves are unaffected.
- The coin toss and rock-paper-scissors panels cover the Pause button for the few seconds they're up (Leave was covered the same way in v16).
