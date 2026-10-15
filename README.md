<div align="center">

# 🔥 Digi-ধুনুচি

### *Balance the flame.*

A tilt-controlled mobile web game inspired by the **dhunuchi** — the clay incense burner carried during Bengal's Durga Puja *dhunuchi naach*. Rest your phone flat on your palm, tilt it, and keep the flame inside the bowl while the wind fights back.

[![Play Live](https://img.shields.io/badge/▶_Play_Live-digi--dhunuchi.netlify.app-ff9a1f?style=for-the-badge)](https://digi-dhunuchi.netlify.app/)

![Single file](https://img.shields.io/badge/Single_file-index.html-120d24?style=flat-square)
![Dependencies](https://img.shields.io/badge/Dependencies-0-success?style=flat-square)
![Canvas 2D](https://img.shields.io/badge/Rendering-Canvas_2D-orange?style=flat-square)
![Web Audio](https://img.shields.io/badge/Audio-Web_Audio_API-blueviolet?style=flat-square)
![Status](https://img.shields.io/badge/Status-Prototype-yellow?style=flat-square)

<img src="docs/screenshots/overview.png" alt="Digi-Dhunuchi screens: menu, early gameplay, later gameplay, game over" width="100%">

</div>

---

## Table of contents

1. [What is this prototype?](#1-what-is-this-prototype)
2. [How to play](#2-how-to-play)
3. [Screenshots](#3-screenshots)
4. [Features](#4-features)
5. [How the game works (design deep-dive)](#5-how-the-game-works-design-deep-dive)
6. [Difficulty curve at a glance](#6-difficulty-curve-at-a-glance)
7. [Tech stack and architecture](#7-tech-stack-and-architecture)
8. [Run it locally](#8-run-it-locally)
9. [Deploy it](#9-deploy-it)
10. [Tuning the difficulty](#10-tuning-the-difficulty)
11. [Browser and device support](#11-browser-and-device-support)
12. [Troubleshooting](#12-troubleshooting)
13. [Known limitations and ideas](#13-known-limitations-and-ideas)
14. [Credits](#14-credits)

---

## 1. What is this prototype?

**Digi-ধুনুচি** ("Digital Dhunuchi") turns a phone into a physical controller. A *dhunuchi* is a clay burner filled with glowing coconut-husk coals and incense, danced with during Durga Puja aarti. The skill is keeping it steady while it smokes and flares. This prototype recreates that feeling digitally:

- A **flame** sits in a top-down view of a decorated clay dhunuchi.
- The phone's **gyroscope/accelerometer** is your hand. Tilt the phone and the flame slides the way you tilt.
- A **wind** pushes the flame toward the rim. Every second it blows harder and flips direction faster.
- You score by **survival time**. When the flame stays outside the dashed ring for too long, it spills and the round ends.

It is designed as a **pass-the-phone party game**: after each round, tap *Next player* and the local leaderboard ranks everyone. That makes it a good fit for exhibitions, college fests, stalls and pandals.

**The core idea in one line:** *a survival game where your own hand is the controller and the difficulty never stops rising.*

### Why it is interesting as a prototype

| Aspect | What it demonstrates |
|---|---|
| Physical input | Real-world motion via `deviceorientation` with an accelerometer fallback |
| Game feel | Particle flame, smoke, sparks, haptics, and a procedurally synthesised *dhak*-style beat that speeds up |
| Cultural design | Terracotta clay body, alpona (rice-paste) rim pattern, Bengali wordmark, serif typography |
| Zero-friction delivery | One HTML file, no install, no assets, works offline once loaded |
| Fairness by design | Wind is capped *below* player control strength, so the game is always theoretically winnable |

---

## 2. How to play

1. Open the [live link](https://digi-dhunuchi.netlify.app/) on your phone (it must be served over **HTTPS**).
2. Tap **Light the dhunuchi**. On iPhone, allow *Motion & Orientation* access when asked.
3. During the **3-2-1 countdown**, hold the phone **flat on your palm**. Your pose at the end of the countdown becomes the neutral "level" position.
4. Tilt the phone to slide the flame. The **red arrow on the rim** shows where the wind is pushing it, so tilt *away* from the arrow.
5. Keep the flame inside the dashed ring. The ring turns from gold to red as the flame nears the edge, and the phone vibrates as a warning.
6. When it spills, your time and level are shown. Tap **Next player** to hand the phone over.

**No motion sensor?** (desktop, or sensor blocked) The game falls back to:

| Input | How |
|---|---|
| Touch / mouse drag | Press anywhere and drag; the flame is pushed toward where your finger is relative to the bowl centre |
| Keyboard | Arrow keys or `W` `A` `S` `D` |

---

## 3. Screenshots

> Captured at a 390 × 844 mobile viewport.

| Menu | Gameplay, Level 1 | Gameplay, Level 3 | Game over |
|:---:|:---:|:---:|:---:|
| <img src="docs/screenshots/01-menu.png" width="200" alt="Menu screen"> | <img src="docs/screenshots/03-gameplay-level1.png" width="200" alt="Gameplay at level 1"> | <img src="docs/screenshots/04-gameplay-level3.png" width="200" alt="Gameplay at level 3"> | <img src="docs/screenshots/05-game-over.png" width="200" alt="Game over screen with leaderboard"> |

**Anatomy of the gameplay screen**

| Element | Where | Meaning |
|---|---|---|
| Timer (large, amber) | Top left | Seconds survived. This is your score |
| `Level N` | Top centre | Increases every 10 s; a toast announces each new level |
| `Best` + ♪ button | Top right | Personal best (saved on the device) and sound mute toggle |
| Dashed ring | On the bowl | The safe boundary; gold when safe, pulsing red near the edge |
| Red arrow | On the rim | Direction the wind is currently pushing; it grows with wind strength |
| Small circle with dot | Below the bowl | **Tilt indicator** showing how the game reads your tilt |

---

## 4. Features

- 🎮 **Gyro-tilt control** with automatic calibration and screen-rotation handling
- 🔁 **Three input methods**: motion sensors, touch/mouse drag, keyboard
- 🔥 **Procedural fire**: up to 420 additive-blended particles (flame, smoke, sparks) that flare as the flame nears the edge
- 🌬️ **Escalating wind** with wandering direction, gusts and random "kicks"
- 🥁 **Synthesised audio**, with no audio files: a dhak-like rhythm whose tempo rises with level, plus a low thud on game over
- 📳 **Haptics**: a short buzz near the edge and a pattern on spill (where the Vibration API exists)
- 👥 **Pass-the-phone leaderboard**: top 6 players of the session
- 💾 **Persistent personal best** via `localStorage`
- 📱 **Mobile-first**: safe-area insets, no scrolling or zoom, screen wake lock during play
- 🧪 **Built-in diagnostics** on the menu: shows whether the page is secure and which sensor source is live

---

## 5. How the game works (design deep-dive)

### 5.1 The control model

The flame has a **position** `(x, y)` measured in bowl radii (the dashed ring is at distance 1.0) and a **velocity** `(vx, vy)`. Each frame, three things accelerate it:

```
acceleration = tilt × control  +  wind  +  gust
velocity    += acceleration × dt
velocity    *= e^(−1.1 × dt)            // damping (friction)
position    += velocity × dt
```

- **Tilt term**: `tilt` is the phone tilt divided by `maxTilt` (22°), clamped to −1…1, and smoothed. At full tilt the push is `control = 5.2`.
- **Wind term**: `(cos θ, sin θ) × amp`, where `θ` is the wind angle and `amp` grows with time.
- **Gust term**: a smooth pseudo-random wobble made of summed sines, so the wind never feels perfectly steady.

Because of damping, a constant acceleration `a` settles to a terminal speed of about `a / 1.1`. This is what makes the game *skill-based*: the flame has momentum, so you have to counter-tilt **early**, not after it is already at the rim.

### 5.2 Wind behaviour

| Mechanism | Rule |
|---|---|
| Strength | `amp = min(0.9 + 0.05·t, 4.4)`, eased in over the first 1.5 s |
| Direction flips | Every `max(0.75, 3.2·e^(−t/40))` seconds, the wind target turns to roughly the **opposite** side (π ± 0.7 rad of jitter) |
| Turn speed | The wind angle rotates toward its target at `(1.6 + 0.06·t)` rad/s, so it sweeps rather than snaps |
| Gust amplitude | `min(0.15 + 0.012·t, 1.2)` |
| Random kicks | From **t > 10 s**, an impulse of size `0.25 + 0.012·min(t, 90)` in a random direction, every `max(0.8, 2.8 − 0.03·t)` s |

### 5.3 Fail condition (and the grace period)

- Distance from centre `> 1` starts an "outside" timer. If the flame stays out for more than **0.25 s** (`graceOut`), it spills.
- While inside, the timer drains at 2× speed, so a brief brush with the edge is forgiven.
- On spill, the flame keeps its outward velocity, bursts into 70 sparks, a low sine thud plays, and the phone vibrates.

### 5.4 Why the game is always winnable

`windCap = 4.4` is deliberately **lower** than `control = 5.2`. Even at maximum wind, a perfectly timed full tilt can overpower it. The game gets harder through **reaction demand** (faster flips, more kicks), not through an impossible force.

### 5.5 Worked example: the game at t = 30 s

| Quantity | Formula | Value |
|---|---|---|
| Level | `1 + floor(30/10)` | **4** |
| Wind strength | `0.9 + 0.05 × 30` | **2.40** |
| Time between wind flips | `3.2 · e^(−30/40) = 3.2 × 0.4724` | **≈ 1.51 s** |
| Wind turn speed | `1.6 + 0.06 × 30` | **3.4 rad/s** |
| Gust amplitude | `0.15 + 0.012 × 30` | **0.51** |
| Kick impulse / interval | `0.25 + 0.36` / `max(0.8, 2.8 − 0.9)` | **0.61 / 1.9 s** |
| Beat tempo | `84 + 4 × 7` | **112 BPM** |
| Spare control (full tilt − wind) | `5.2 − 2.4` | **2.8** |
| Unopposed wind speed | `2.4 / 1.1` | **≈ 2.18 radii/s**, so the flame crosses the ring in about **0.46 s** |

That last line is the heart of the design: at 30 s, if you stop reacting for half a second, the flame is gone.

### 5.6 Calibration and orientation

- Whenever you are on the **menu or countdown**, the current phone angle is stored as the "zero" reference, so you do not need to hold the phone perfectly flat.
- Tilt axes are remapped for the current screen rotation (0°, 90°, 180°, 270°).
- If `deviceorientation` never fires, tilt is derived from the **accelerometer** (`devicemotion`) with `atan2` as a fallback.

### 5.7 Visual and audio design

- **Clay body**: radial gradient terracotta with a drop shadow.
- **Alpona rim**: two concentric rings and 28 rice-paste petals drawn on the rim.
- **Bowl and coals**: dark bowl with 46 pulsing embers placed on a golden-angle (phyllotaxis) spiral.
- **Flame**: three pre-rendered radial-gradient sprites (white-yellow, amber, red) layered with `lighter` compositing; size and flicker are driven by sine waves, and the flame flares when `distance > 0.65`.
- **Audio**: every sound is an oscillator with an exponential frequency and gain ramp. The beat runs an 8-step pattern at `84 + 7·min(level, 15)` BPM, so it is 91 BPM at level 1 and tops out at 189 BPM.

---

## 6. Difficulty curve at a glance

| Time | Level | Wind strength | Flip interval | Notes |
|---:|:---:|:---:|:---:|---|
| 0 s | 1 | 0.90 | 3.2 s | Gentle introduction |
| 10 s | 2 | 1.40 | ≈ 2.5 s | **Random kicks begin** |
| 20 s | 3 | 1.90 | ≈ 1.94 s | |
| 30 s | 4 | 2.40 | ≈ 1.51 s | |
| 40 s | 5 | 2.90 | ≈ 1.18 s | |
| 58 s | 6 | 3.80 | **0.75 s (floor)** | Flip speed stops tightening |
| 70 s | 8 | **4.40 (cap)** | 0.75 s | Wind maxed out; only kicks and gusts keep growing |
| 90 s | 10 | 4.40 | 0.75 s | Kick size reaches its cap |

---

## 7. Tech stack and architecture

| Layer | Technology |
|---|---|
| Language | Vanilla JavaScript (ES6+), HTML5, CSS3 |
| Rendering | HTML5 Canvas 2D with additive blending and pre-rendered gradient sprites |
| Input | `DeviceOrientationEvent`, `DeviceMotionEvent`, Pointer Events, Keyboard Events |
| Audio | Web Audio API (oscillators and gain envelopes) |
| Device APIs | Vibration API, Screen Wake Lock API, Screen Orientation API |
| Storage | `localStorage` (personal best only) |
| Dependencies | **None** |

### Project structure

```
digi-dhunuchi/
├── index.html              # The entire game: markup, styles and logic
├── README.md
└── docs/
    └── screenshots/        # Images used in this README
```

### Inside `index.html`

| Section (in code) | Responsibility |
|---|---|
| `CFG` | All tuning constants in one object |
| State `S` | Mode (`menu` / `count` / `play` / `over`), position, velocity, wind, timers |
| Input | Sensor reading, calibration, orientation remapping, pointer and keyboard fallbacks |
| Audio | `tone()` helper that synthesises every sound |
| Particles | Pooled particle system with three types: flame, smoke, spark |
| Game flow | `startCount`, `begin`, `die`, `showOver` |
| Simulation | `sim(dt)`: wind, forces, integration, fail check, beat, HUD |
| Rendering | `draw()`: layered canvas painter |
| UI wiring | Permission request, buttons, menu diagnostics |

The loop runs on `requestAnimationFrame` with a clamped timestep (`dt ≤ 33 ms`), so physics stays stable if a frame is dropped. Canvas resolution is capped at 2× device pixel ratio for performance.

---

## 8. Run it locally

No build step is needed.

```bash
git clone https://github.com/anxshh007/digi-dhunuchi.git
cd digi-dhunuchi
```

**On a desktop**, open `index.html` directly. Use drag or the arrow keys to play.

**On a phone**, motion sensors need a **secure context**, so serve the folder over HTTPS. A quick way is a tunnel:

```bash
python3 -m http.server 8000
# in another terminal, expose it over HTTPS, for example:
npx localtunnel --port 8000
```

Then open the HTTPS URL on your phone.

---

## 9. Deploy it

Because it is a single static file, any static host works.

- **Netlify**: drag the folder onto <https://app.netlify.com/drop> (this is how the live demo is hosted).
- **GitHub Pages**: *Settings → Pages → Deploy from branch → `main` / root*.
- **Vercel / Cloudflare Pages**: import the repo, with no build command and `/` as the output directory.

All of these serve over HTTPS, which Chrome requires before it will deliver sensor data.

---

## 10. Tuning the difficulty

Everything is in the `CFG` object near the top of the script:

```js
const CFG = {
  maxTilt:   22,   // degrees of tilt that give full control
  control:   5.2,  // control strength (higher = more responsive)
  windStart: .9,   // starting wind strength
  windGrow:  .05,  // wind increase per second
  windCap:   4.4,  // wind never exceeds this (keep below control to stay winnable)
  switchStart: 3.2,// seconds between wind flips at the start
  switchMin: .75,  // fastest flip interval
  graceOut:  .25   // seconds the flame may stay outside before you are out
};
```

| Goal | Change |
|---|---|
| Easier party mode | Lower `windGrow` (e.g. `.03`), raise `graceOut` (e.g. `.4`) |
| Harder / expert | Raise `windGrow`, lower `switchMin` |
| Less wrist movement needed | Lower `maxTilt` (more sensitive) |
| Keep it fair | Always keep `windCap < control` |

---

## 11. Browser and device support

| Platform | Motion control | Notes |
|---|---|---|
| Android, Chrome | ✅ | Needs HTTPS |
| iPhone, Safari | ✅ | Shows a one-time motion-permission prompt |
| Desktop browsers | ➖ | Plays with mouse drag or keyboard |
| Vibration | Android only | iOS Safari does not expose the Vibration API; the game still works |

---

## 12. Troubleshooting

The menu shows a live status line; use it to diagnose problems.

| Message | Meaning and fix |
|---|---|
| `NOT a secure page ...` | Host the game over HTTPS (see [Deploy](#9-deploy-it)) |
| `no sensor data yet` | The browser has not sent motion events. Tilt the phone once, or check the site's motion-sensor permission |
| `orientation sensor` | Normal; tilt is read from the gyroscope |
| `accelerometer fallback` | Orientation events are unavailable; tilt is derived from the accelerometer |
| `Motion access denied ...` | Allow motion access in browser settings, or just drag on the screen |
| Flame drifts at rest | Hold the phone steady during the countdown; that pose becomes the zero point |
| Phone auto-rotates oddly | Lock the screen orientation to portrait |

---

## 13. Known limitations and ideas

**Current limitations**

- The session leaderboard is held in memory and resets when the page reloads (only the personal best persists).
- Player names are automatic (`Player 1`, `Player 2`, ...).
- Vibration is not available on iOS.

**Ideas for future versions** *(not implemented)*

- Editable player names and a persistent all-time leaderboard
- Difficulty modes (Easy / Normal / Expert) using the `CFG` presets
- Shareable result card
- Multiple "festival" themes or flame colours
- PWA manifest for install-to-home-screen and full offline use

---

## 14. Credits

**Designed and developed by Angshuman Bhagat**

[![GitHub](https://img.shields.io/badge/GitHub-anxshh007-181717?style=flat-square&logo=github)](https://github.com/anxshh007)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Angshuman_Bhagat-0A66C2?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/angshuman-bhagat-profilevisit)

Inspired by the dhunuchi tradition of Bengal's Durga Puja.

<div align="center">

*শুভ বিজয়া — may your flame never spill.* 🔥

</div>
