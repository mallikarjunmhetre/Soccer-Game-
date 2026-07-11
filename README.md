# ⚽ Penalty Shootout — Football Game

> A **fully browser-based, single-file 2D football penalty shootout game** featuring a realistic animated pitch, intelligent goalkeeper AI, streak-based scoring, and polished visual effects. No frameworks, no dependencies, no server — just open and play.

---

## 📋 Table of Contents

1. [Demo & Quick Start](#-demo--quick-start)
2. [Project Structure](#-project-structure)
3. [Architecture Overview](#-architecture-overview)
4. [Technology Stack](#-technology-stack)
5. [Game Mechanics](#-game-mechanics)
6. [Goalkeeper AI System](#-goalkeeper-ai-system)
7. [Scoring System](#-scoring-system)
8. [Animation Engine](#-animation-engine)
9. [Rendering Pipeline](#-rendering-pipeline)
10. [Design System](#-design-system)
11. [Difficulty System](#-difficulty-system)
12. [Controls Reference](#-controls-reference)
13. [State Management](#-state-management)
14. [Math & Physics](#-math--physics)
15. [Browser Compatibility](#-browser-compatibility)
16. [Known Limitations](#-known-limitations)
17. [Future Improvements](#-future-improvements)
18. [License](#-license)

---

## 🚀 Demo & Quick Start

```bash
# No installation needed — just open the file in any modern browser
double-click  →  index.html
```

Or navigate to:
```
c:\Users\Mallikarjun\OneDrive\Desktop\ARJUN\football-penalty-shootout\index.html
```

> **Requirements:** Any modern browser (Chrome 90+, Firefox 88+, Edge 90+, Safari 14+)  
> **Internet:** Only needed to load Google Fonts. Game logic runs 100% offline.

---

## 📁 Project Structure

```
football-penalty-shootout/
│
├── index.html          ← Entire game (HTML + CSS + JS in one file)
└── README.md           ← This file
```

> The entire game is intentionally shipped as a **single self-contained HTML file** for maximum portability — no build tools, no package managers, no dependencies to install.

---

## 🏗️ Architecture Overview

The game follows a **layered single-file architecture** with clear separation of concerns inside the HTML file:

```
┌─────────────────────────────────────────────────────────────┐
│                        index.html                           │
├────────────────┬──────────────────────┬─────────────────────┤
│   HTML Layer   │      CSS Layer       │   JavaScript Layer  │
│                │                      │                     │
│  - DOM Shell   │  - Design Tokens     │  - Game State       │
│  - Scoreboard  │  - Layout System     │  - Shot Logic       │
│  - Controls    │  - Component Styles  │  - GK AI Engine     │
│  - Canvas Wrap │  - Animations        │  - Drawing Engine   │
│  - Game Over   │  - Responsive Grid   │  - Physics/Math     │
│    Screen      │  - Glassmorphism     │  - Event Handlers   │
└────────────────┴──────────────────────┴─────────────────────┘
```

### Module Breakdown (JavaScript IIFE)

All JavaScript is wrapped in an **Immediately Invoked Function Expression (IIFE)** to avoid polluting the global scope:

```
IIFE
├── Canvas Setup            — 2 canvases (game + confetti overlay)
├── Difficulty Config       — Easy / Medium / Hard parameter tables
├── Game State              — createState() factory function
├── Field Geometry          — FIELD constant object (goal/spot coords)
├── Goalkeeper State        — resetGkState() factory
├── Ball State              — resetBallState() factory
├── Crowd Particles         — Pre-generated ambient particle array
├── Confetti System         — spawnConfetti() + updateConfetti()
├── Direction Grid          — 3×3 click grid, maps to goal coords
├── UI Bindings             — All event listeners for sliders/buttons
│
├── Shot Logic              — takeShot()
│   ├── Target Calculation  — Bezier end-point per direction cell
│   ├── Miss Chance         — Power/height/direction miss probability
│   ├── Outcome Resolution  — computeGkOutcome() → 'goal'|'save'|'miss'|'post'
│   └── GK Dive Setup       — setupGkDive() with reaction delay
│
├── Animation Engine        — startShotAnimation() → shotFrame() RAF loop
│   ├── Ball Updater        — Quadratic Bézier + perspective scale
│   └── GK Updater          — Lerp dive with easeOut
│
├── Stats & Scoring         — updateStats(), bonus point calculation
├── Overlay System          — showOverlay(), streak banner
├── Scoreboard UI           — updateScoreboard(), updateStatsUI()
├── Game Over               — showGameOver(), performance rating
├── Reset                   — resetGame()
│
└── Drawing Engine          — drawScene() → layer-by-layer render
    ├── drawStands()        — Stadium seating + floodlights
    ├── drawCrowd()         — Animated crowd dot particles
    ├── drawPitch()         — Grass + mowing stripes
    ├── drawPenaltyArea()   — Line markings, D-arc, 6-yard box
    ├── drawGoal()          — 3D metallic posts + crossbar
    ├── drawNet()           — Grid lines inside goal
    ├── drawPenaltySpot()   — Spot circle
    ├── drawGoalkeeper()    — Full animated GK figure
    ├── drawBall()          — Ball with pentagon patches + glint
    └── drawHUD()           — Progress bar + labels
```

---

## 🛠️ Technology Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| **Structure** | HTML5 | Semantic shell, accessible elements |
| **Styling** | Vanilla CSS | Full control, CSS custom properties, no overhead |
| **Logic** | Vanilla JavaScript (ES5 strict mode) | Maximum browser compatibility |
| **Rendering** | HTML5 Canvas API | Hardware-accelerated 2D drawing |
| **Animation** | `requestAnimationFrame` | Smooth 60fps rendering |
| **Fonts** | Google Fonts (Orbitron + Outfit) | Premium football-game aesthetic |
| **Build Tool** | None | Zero-dependency single file |

### Canvas Setup

```
┌──────────────────────────────────────────┐
│           #canvas-wrap (div)             │
│  ┌─────────────────────────────────────┐ │
│  │  #gameCanvas   (W=900 × H=560 px)   │ │  ← Game rendering
│  ├─────────────────────────────────────┤ │
│  │  #confettiCanvas (same dimensions)  │ │  ← Confetti overlay
│  ├─────────────────────────────────────┤ │
│  │  #overlay (div, pointer-events:none)│ │  ← Text messages
│  └─────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```

- **Logical resolution:** 900 × 560 px (scaled via CSS `width: 100%`)
- **Two-canvas approach:** Separates costly confetti clearing from the main scene

---

## 🎮 Game Mechanics

### Game Flow

```
START
  │
  ▼
Select Difficulty ──────────────────────────────────┐
  │                                                  │
  ▼                                                  │
Configure Shot (Direction + Power + Height + Curve)  │
  │                                                  │
  ▼                                                  │
Press SHOOT                                          │
  │                                                  │
  ├──► Miss Chance Roll ──► MISS                     │
  │                                                  │
  ├──► GK AI Roll ──► POST / CROSSBAR                │
  │                                                  │
  ├──► GK AI Roll ──► SAVE                           │
  │                                                  │
  └──► GOAL ──► Update Streak & Points               │
                    │                                │
  shots < 10 ◄──── │ ────────────────────────────────┘
  shots = 10 ──► GAME OVER SCREEN
```

### Shot Configuration Parameters

| Parameter | Input Type | Range | Effect |
|-----------|-----------|-------|--------|
| **Direction** | 3×3 Grid | 9 zones | Sets X/Y target inside goal |
| **Power** | Slider | 20% – 100% | Affects save chance & bonus points |
| **Height** | Button | Low / Mid / High | Adjusts Y target in goal |
| **Curve / Spin** | Slider | -5 to +5 | Shifts Bézier mid-point horizontally |

### Outcome Types

| Outcome | Trigger | Streak Effect | Points |
|---------|---------|--------------|--------|
| ⚽ **Goal** | Ball clears GK + post check | +1 streak | +100 base |
| 🧤 **Save** | GK AI intercepts | Reset to 0 | +0 |
| 💨 **Miss** | Random miss roll | Reset to 0 | +0 |
| 🏗️ **Post/Bar** | 5% chance any non-miss | Reset to 0 | +0 |

---

## 🤖 Goalkeeper AI System

The GK is a **probabilistic AI** — not deterministic. Each shot is evaluated independently using the following pipeline:

### Decision Pipeline

```
computeGkOutcome(dir, power, curve, height)
│
├── 1. Base Save Chance      ← from difficulty table
├── 2. Streak Bonus          ← +3% per streak level (max +20%)
│
├── 3. Correct Guess Roll    ← GK guesses direction (prediction accuracy %)
│      ├── CORRECT → full save probability
│      └── WRONG   → saveP × 0.35 (much harder to save)
│
├── 4. Modifiers Applied:
│      ├── Power Penalty     ← high power = harder to save
│      ├── Height Modifier   ← high shots slightly harder
│      ├── Curve Modifier    ← curved shots harder
│      └── Top Corner Bonus  ← -14% save chance for top corners
│
└── 5. Roll:
       ├── < 5%       → POST
       ├── < 5%+saveP → SAVE
       └── else       → GOAL
```

### GK Dive Behavior

- **On Save:** Dives directly toward ball's target zone
- **On Miss/Goal:** Dives in a *potentially wrong* direction (50% chance wrong side)
- **Reaction Delay:** Timed `setTimeout` simulates human reaction lag
  - Easy: 420ms | Medium: 280ms | Hard: 160ms

### Dynamic Difficulty Scaling

As your streak grows, the GK gets harder automatically:

```
effectiveSaveChance = baseSaveChance + min(streak × 0.03, 0.20)
effectivePrediction = basePrediction + min(streak × 0.015, 0.10)
```

---

## 🏆 Scoring System

### Base Points

| Event | Points |
|-------|--------|
| Goal scored | **+100** |
| Save / Miss / Post | +0 |

### Bonus Points

| Bonus | Condition | Extra Points |
|-------|-----------|-------------|
| 💥 Power Shot | Power ≥ 85% | **+20** |
| 🔥 Streak Bonus | Streak ≥ 3 | **+15 × streak** |
| 🎯 Top Corner | Top-left or Top-right cell | **+50** |

### Streak Bonus Examples

| Streak | Bonus Points |
|--------|-------------|
| 3× | +45 |
| 5× | +75 |
| 8× | +120 |
| 10× | +150 |

### Performance Rating (Game Over)

| Accuracy | Rating | Trophy |
|----------|--------|--------|
| ≥ 80% | Legendary Shooter! | 🥇 |
| ≥ 60% | Excellent Performance! | 🏆 |
| ≥ 40% | Decent Effort! | 🥈 |
| < 40% | Better Luck Next Time! | 😅 |

---

## 🎬 Animation Engine

### Ball Animation

The ball travels along a **Quadratic Bézier Curve**:

```
P(t) = (1-t)²·P0 + 2(1-t)t·P1 + t²·P2

Where:
  P0 = Penalty spot (start)
  P1 = Mid-arc control point (height + curve offset)
  P2 = Target position in goal (end)
  t  = Animation progress [0 → 1]
```

**Perspective Scaling:** Ball shrinks as it moves toward the goal:
```
ball.scale = 1 - 0.42 × easeInOut(t)
```

**Arc Height by Shot Type:**

| Height | Arc Multiplier |
|--------|---------------|
| Low    | × 0.5 |
| Mid    | × 0.9 |
| High   | × 1.4 |

### Goalkeeper Dive Animation

GK position is linearly interpolated with cubic ease-out:
```
gk.x = lerp(restX, diveX, easeOut(diveProgress))
gk.y = lerp(restY, diveY, easeOut(diveProgress))
```

### Confetti System

On goals, 130 confetti particles are spawned:
- Random X start position, fall from top
- Gravity applied each frame (`vy += 0.12`)
- Fades out via `life -= 0.008`
- Rotates with random spin speed

### Easing Functions

```javascript
easeInOut(t) → t < 0.5 ? 2t² : -1+(4-2t)t   // Ball travel
easeOut(t)   → 1 - (1-t)³                     // GK dive
```

---

## 🖼️ Rendering Pipeline

Each frame calls `drawScene()` which renders layers **bottom-to-top**:

```
Layer 1: drawStands()       — Stadium, seats, floodlights (static bg)
Layer 2: drawCrowd()        — Animated particle crowd (sine wave)
Layer 3: drawPitch()        — Grass gradient + mowing stripes
Layer 4: drawPenaltyArea()  — White line markings + D-arc
Layer 5: drawGoal()         — Metallic 3D posts + crossbar
Layer 6: drawNet()          — Semi-transparent net grid
Layer 7: drawPenaltySpot()  — White spot dot
Layer 8: drawGoalkeeper()   — Full GK character (11 sub-elements)
Layer 9: drawBall()         — Ball with patches, glint, shadow
Layer 10: drawHUD()         — Progress bar, shot count, difficulty label
```

### Goalkeeper Rendering Sub-layers

The goalkeeper is drawn as a composite of **11 drawn elements**:
1. Ground shadow ellipse
2. Shorts (left leg)
3. Shorts (right leg)
4. Boot (left)
5. Boot (right)
6. Jersey (gradient filled rounded rect)
7. Jersey number `"1"`
8. Left arm + glove
9. Right arm + glove
10. Head (radial gradient)
11. Cap + brim + eyes + mouth

---

## 🎨 Design System

### Color Palette (CSS Custom Properties)

```css
--bg-dark:      #0d1117   /* Page background */
--green-dark:   #1a4a1a   /* Pitch dark */
--green-mid:    #236b23   /* Pitch mid */
--gold:         #f5c518   /* Primary accent (Orbitron, buttons) */
--red:          #e63946   /* GK jersey, saves indicator */
--card-bg:      rgba(255,255,255,0.06)   /* Glassmorphism card */
--card-border:  rgba(255,255,255,0.12)   /* Card border */
--text-primary: #f0f4f8                  /* Main text */
--text-muted:   rgba(255,255,255,0.55)   /* Secondary text */
```

### Typography

| Font | Weight | Usage |
|------|--------|-------|
| **Orbitron** | 700, 900 | Title, scoreboard values, overlays, buttons |
| **Outfit** | 300–900 | All body text, labels, stats |

### Component Design Patterns

- **Glassmorphism Cards:** `background: rgba(255,255,255,0.06)` + `backdrop-filter: blur(8px)` + `border: 1px solid rgba(255,255,255,0.12)`
- **Gold Gradient Buttons:** `linear-gradient(135deg, #f5c518, #e0a800)`
- **Score Pills:** Rounded cards with colored value display
- **Pulse Glow Animation:** Shoot button breathes with CSS `@keyframes`

### Responsive Layout

```css
/* Grid adapts to screen width */
@media (max-width: 640px)  → 2-column controls grid
@media (max-width: 420px)  → 1-column controls grid
```

Canvas scales via `width: 100%` while preserving internal 900×560 resolution.

---

## ⚙️ Difficulty System

### Difficulty Parameter Table

| Parameter | Easy | Medium | Hard |
|-----------|------|--------|------|
| Base Save Chance | 28% | 44% | 62% |
| Reaction Delay | 420ms | 280ms | 160ms |
| Prediction Accuracy | 30% | 58% | 80% |
| Max Dynamic Bonus | +20% | +20% | +20% |

### Dynamic Scaling Formula

```
streakBonus       = min(streak × 0.03, 0.20)
effectiveSaveChance = min(baseSave + streakBonus, 0.78)
effectivePrediction = min(basePred + streakBonus × 0.5, 0.90)
```

This means even on Easy mode, a player with a 7-goal streak will face a GK that saves ~49% of shots.

---

## 🕹️ Controls Reference

| Control | Type | Description |
|---------|------|-------------|
| **Direction Grid** | 3×3 click grid | Select any of 9 goal zones |
| **Power Slider** | Range 20–100 | Shot speed & power |
| **Low / Mid / High** | Button group | Shot height above ground |
| **Curve Slider** | Range -5 to +5 | Left (−) or right (+) spin |
| **SHOOT Button** | Click | Fire the penalty kick |
| **Reset Game** | Click | Clear all stats, restart |
| **Play Again** | Click (game over) | New session, keep difficulty |
| **Easy/Medium/Hard** | Button group | Change AI difficulty |

### Direction Grid Zones

```
┌──────────┬──────────┬──────────┐
│ Top-Left │  Top-Mid │ Top-Right│  ← Hardest to save (+50 bonus)
├──────────┼──────────┼──────────┤
│ Mid-Left │  Center  │ Mid-Right│
├──────────┼──────────┼──────────┤
│ Bot-Left │ Bot-Mid  │ Bot-Right│  ← Slightly more miss-prone
└──────────┴──────────┴──────────┘
```

---

## 📊 State Management

All game state lives in a single `state` object created by `createState()`:

```javascript
state = {
  shots:       0,    // Total shots taken
  goals:       0,    // Goals scored
  saves:       0,    // GK saves
  misses:      0,    // Off-target shots
  postHits:    0,    // Post / crossbar hits
  topCorner:   0,    // Top-corner goals scored
  streak:      0,    // Current consecutive goal streak
  bestStreak:  0,    // All-time best streak this session
  points:      0,    // Total points scored
  bonusPoints: 0,    // Bonus points only
  difficulty: 'easy',
  animating:  false, // Prevents double-shooting during animation
  maxShots:   10,    // Shots per game
  gameOver:   false,
}
```

**State Reset on `resetGame()`:** All fields reset except `difficulty` (preserved).

---

## 📐 Math & Physics

### Quadratic Bézier Curve

```
B(t) = (1-t)²P₀ + 2(1-t)tP₁ + t²P₂,  t ∈ [0,1]
```

Applied to both X and Y independently for the ball's flight path.

### Miss Probability Calculation

```
missChance = 0.06 (base)
           + 0.10 (if power < 35%)
           + 0.07 (if bottom-corner cell)
           + 0.05 (if high shot to top row)
```

### GK Save Probability

```
saveP = baseSave
      - powerPenalty      (power − 0.5) × 0.25
      + heightMod         (+0.04 low / -0.05 high)
      + curveMod          (-0.08 if |curve| > 2)
      + topCornerMod      (-0.14 if top corner)
      × 0.35              (if GK guesses wrong direction)

saveP = clamp(saveP, 0.04, 0.90)
```

---

## 🌐 Browser Compatibility

| Browser | Version | Status |
|---------|---------|--------|
| Chrome | 90+ | ✅ Full support |
| Edge | 90+ | ✅ Full support |
| Firefox | 88+ | ✅ Full support |
| Safari | 14+ | ✅ Full support |
| Opera | 76+ | ✅ Full support |
| IE 11 | — | ❌ Not supported |

> Uses: `Canvas 2D API`, `requestAnimationFrame`, `CSS custom properties`, `backdrop-filter`, `CSS Grid`

---

## ⚠️ Known Limitations

- **No sound effects** — browser audio would require user interaction unlocking; kept out of scope for simplicity.
- **No local storage** — high scores are not persisted between sessions.
- **No touch drag controls** — direction must be tapped from the grid (no swipe-to-aim).
- **Fixed 10-shot game** — session length is not yet configurable.
- **Single player only** — no multiplayer or 2-player mode.
- **No mobile landscape optimization** — works but may need scrolling on small phones.

---

## 🚀 Future Improvements

| Feature | Priority | Complexity |
|---------|----------|-----------|
| 🔊 Sound effects (kick, save, crowd roar) | High | Medium |
| 💾 Local storage high score leaderboard | High | Low |
| 📱 Swipe/drag aim on mobile | Medium | Medium |
| 🎮 Configurable shot count (5/10/20) | Medium | Low |
| 🌙 Animated crowd wave during goals | Medium | Medium |
| 🔄 Sudden death shootout mode | Low | Medium |
| 🧠 Improved GK learning AI (tracks patterns) | Low | High |
| 🏟️ Multiple stadium themes | Low | Medium |
| 👥 Two-player pass-and-play mode | Low | High |
| 📊 Post-game shot map visualization | Low | Medium |

---

## 👤 Author

**Mallikarjun (Arjun)**  
Built with ❤️ using pure HTML, CSS & JavaScript — no frameworks needed.

---

## 📄 License

```
MIT License

Copyright (c) 2026 Mallikarjun

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
```

---

<div align="center">

**⚽ Score goals. Build streaks. Beat the keeper. ⚽**

*Open `index.html` in your browser and play instantly!*

</div>
