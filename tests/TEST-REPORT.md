# PULSE SURVIVOR — Test Report (QA Evidence)

- **Test date:** 2026-06-04
- **Build:** `index.html` (single-file, zero-build)
- **Method:** Real-logic playthroughs driven through the actual game code (movement, auto-fire, spawning, collisions, shard pickup, XP/level-up, OVERDRIVE, boss, scoring, game-over). Screenshots captured natively (HTML + canvas) via local headless Chrome against `http://127.0.0.1:8800`.
- **Viewports:** mobile 390×844 (primary — layout strategy A) · desktop 1280×800 (letterboxed frame)
- **Console errors:** 0 (no `console.error` / `pageerror` across all states)

## Results

| ID | Area | Test | Viewport | Evidence | Result | Notes |
|----|------|------|----------|----------|--------|-------|
| T01 | UI | Main menu renders (title, pills, Play, How-to, Best) | mobile | [01-menu-m.jpg](screenshots/01-menu-m.jpg) | ✅ PASS | Orbitron fonts + gradient title |
| T02 | UI | Difficulty selection (Hard pill highlights) | mobile | [02-menu-hard-m.jpg](screenshots/02-menu-hard-m.jpg) | ✅ PASS | pill toggles accent |
| T03 | Onboarding | First-run "How to Play" overlay | mobile | [03-howto-m.jpg](screenshots/03-howto-m.jpg) | ✅ PASS | shows once, flag saved |
| T04 | Core loop | Early combat: move, auto-fire, enemies, shards, dmg numbers, HUD bars | mobile | [04-combat-early-m.jpg](screenshots/04-combat-early-m.jpg) | ✅ PASS | "SURVIVE" banner, HP/XP/OD bars |
| T05 | Progression | Level-up overlay — 3 random upgrades | mobile | [05-levelup-m.jpg](screenshots/05-levelup-m.jpg) | ✅ PASS | Move Speed / Fire Rate / Max HP cards |
| T06 | Signature hook | OVERDRIVE burst active (aura, screen pop, ACTIVE timer) | mobile | [06-overdrive-m.jpg](screenshots/06-overdrive-m.jpg) | ✅ PASS | "ACTIVE 4.7s", invincibility + rapid fire |
| T07 | Difficulty ramp | Mid-run swarm + UNLEASH button + multiple enemy types | mobile | [07-midrun-swarm-m.jpg](screenshots/07-midrun-swarm-m.jpg) | ✅ PASS | wave ramp escalates spawns |
| T08 | Enemy variety | BOSS (~90s) + brutes (triangles) + minions (circles) | mobile | [08-boss-m.jpg](screenshots/08-boss-m.jpg) | ✅ PASS | "BOSS INCOMING", spinning boss, HP bar |
| T09 | UI / Scoring | Game-over (hero score, best, survived, level, kills) | mobile | [09-gameover-m.jpg](screenshots/09-gameover-m.jpg) | ✅ PASS | score floored (5,829), all stats correct |
| T10 | Responsive | Desktop menu — centered mobile-frame + letterbox | desktop | [10-menu-desktop.jpg](screenshots/10-menu-desktop.jpg) | ✅ PASS | layout strategy A confirmed |
| T11 | Responsive | Desktop combat — game runs in letterboxed frame | desktop | [11-combat-desktop.jpg](screenshots/11-combat-desktop.jpg) | ✅ PASS | legible HUD bars |
| T12 | UI | Pause dialog (settings toggles) | mobile | [12-pause-m.jpg](screenshots/12-pause-m.jpg) | ✅ PASS | clean, no HUD bleed |

## UI / UX layout + color review pass

Every screenshot was re-audited for *visual* breakage (not just "does the feature render"). Issues found and fixed:

| Issue | Where | Fix |
|---|---|---|
| Upgrade card name + description rendered **inline / run-together** ("Fire Rate +22%Shoot faster") | level-up cards | `.nm`/`.ds` → `display:block`, `.meta` flex-column with gap |
| **HUD bled through overlays** — score/timer chips, wave banner, and bottom HP/XP/OD bars showed (clipped) behind the menu, level-up, pause & game-over panels | all overlay screens | hide `#hud` via `#app.overlay-open` toggled in `show()`/`hideAll()` |
| Bottom bars too thin, labels (8px) hard to read | in-game HUD | taller bars (15/17px), 9px labels, stronger shadow |
| Damage-number popups overlapped into unreadable clusters | combat | smaller hit numbers (11px) + horizontal jitter; kill numbers 14px |

**Color/comfort:** violet `#c77dff` + cyan/pink accents on near-black read clearly; panels use consistent glass treatment; HP=warm, XP=cyan, OVERDRIVE=violet are distinct and intuitive. Verdict: comfortable, on-theme, no harsh contrast.

## Telemetry from real-logic playthroughs (sampled, Normal difficulty)

| t (s) | live enemies | kills | level | HP | OD |
|---|---|---|---|---|---|
| 5  | 3  | 3  | 1 | 100 | 14 |
| 10 | 5  | 6  | 1 | 100 | 28 |
| 15 | 6  | 10 | 2 | 100 | 56 |
| 20 | 10 | 12 | 2 | 100 | 65 |

A full bot run reached **game-over at ~50–56s, score ~5.8–9.1k, level 3, ~46–56 kills**, with boss spawn at 90s verified separately. Spawn rate ≈ 1/s (matches the 0.95s Normal interval); XP, OVERDRIVE charge, and level-ups all flow correctly.

## Bugs found & fixed during QA
1. **Contact damage stacking (no i-frames)** — overlapping enemies drained HP every frame (death in ~24s). Fixed: discrete hit + 0.7s i-frames. (`cedd520`/`f2d1d2a`)
2. **Dead enemies never removed** — corpses persisted: rendered, chased, dealt damage, and re-triggered kills (inflating kills/score/shards). Fixed: filter dead enemies each frame + guard double-kills. (`cedd520`)
3. **Game-over score shown as float** (`9,113.167`). Fixed: floor score in `gameOver()`.

## Summary
- **Test cases:** 12 · **PASS:** 12 · **FAIL:** 0
- **Coverage:** menu / difficulty / onboarding / HUD / core loop / shard pickup / level-up upgrades / OVERDRIVE burst / wave ramp / enemy variety (minion + ranger + brute + boss) / scoring / pause / game-over / localStorage best / mobile + desktop + **UI/UX layout review**.
- **Console errors:** 0
- **DOCS.md matches code:** ✅ (CONFIG numbers, difficulty table, scoring, OVERDRIVE all synced)

## ✅ VERDICT: **PASS — GAME DONE**
