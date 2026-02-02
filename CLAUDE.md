# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is "阿嬤逃脫 2D" (Grandma Escape 2D) - a single-file HTML5 canvas game. The entire game is contained in `grandma_escape.html` with embedded CSS and JavaScript. No build tools or dependencies are required.

## Running the Game

Open `grandma_escape.html` directly in a web browser. No server required.

## Architecture

The game uses a standard game loop pattern with these key components:

- **CONFIG object** (line ~296): Game constants (speeds, ratios, colors)
- **state object** (line ~313): All mutable game state (player position, HP, score, objects, etc.)
- **Game Loop**: `loop()` → `update(dt)` → `render()` at 60fps via requestAnimationFrame
- **Input**: Touch swipes and keyboard (WASD/arrows) handled in `initInput()`
- **Audio**: Web Audio API synth sounds in `audio.play()`

### Game Mechanics

- 3-lane endless runner (top-down view)
- Player actions: lane change (left/right), jump (dodge low obstacles), slide (dodge high obstacles)
- Hazard types: `hazard_low` (jump over), `hazard_high` (slide under), `hazard_block` (lane change)
- Powerups: invincibility (star), health (pill), shield
- Chaser (nurse) appears when player takes damage

### Key Functions

- `startGame()`: Initializes/resets game state
- `spawnObject()`: Creates obstacles and collectibles
- `handleCollision()`: Processes player-object interactions
- `render()`: Canvas drawing with emoji sprites
