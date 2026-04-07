# Architect Design Document: Crab Cookie Clicker ("nom")

> Single-file browser game. No backend. No auth. Pure client-side HTML/CSS/JS with localStorage persistence.
> Aesthetic: Atari 2600 pixel art. Theme: Crabs eating cookies. Tone: Absurd, punny, addictive.

---

## 1. System Overview

A Cookie Clicker clone where players click a giant cookie to earn cookies, then spend cookies hiring crabs that passively generate more cookies. The game escalates through 10 upgrade tiers from a single baby crab to an interdimensional Crab God. A prestige system ("Molting") resets progress in exchange for permanent multipliers, creating long-term replayability. Everything ships as one HTML file with zero external dependencies.

---

## 2. Tech Stack

| Layer | Choice | Justification |
|-------|--------|---------------|
| Markup | HTML5 (single file) | Zero build step, runs anywhere |
| Styling | Embedded CSS with CSS custom properties | Pixel art via box-shadow sprites, no image assets |
| Logic | Vanilla ES6+ JavaScript | No framework overhead for a game this simple |
| Persistence | localStorage | Only viable client-side option with no backend |
| Game Loop | requestAnimationFrame | Smooth 60fps updates, battery-friendly |
| Layout | CSS Grid + Flexbox | Responsive without media query hell |
| Fonts | System monospace + CSS pixel font via box-shadow | Atari aesthetic without font file dependencies |

---

## 3. Architecture

### 3.1 File Structure

One file: `index.html` containing three embedded blocks:

```
<html>
  <head>
    <style>/* All CSS here */</style>
  </head>
  <body>
    <!-- All HTML here -->
    <script>/* All JS here */</script>
  </body>
</html>
```

### 3.2 JavaScript Architecture

The JS is organized into these logical modules (all in one script block, separated by comment headers):

```
// ===== CONFIG =====
// All game constants, upgrade definitions, achievement definitions

// ===== STATE =====
// Single game state object, save/load functions

// ===== GAME ENGINE =====
// Core loop (requestAnimationFrame), tick calculations, CPS computation

// ===== UPGRADES =====
// Purchase logic, cost scaling, effect application

// ===== PRESTIGE =====
// Molt calculation, reset logic, permanent bonus application

// ===== ACHIEVEMENTS =====
// Check conditions, unlock logic, notification queue

// ===== PARTICLES =====
// Click effects, floating numbers, cookie crumb animations

// ===== RENDERER =====
// DOM updates, number formatting, UI state sync

// ===== INPUT =====
// Click handlers, keyboard shortcuts, touch events

// ===== SAVE SYSTEM =====
// localStorage read/write, auto-save timer, export/import

// ===== INIT =====
// Bootstrap: load save, render initial state, start loop
```

### 3.3 Game State Object

```javascript
const DEFAULT_STATE = {
  cookies: 0,
  totalCookiesBaked: 0,
  totalClicks: 0,
  cookiesPerClick: 1,
  upgrades: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
  moltCount: 0,
  shellShards: 0,
  achievements: [],
  startedAt: null,
  lastSavedAt: null,
  version: 1
};
```

### 3.4 Game Loop

```
requestAnimationFrame loop:
  1. Calculate deltaTime since last frame
  2. Accumulate fractional cookies from CPS * deltaTime
  3. Check achievement conditions
  4. Update particle animations
  5. Render: sync DOM with state (throttled to 4 updates/sec for DOM, 60fps for particles)

Auto-save: every 30 seconds via setInterval
Offline earnings: on load, calculate (now - lastSavedAt) * CPS, cap at 8 hours
```

---

## 4. Game Mechanics

### 4.1 Core Loop

```
Click Cookie --> +cookiesPerClick cookies
  |
  v
Buy Upgrades --> Crabs generate passive CPS (cookies per second)
  |
  v
Reach milestones --> Unlock achievements (some give bonuses)
  |
  v
Hit wall --> Molt (prestige) for Shell Shards --> permanent multiplier
  |
  v
Repeat with higher numbers
```

### 4.2 Click Mechanics

- Base: 1 cookie per click
- Prestige bonus: +10% per Shell Shard (multiplicative)
- Click shows: floating "+N" number, cookie crumb particles, crab claw pinch animation
- Touch support: works on mobile, no double-tap-to-zoom interference (use touch-action: manipulation)

### 4.3 Cookies Per Second (CPS) Formula

```
Total CPS = SUM(upgrade[i].count * upgrade[i].baseCPS) * (1 + 0.10 * shellShards)
```

### 4.4 Cost Scaling Formula

```
cost(n) = baseCost * 1.15^n
```

---

## 5. Upgrade Tree (10 Tiers)

| # | Name | Description | Base Cost | Base CPS |
|---|------|-------------|-----------|----------|
| 1 | Baby Crab | "Smol. Angy. Hungry." | 15 | 0.1 |
| 2 | Sand Crab | "Digs for cookies in the sand. Finds mostly sand." | 100 | 1 |
| 3 | Coconut Crab | "Strong enough to crack coconuts. Cookies are easier." | 1,100 | 8 |
| 4 | Hermit Crab | "Lives in a cookie jar. Rent-free." | 12,000 | 47 |
| 5 | Crab Army | "They march in formation. The formation is 'hungry'." | 130,000 | 260 |
| 6 | Crab King | "Wears a crown made of chocolate chips. Rules with an iron claw." | 1,400,000 | 1,400 |
| 7 | Crab Scientist | "Has discovered that cookies are, in fact, delicious. Published a paper." | 20,000,000 | 7,800 |
| 8 | Crab Wizard | "Conjures cookies from the aether. Side effects include crumbs." | 330,000,000 | 44,000 |
| 9 | Crab Mothership | "They came from the Crabulon Nebula. They came for cookies." | 5,100,000,000 | 260,000 |
| 10 | Crab God | "SNIP SNIP, MORTAL. YOUR COOKIES ARE MINE." | 75,000,000,000 | 1,600,000 |

---

## 6. Prestige System: "Molting"

- Shell Shards earned: `floor(sqrt(totalCookiesBaked / 1,000,000,000))`
- Each shard gives +10% multiplicative bonus to ALL production
- Resets: cookies, CPS, upgrade counts
- Persists: Shell Shards, achievements, total stats, molt count
- Confirmation: "Are you sure? You'll lose all your crabs but gain X Shell Shards."

---

## 7. Achievement System (20 achievements)

**Click:** First Nibble (1), Clicker Crab (100), Carpal Claw Syndrome (1000), The Snappening (10000)
**Cookies:** Cookie Crumb (100), Cookie Jar (10K), Cookie Monster (1M), Cookie Galaxy (1B), Cookie Singularity (1T)
**Upgrades:** Crab Dad (1 Baby), Army of Darkness (10 Army), ALL HAIL (1 King), Peer Reviewed (1 Scientist), ASCENDED (1 God)
**CPS:** Passive Income (10), Cookie Factory (1000), Cookie Singularity Engine (1M)
**Prestige:** First Molt (1), Serial Molter (5)
**Secret:** Why Are You Still Here (24h played)

---

## 8. UI Layout

Desktop: 2-column (cookie left, shop right). Mobile: single column stacked.
Color palette: navy bg (#1a1a2e), crab red (#e74c3c), cookie gold (#f39c12), green buy (#2ecc71).

---

## 9. Visual Design

- CSS pixel art via box-shadow sprites
- Monospace font throughout
- Cookie click squish animation
- Floating numbers, cookie crumbs, achievement toasts
- Optional scanline CRT effect
- Particle budget: 20 floating numbers, 30 crumbs max

---

## 10. Number Formatting

Suffixes: K, M, B, T, Qa, Qi, Sx, Sp, Oc

---

## 11. Persistence

- localStorage key: "nom_save"
- Auto-save every 30s + beforeunload
- Offline earnings: min(elapsed, 8h) * CPS * 0.5
- Export/import via base64

---

## 12. Flavor Text Rotation

"The crabs whisper. They want more."
"Somewhere, a cookie screams."
"This is what peak performance looks like."
"Your crabs have developed a cookie-based economy."
"SNIP SNIP SNIP SNIP SNIP"
"Fun fact: crabs can't actually eat cookies. Don't tell them."
"A cookie a day keeps the crab doctor away."
"The Crab King demands tribute."
