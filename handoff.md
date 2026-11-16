# Beat Grid (กล่องจังหวะ) · handoff

Built from `specs/new/21-beat-grid.md` with `specs/new/20-new-apps-shared.md` and `specs/v2/10-v2-shared-upgrades.md`
sections 1 to 4. One file, `index.html`, assembled by `node tools/build-new-apps.mjs beat-grid` from the parts in
`tools/new-apps/` (the parts are development-only and are not shipped).

## What was built

- Three modes: `free` (build your own, tempo, 4/4 and 3/4, 15 presets with their `about`, counting strip, saved pattern),
  `copy` (listen then rebuild, needs sound) and `read` (read the counting, then rebuild). Challenge games are 4 rounds.
- Home, play, result, how-to and badges screens; settings with theme, reduced motion, volume, clear data, self-check.
- One CSS grid with one DOM. Narrow screens run time downward with a sticky track header; at 720 px and up time runs to
  the right (only CSS custom properties `--s`, `--t` and media-query placement change).
- Drum synthesis in zone D: `kick` (gliding sine), `noise` (one generated 1 s noise buffer, high-pass or band-pass, plus an
  optional triangle body) and `tone`. Attack 2 ms (the shared spec allows 1–3 ms for drums), every hit ends in a ramp to 0.
- Playback loop on `requestAnimationFrame` only while playing, `performance.now()` clock, `lookaheadMs` scheduling,
  late steps beyond `maxLateMs` dropped, stops when the tab is hidden.
- Self-check with 15 tests (the spec lists 14; one was added for saved patterns), independent of the editable CONFIG and
  zone A tables (it uses fixed spec values and fixture grooves).
- Five slots, ten events (plus `screenChange`, `scoreChange`, `badgeEarned`), all helpers named in the spec.

## Deviations and decisions

1. **Added Thai strings** (all in `TEXT`, zone A): `resultCopy` (heading of the result screen for both challenge modes; the
   spec gives none), `statStars`, `grooveBeatsLine`, `grooveSkipped`, `grooveUnknownTrack`, `grooveBadLength`,
   `grooveBadChar`, `grooveBadBeats`, `grooveNoHit`, `cntTracks`, `cntGrooves`, `cntModes`, `cntBadges`, `cntScreens`,
   `metre34`, `metre44`, `badgeHad`, `badgeNot`, `badgeAllDone`, `extensionLine`, `backHome`, `storageOk`, `storageBlocked`.
2. **`Core.pickRounds`** draws a random subset of the level's grooves and then sorts it by groove level, so `medium` and
   `hard` games mix levels and still ramp up. (Taking whole levels in order would never reach level 2 on `medium`.)
3. **`statChecks`** counts every check pressed: failed checks plus the successful one; a round that was revealed counts only
   its failed checks.
4. **`stats.freePlays`** goes up on the first play after the player has edited the grid (not after loading an unchanged
   preset) and only when at least one cell is filled; that play also evaluates the `composer` badge.
5. **Presets overwrite the saved pattern.** Choosing a preset (or the retry button on the result screen) replaces the saved
   pattern; there is no undo. This is a likely surprise for students.
6. **Placing a hit plays its sound** (a short preview) when audio is available and nothing is playing.
7. **After a solved round, or a reveal, the groove plays once** when audio exists; the grid is locked until `ต่อไป`.
8. **`copy` target playback** moves the playhead but does not flash the target's hits (the `stepTick` event still lists `hits`).
9. **`loopEnd`** fires when playback moves from the last step to step 0 and once more when a finite playback ends.
10. **Tempo changes while playing** re-anchor the timeline so the playhead does not jump.
11. **Selection of `CONFIG.seed`** (0 = random) was added for tests and for repeatable exam rounds.
12. **Wide screens:** page width rises to 960 px at 720 px and up so 16 steps fit with cells of at least 44 px.
13. **Noise buffer** is filled with `Math.random()` (not the seeded rng) when the audio context is created.

## Tests run in this session (from `final-exam/`)

| Command | Result |
|---|---|
| `cd tests && node --test beat-grid-core.test.cjs beat-grid-ui.test.cjs` | 35 tests, 35 passed (8 core, 27 UI) |
| `cd tests && node --test new-apps-packaging.test.cjs` | 21 tests, 21 passed (7 for this app) |
| `node tools/beat-grid-browser.mjs` | Chrome: every mode with and without audio, keyboard-only flows, pacing measured over two bars at 96 BPM, cell sizes at 320 px, orientation at 1280 px, 72 screenshots |
| `node tools/new-apps-contrast.mjs` | passed: text contrast in light and dark |
| `Game.runSelfCheck()` in jsdom and in headless Chrome | 15 of 15 |

The overall record, including the nine stale older tests that make `npm test` red, is in `REPORT.md`.

## What was not verified

- Real sound on speakers: drum timbres (kick, snare, hat, wood) were checked against a fake `AudioContext` and by reading the
  synthesis code, not by ear. They are the least certain part and need a listen.
- Touch feel on a real phone (clicks and keyboard only).
- The school lab Windows PCs.
- Deployment to GitHub Pages or any host.
- Musical content: the 15 groove patterns, the counting and the `about` sentences (reggae, funk, bossa nova in particular)
  come from the spec; they were checked for row lengths, characters and counting text only. A musician must review them.
- Sticky header and sticky transport bar behaviour on iOS Safari.
- Whether a free AI model can build feature-menu item 1 (random beat) from the README in the exam window.
