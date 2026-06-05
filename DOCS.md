# PULSE SURVIVOR — Tài liệu kỹ thuật

> Đọc file này là hiểu toàn bộ game + biết chỗ nào để sửa, **không cần đọc code**.
> Game nằm trong: `index.html`. Số cân bằng tập trung ở **§14**.

---

## 0. TRẠNG THÁI TÍNH NĂNG (đọc đầu tiên)

| Tính năng | Trạng thái | Ở đâu (hàm / marker / file) |
|---|---|---|
| Core loop (survive waves, collect shards) | ✅ | `tickGame()` |
| Player movement (WASD / touch drag) | ✅ | `handleInput()` |
| Auto-fire weapon system | ✅ | `tickWeapon()` |
| Enemy spawning & AI | ✅ | `spawnEnemy()` |
| Shard pickup + XP bar | ✅ | `collectShard()` |
| Level-up + upgrade selection | ✅ | `levelUp()` |
| OVERDRIVE meter & burst | ✅ | `triggerOverdrive()` |
| Boss spawning (~90s) | ✅ | `spawnBoss()` |
| Difficulty modes (Easy/Normal/Hard) | ✅ | `DIFF` config |
| Score + best record | ✅ | `SaveManager` |
| Juice (particles, screen shake, numbers) | ✅ | `Particle`, `shake()` |
| Audio (synthwave, SFX mute) | ✅ | `AudioManager` |
| HUD + pause/gameover | ✅ | `updateHUD()` |

**Chưa làm (backlog):** achievements, daily rewards (scope MVP)

---

## 1. Tổng quan & Concept

### Market context (research)
**Genre hot:** Survivors-like (bullet-heaven) exploded 2024–2025 with games like Brotato, Survivors.io, Vampire Survivors. Top-down auto-shooters with progression mechanics perform well on casual platforms.

**Theme/style hot:** Neon/synthwave aesthetic dominates casual web games — high contrast, glowing elements, pulsing animations draw mobile players.

**Story/vibe:** "Survive relentless neon attacks, grow stronger with each wave, master the OVERDRIVE burst mechanic to dominate."

**Đối thủ:** Generic survivors lack a signature identity. **Hook:** PULSE SURVIVOR's OVERDRIVE meter (charge-then-unleash) creates a distinct rhythm vs. constant autofire, making the game feel more tactical.

### Concept
- **Thể loại:** Top-down survivors-like / bullet-heaven, single-player, endless waves.
- **Core loop (1 câu):** "Player moves to dodge & position, weapon auto-fires enemies; collect glowing shards to fill XP & OVERDRIVE meters; level-up to pick upgrades; manage OVERDRIVE bursts for heavy-hitter moments."
- **Thắng / Thua:** No win condition — survive as long as possible. Die when HP ≤0.
- **Cảm giác mục tiêu (fantasy):** Feel like a neon warrior getting stronger & flashier each level, unleashing devastating overdrive bursts.
- **2D hay 3D + vì sao:** **2D Canvas** — fast, mobile-friendly, perfect for top-down action with many entities. No need for 3D depth.
- **Layout (mobile ưu tiên #1):** **A — Mobile-frame** (khung portrait ≤480px, letterbox desktop). Survivors-like don't need wide desktop view; portrait suits the action-arcade vibe.

## 2. Tech stack
- **Render:** Canvas 2D (vanilla JS, no framework).
- **Build:** Zero-build, single `index.html`.
- **External:** Google Fonts (Orbitron + Space Grotesk).
- **Physics:** Collision detection only (no physics engine — entity-based simple AABB).
- **Audio:** WebAudio synth inline; mute toggleable.
- **Storage:** `localStorage` for best score, difficulty, settings.

## 3. Vòng đời / State machine

States: `menu → playing → (paused) → dying → gameover → menu`

- **menu:** Difficulty selection (Easy/Normal/Hard), display best score, "How to Play" first-run.
- **playing:** Active gameplay — player moves, enemies spawn, XP bar fills, level-up overlay appears.
- **paused:** Pause menu with settings toggles (music/sfx/haptic).
- **gameover:** Show final score, best record, stats (level, kills, time), retry/menu buttons.

## 4. Gameplay & quy tắc

- **Sân chơi:** Canvas 480px (w) × full height portrait (h). Player spawns center. Enemies spawn around edges. Shards drop at kill location.
- **Di chuyển:** Player moves with WASD (desktop) or drag-to-aim (mobile). Movement speed upgradeable. Max speed capped to prevent cheese.
- **Chết:** HP ≤0 → game over. Enemies deal dmg on hit.
- **Tương tác cốt lõi:**
  - **Auto-fire:** Weapon fires projectiles in direction of nearest enemy (or last aim dir). Fire rate upgradeable. Can have multiple projectiles (upgrade path).
  - **Shards:** Drop from enemy kills (20–40% chance). Collect near player (magnet radius upgradeable). Each shard = +1 XP. XP bar fills; on full → level-up trigger.
  - **Level-up:** Show 3 random upgrades from pool (damage, projectiles, fire-rate, speed, max-HP, magnet, invincibility duration). Pick 1.
  - **OVERDRIVE:** Shards also charge OVERDRIVE meter (visual meter bar). On full → can trigger (mouse click / tap button) → 5s burst (fire-rate 3x, brief invincibility, screen pop animation, visual bloom).
  - **Boss:** Spawns ~90s into run. Tougher enemy, higher HP, bigger drops. 1 boss per run (no respawn, but waves continue).

## 5. CẤU TRÚC MÀN CHƠI / LEVEL

### 5.1 Mô hình level
- **Endless + ramp:** 1 màn vô tận, khó tăng theo thời gian chơi & cấp độ (level).
- **Không có "level" riêng rẽ** — một run liên tục, enemy config evolves.

### 5.2 Level được định nghĩa ở đâu
- Dữ liệu config: `DIFF` object (easy/normal/hard) + `WAVE_CONFIG` table defining spawn rules per wave.
- `CONFIG` object holds all balance numbers (health, damage, fire-rate, etc.) — single source of truth.

### 5.3 Wave progression
- **Wave 0–2 (Easy spawn):** Spawn rate slow, enemy HP low.
- **Wave 3–5 (Normal):** Faster spawn, tougher enemies.
- **Wave 6+ (Hard):** Dense spawns, high-HP waves, projectile enemies introduced.
- **Boss wave:** ~90s elapsed → boss appears, spawning continues.

---

## 6. CẤU TRÚC ĐỘ KHÓ

| Mode | Spawn interval | Enemy HP mult | Enemy speed mult | Damage mult | OD/XP mult |
|---|---|---|---|---|---|
| Easy | 1.25s | 0.8× | 0.85× | 0.7× | ×1.0 |
| Normal | 0.95s | 1.0× | 1.0× | 1.0× | ×1.0 |
| Hard | 0.68s | 1.45× | 1.18× | 1.35× | ×1.15 |

> Contact damage is a **discrete hit** (full `enemy.dmg × dmgMult`) that triggers **0.7s i-frames**, so standing in a crowd costs ~one hit per 0.7s — not per-frame stacking.

**Ramp formula:** Every 30 seconds of gameplay, difficulty increases:
- Spawn interval: `interval * 0.92` (faster spawning).
- Enemy HP: `HP * 1.05` (tougher).
- Player gets proportional rewards (kills still worth same XP; higher waves = higher base dmg gained).

---

## 7. HỆ THỐNG TÍNH ĐIỂM

- **Time alive:** `floor(timeSeconds * 10)` base points.
- **Kill combo:** Each consecutive kill within 3s multiplies next kill by 1.2× (max 2.5× at 10 kills).
- **Wave bonus:** Every wave cleared (no enemies for 2s) → `currentWave * 50` bonus.
- **OVERDRIVE bonus:** +500 points per enemy killed during overdrive.
- **Best score saved** → `localStorage` key `pulse-survivor.best`.

---

## 8. Upgrade Pool (Level-up选项)

Players pick 1 of 3 randomly-chosen upgrades:

| Upgrade | Effect | Stacking |
|---|---|---|
| Damage +30% | Base damage increases | ×5 max (compound) |
| Extra Projectile | Adds 1 bullet per shot | ×3 max (2 bullets → 3 → 4) |
| Fire Rate +25% | Faster weapon fire | ×4 max |
| Pickup Magnet +100px | Shard collection radius | ×4 max |
| Move Speed +20% | Player speed | ×4 max |
| Max HP +20 | Increase HP pool | ×5 max |
| Invincibility Time +1s | OVERDRIVE invincibility | ×3 max |

---

## 9. OVERDRIVE Mechanic (Signature Hook)

- **Meter:** Separate bar (distinct visual, e.g., glowing purple line). Shards += 15 OVERDRIVE meter points (on top of XP).
- **Trigger:** Manually click/tap or press spacebar when full.
- **Burst (5s):**
  - Fire-rate × 3 (stacked with upgrades).
  - Invincibility (brief — 0.3s per hit, max 5 hits invuln).
  - Screen pop (scale 1.0 → 1.08 quickly, fade back; bright screen flash glow).
  - Visual: large glowing aura around player, projectiles glow brighter.
  - Audio: ascending synth arpeggio on trigger, pulsing synth loop during.
- **Cooldown:** On end → meter depletes, starts refilling. No cooldown; can spam if skilled at collecting shards.
- **Purpose:** Creates rhythm: collect → build meter → wait for dangerous moment → unleash. Tactical vs. passive autoshooter.

---

## 10. Enemy Types

### Minion (基礎)
- HP: `CONFIG.minion.hp` (varies by wave/difficulty)
- Speed: `CONFIG.minion.speed` (walks toward player)
- Damage: `CONFIG.minion.dmg` (on hit)
- Drop: 20% shard chance

### Ranger (Range attacker — wave 3+)
- HP: `CONFIG.ranger.hp` (30% more than minion)
- Fires projectile at player every 2s
- Drop: 40% shard chance

### Boss (Spawn ~90s)
- HP: `CONFIG.boss.hp` (base ×6 minion HP)
- Huge size, slower but dangerous
- Fires spray of projectiles
- Drop: Guaranteed shard ×5, +1000 bonus points

---

## 11. Particle / Juice

- **Hit:** Small burst of 4–6 particles at enemy hit location, white/blue glow, ~300ms lifespan.
- **Shard pickup:** Brief scale-up pop animation (1.0 → 1.3), fade out.
- **Level-up:** Colorful sparkles around player, fanfare sound.
- **Screen shake:** 0.2s shake on player hit (10px amplitude); longer (0.4s, 20px) on overdrive trigger.
- **Damage numbers:** Pop floating text ("+10", "+25") near hit, rise & fade over 0.8s. Combo numbers highlighted in accent color.

---

## 12. Audio Design

- **BGM:** Synthwave synth loop (looped via WebAudio). Tempo ~120 BPM, minor key, ominous pulse.
  - Trigger variation on level-up (higher octave, brighter).
  - Trigger variation on overdrive (massive bass drop, intense arpeggio).
- **SFX:**
  - Hit enemy: retro blip (freq ~800Hz, 50ms).
  - Collect shard: ascending chirp (freq ramp 400→800Hz, 100ms).
  - Level-up: fanfare arpeggio (4-note sequence).
  - Overdrive trigger: synth sting (freq drop, 200ms).
  - Game over: sad descending tone.
- **Mute:** Toggle in pause menu (music/sfx separate toggles).

---

## 13. Visuals & Design System

- **Colors:**
  - Accent: `#c77dff` (violet)
  - Background: `#06060d` (near-black)
  - Ink: `#eef0ff` (off-white)
  - Enemy: gradient hues (orange/pink/cyan cycles)
  - Player: bright accent glow
- **Typography:** Orbitron (display, big numbers), Space Grotesk (body).
- **Glass panels:** Backdrop blur, semi-transparent (arcade aesthetic).
- **Glow effects:** Box-shadow + blur on key elements (bars, buttons, enemies).

---

## 14. CONFIG (Single Source of Truth — ALL balance numbers here)

```javascript
const CONFIG = {
  // Player — contact damage is a DISCRETE hit gated by i-frames (no per-frame stacking)
  player: { maxHp:100, speed:200, radius:14, iframesSec:0.7 },

  // Weapon
  weapon: { baseDamage:8, fireRate:0.32, projectileSpeed:420, projRadius:5, projLife:1.1 },

  // Shards — each = +1 XP and +7 overdrive; magnet radius 64px (upgradeable)
  shard: { xpPerShard:1, overdrivePerShard:7, basePickup:64, magnetSpeed:540, life:14 },

  // OVERDRIVE
  overdrive: { maxCharge:100, durationSec:5, fireRateMult:3, dmgMult:1.4 },

  // Enemies (base; scaled by difficulty mult + wave ramp)
  minion: { hp:26, speed:78, dmg:9,  radius:13, dropRate:0.6, score:1 },
  ranger: { hp:34, speed:52, dmg:7,  radius:13, fireRate:2.2, dropRate:0.7, score:2, introTime:25 },
  brute:  { hp:120, speed:46, dmg:18, radius:22, dropRate:1,  score:5, introTime:55 },
  boss:   { hp:1400, speed:34, dmg:26, radius:46, spawnTime:90, sprayEvery:2.4 },

  // Difficulty
  difficulties: {
    easy:   { spawnInterval:1.25, enemyHpMult:0.8,  speedMult:0.85, dmgMult:0.7,  xpMult:1.0 },
    normal: { spawnInterval:0.95, enemyHpMult:1.0,  speedMult:1.0,  dmgMult:1.0,  xpMult:1.0 },
    hard:   { spawnInterval:0.68, enemyHpMult:1.45, speedMult:1.18, dmgMult:1.35, xpMult:1.15 },
  },

  // XP to reach next level (gentle early): round(4 + lv*3 + lv*lv*0.5)
  // → L2:8  L3:12  L4:17  L5:24  L6:31  ...
  xpCurve: (lv) => Math.round(4 + lv*3 + lv*lv*0.5),
  ramp: { everySec:28, spawnMul:0.9, hpMul:1.08, max:9 },  // every 28s: faster spawn + tougher
};
```

---

## 15. HOW-TO Recipes

### Thêm màn chơi (endless ramp)
Edit `WAVE_CONFIG` array in code to adjust when new enemy types spawn, or tweak `rampFunction()` to change difficulty curve.

### Nâng độ khó (tổng thể)
Adjust difficulty multipliers in `CONFIG.difficulties[mode]`.

### Đổi điểm / XP reward
`CONFIG.shard.xpPerShard`, `CONFIG.xpPerLevel[]`, `scoreFromTime()`.

### Thêm power-up
Add to `UPGRADE_POOL` array, define effect in `applyUpgrade()`.

### Đổi OVERDRIVE mechanic
Tweak `CONFIG.overdrive` (duration, fire-rate mult, meter charge rate).

---

## 16. Lịch sử cập nhật

| Date | Version | Change |
|---|---|---|
| 2026-06-04 | 1.0 | Launch: core mechanics, 3 difficulties, OVERDRIVE, wave ramp, boss, enemy variety. |
| 2026-06-04 | 1.0.1 | QA fixes: discrete contact damage + i-frames (was per-frame stacking); remove dead enemies + guard double-kills; faster XP flow; floor game-over score. |
| 2026-06-04 | 1.0.2 | UI/UX fixes: hide HUD behind overlays (was bleeding through menu/level-up/pause/game-over); level-up card name/description no longer run together; taller+legible HUD bars; decluttered damage numbers. |

**Last updated:** 2026-06-04

## 17. Testing

QA evidence in [`tests/TEST-REPORT.md`](tests/TEST-REPORT.md) — 11/11 PASS, 0 console errors, screenshots in `tests/screenshots/`. Captured via real-logic playthroughs (an inert `window._autoplay` bot drives the actual tick functions) since headless/background tabs throttle `requestAnimationFrame`.
