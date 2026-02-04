# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is "阿嬤逃脫 2D" (Grandma Escape 2D) - a single-file HTML5 canvas game built entirely in `index.html` with embedded CSS and JavaScript. No build tools, bundlers, or dependencies are required.

## Running the Game

Open `index.html` directly in a web browser. No server required.

**Controls:**
- **Mobile:** Touch swipes (left/right to change lanes, up to jump, down to slide)
- **Desktop:** WASD or Arrow keys

## File Structure

```
grandma_escape/
├── index.html      # Complete game (HTML + CSS + JS)
├── CLAUDE.md       # This file
└── .gitignore      # Excludes settings.local.json
```

## Architecture

The game uses a standard game loop pattern with all code contained in a single `<script>` block within `index.html`.

### Core Components (with line references)

| Component | Line | Description |
|-----------|------|-------------|
| **CSS Variables** | ~10 | Color theming (--primary, --secondary, --accent, etc.) |
| **CONFIG object** | ~397 | Game constants (lanes, speeds, colors, spawn intervals) |
| **state object** | ~414 | All mutable game state (player, objects, particles, etc.) |
| **ASSETS object** | ~462 | Visual definitions for sprites and obstacles |
| **input object** | ~474 | Touch/keyboard input tracking |

### Key Functions

| Function | Line | Purpose |
|----------|------|---------|
| `initInput()` | ~479 | Sets up touch and keyboard event listeners |
| `audio.init()` / `audio.play()` | ~514-553 | Web Audio API synth sound effects |
| `startGame()` | ~559 | Initializes/resets all game state |
| `move(dir)` | ~610 | Lane change (-1 = left, +1 = right) |
| `jump()` | ~616 | Initiates jump action |
| `slide()` | ~629 | Initiates slide/duck action |
| `spawnObject()` | ~636 | Creates obstacles and collectibles |
| `update(dt)` | ~722 | Main update tick (physics, collisions, spawning) |
| `handleCollision(obj)` | ~819 | Processes player-object interactions |
| `render()` | ~896 | Canvas drawing with emoji sprites |
| `loop(time)` | ~1048 | Main game loop via requestAnimationFrame |
| `gameOver()` | ~1108 | End game and show results screen |
| `resize()` | ~1123 | Handle window resize events |

### Game Loop Flow

```
requestAnimationFrame
       ↓
    loop(time)
       ↓
   update(dt)  →  Progression, Physics, Spawning, Collisions
       ↓
    render()   →  Background, Objects, Player, Chaser, Particles
```

## Game Mechanics

### Gameplay
- 3-lane endless runner (top-down perspective)
- Player is fixed at 80% down the screen (`playerYRatio: 0.8`)
- Speed increases over time from `baseSpeed` (400) to `maxSpeed` (1000)
- HP system with slow regeneration

### Player Actions
| Action | Input | Effect |
|--------|-------|--------|
| Lane Change | Left/Right swipe or A/D or ←/→ | Move between 3 lanes |
| Jump | Swipe up or W or ↑ | Dodge low obstacles |
| Slide | Swipe down or S or ↓ | Dodge high obstacles |

### Hazard Types
| Category | Dodge Method | Examples |
|----------|--------------|----------|
| `hazard_low` | Jump | 🧹 💧 🚕 |
| `hazard_high` | Slide | 👙 ⚡ 🚧 |
| `hazard_block` | Lane change | 🎍 🦽 ⛔ |

### Powerups
| Item | Effect |
|------|--------|
| ⭐ Star | 5 seconds invincibility |
| 💊 Pill | +30 HP |
| 🛡️ Shield | Blocks one hit |

### Stages (cycle every 200m)
| Stage | Index | Background | Theme |
|-------|-------|------------|-------|
| Hallway | 0 | #2c3e50 | Indoor |
| Garden | 1 | #27ae60 | Outdoor |
| Street | 2 | #34495e | Urban |

### Chaser System
- Nurse (👩‍⚕️) appears behind player when damage is taken
- Chases for 5 seconds, then retreats off-screen
- Visual pressure mechanic (no direct gameplay effect currently)

## Code Conventions

### State Management
- All mutable game state lives in the `state` object
- Configuration constants live in the `CONFIG` object
- No external state management library

### Rendering
- Uses HTML5 Canvas 2D context
- Emoji-based sprites for all game objects
- Shadow ellipses drawn beneath objects for depth
- Player has smooth X-position lerping: `state.renderX += (targetX - state.renderX) * 0.2`

### Audio
- Web Audio API with programmatic synth sounds
- No external audio files
- Sound types: `coin`, `jump`, `hit`

### Collision Detection
- Simple lane-based collision (same lane + Y intersection)
- Hitbox adjustment for sliding state
- Objects have `active` flag to prevent multiple collision triggers

### CSS
- CSS custom properties for theming
- Responsive design with max-width container
- iOS safe area support via `env(safe-area-inset-top)`

## Modification Guidelines

### Adding a New Obstacle Type
1. Add emoji variants to `spawnObject()` around line ~669
2. Create new category if needed (e.g., `hazard_new`)
3. Add dodge logic in `handleCollision()` around line ~848

### Adding a New Powerup
1. Add to powerup spawn logic in `spawnObject()` around line ~698
2. Add buff handling in `handleCollision()` around line ~839
3. Add state tracking in `state` object
4. Add visual effect in `render()` if needed (around line ~1003)

### Adjusting Difficulty
- `CONFIG.baseSpeed` / `CONFIG.maxSpeed` - Movement speed
- `CONFIG.spawnInterval` - Base spawn rate
- `hazardChance` calculation in `spawnObject()` - Obstacle frequency
- HP damage values in `handleCollision()` - Currently 35 damage per hit

### Adding New Sound Effects
Add a new case in `audio.play()` around line ~531:
```javascript
else if (type === 'newSound') {
    osc.type = 'sine';
    osc.frequency.setValueAtTime(440, now);
    // ... configure envelope
}
```

## Known Patterns

### Double-Start Prevention
```javascript
if (state.running) return; // Prevent double start race condition
```

### Delta Time Clamping
```javascript
if (!state.paused && dt < 0.5) {
    update(Math.min(dt, 0.1));
}
```
Prevents physics explosions when returning from tab switch.

### Object Garbage Collection
Objects are marked with `garbage: true` and filtered out each frame:
```javascript
if (obj.y > state.screenHeight + 100) obj.garbage = true;
state.objects = state.objects.filter(o => !o.garbage);
```

## Testing

No automated tests. Manual testing workflow:
1. Open `index.html` in browser
2. Use browser DevTools console for debugging
3. Uncomment debug drawing (line ~951) to visualize hitboxes:
   ```javascript
   ctx.strokeStyle = 'red';
   ctx.strokeRect(cx - 30, cy - 30, 60, 60);
   ```
