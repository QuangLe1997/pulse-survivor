# ⚡ PULSE SURVIVOR — Charge the Overdrive

A neon **survivors-like / bullet-heaven**. You only move — your weapon auto-fires. Dodge escalating waves of geometric neon enemies, hoover up the glowing shards they drop to fill your XP bar, and pick from 3 random upgrades on every level-up. Shards also charge the signature **OVERDRIVE** meter: when it's full, unleash a 5-second burst of rapid fire and invincibility. A boss storms in around 90 seconds. Survive as long as you can.

**▶ Live:** https://quangle1997.github.io/pulse-survivor/ · part of [QUANG ARCADE](https://quangle1997.github.io/arcade/)

## Features
- **Core loop:** top-down auto-shooter — move to position & dodge, weapon auto-targets the nearest foe.
- **Signature OVERDRIVE hook:** charge → unleash rhythm (5s rapid fire + invincibility + screen pop).
- **Roguelite progression:** 9 stackable upgrades (damage, extra projectiles, fire rate, magnet, speed, max HP, spread, overdrive charge, piercing shots).
- **Enemy variety:** minions, ranged shooters, heavy brutes, and a spinning boss with bullet sprays.
- **Wave ramp:** difficulty escalates every ~28s; 3 modes (Easy / Normal / Hard).
- **Controls:** drag-to-move (mobile) + WASD/arrows + Space for overdrive.
- **Juice:** hit flashes, screen shake, damage numbers, shard pickup pings, level-up fanfare.
- **Audio:** synthesized synthwave music engine + WebAudio SFX, mute toggle.
- **Scoring:** time survived + kills + combos, best score in `localStorage`.
- Single `index.html`, zero build, mobile-first portrait, deployed on GitHub Pages.

## Run locally
```bash
python3 -m http.server 8773
# open http://localhost:8773
```

## Docs
See [`DOCS.md`](DOCS.md) for the full technical breakdown — state machine, balance config (single source of truth), upgrade pool, scoring formulas, and how-to recipes for tuning.

---
Built by [QuangLe1997](https://github.com/QuangLe1997) · crafted with ♥ & Claude Code.
