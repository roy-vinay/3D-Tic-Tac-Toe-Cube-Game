# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

Open `3D-tic-tac-toe-cube-game.html` directly in any modern browser — no build step or dependencies to install. Three.js is loaded via an ES-module importmap from jsDelivr CDN (`three@0.160.0`).

To serve locally (needed for the preview panel):
```
npx serve -p 5500 .
```
The `.claude/launch.json` already defines this server for the Claude Code preview panel.

## Architecture

Everything lives in a single HTML file with inline CSS and a `<script type="module">` block. There is no bundler, no framework, and no separate JS files.

### Three.js scene hierarchy

```
scene
└── worldGroup        ← rotates on pointer drag (view-only, never modify for game logic)
    └── cubeGroup     ← the cube's intrinsic coordinate system (never rotated)
        └── cubies[]  ← 27 BoxGeometry meshes at positions {-1,0,1}³ × CUBIE_SPACING
            └── stickers[]  ← PlaneGeometry children of cubies (54 total, outward faces only)
```

**Critical invariant:** `cubeGroup` is never rotated or translated — its local space IS the cube's coordinate system. All game logic (move layer selection, win detection) uses positions and quaternions relative to `cubeGroup`, not world space. Using world-space coordinates breaks when `worldGroup` is rotated.

### Move execution

`MOVES` maps move names (`U`, `U'`, `R`, …) to `{ axis, layer, dir }` where layer is `+1` or `-1` in cubeGroup space. Executing a move:
1. Selects the 9 cubies where `round(position[axis] / CUBIE_SPACING) === layer`
2. Creates a temporary `pivot` Group, `attach()`es those cubies to it (preserves world transform)
3. Animates `pivot.rotation[axis]` with ease-in-out over `ANIM_DURATION` ms
4. Bakes: re-`attach()`es each cubie back to `cubeGroup`, snaps position to grid, quantizes quaternion to kill float drift

The module-level `busy` flag blocks all input during animation. Everything that checks whether a move is allowed must check `busy`.

### Win detection

`getStickerFaceAndGrid(sticker)` computes which face a sticker belongs to and its (row, col) within that face using:
- `cubie.quaternion × sticker.quaternion` → sticker's outward normal in cubeGroup space (NOT world space)
- `cubie.position` → grid coordinates `{-1,0,1}`

`checkWin()` rebuilds 6 face grids from all 54 stickers on each call. It is called only in `afterMove()` — never at placement time. This is intentional: a line formed by placement but destroyed by the mandatory move must not win.

### Turn state machine

```
phase: 'place' → player clicks sticker → placeMark() → phase: 'move'
phase: 'move'  → player clicks button or drag-twists → doMove() → executeMoveByName()
                 → afterMove() → checkWin() → endGame() or swap player → phase: 'place'
```

`gameState` holds `{ mode, current, phase, busy, gameOver }`. The `busy` module-level variable (separate from `gameState.busy`) is the authoritative animation lock.

### Input disambiguation

One `pointerdown` handler on the canvas resolves into three cases based on hit-test and drag distance (`DRAG_THRESH = 6px`):
- **No sticker hit OR drag from sticker in 'place' phase** → view rotation (`worldGroup`)
- **Sticker tap in 'place' phase** → `placeMark()`
- **Sticker drag in 'move' phase** → `commitTwist()` (projects screen-space drag onto face plane to pick axis/direction)

### Textures

Three shared `CanvasTexture` instances (`TEX.blank`, `TEX.X`, `TEX.O`) are created once at load. Stickers use `MeshBasicMaterial` (unlit) so they stay bright white regardless of lighting. Changing a sticker's mark swaps `material.map` and sets `material.needsUpdate = true`.

### Layer highlight

`highlightLayer(moveName)` tints affected stickers by setting `material.color` to `0xffcc33`. `clearHighlight()` resets to `0xffffff`. Wired to `mouseenter`/`mouseleave` on move buttons (only fires when buttons are not disabled).

### AI

`aiTurn()` runs when `mode === 'ai'` and `current === 'O'`. Placement strategy: win if possible → block X from winning → random. Move strategy: random (cloning 3D cube state for move simulation is not implemented).

## Current state

Feature-complete single-file game. Known gaps worth addressing:
- AI cube move is random — it doesn't simulate outcomes before picking a move, because cloning the 3D geometry state cheaply isn't implemented
- Drag-to-twist direction heuristic (`commitTwist`) can misfire on edge/corner stickers at oblique view angles; the button panel is the reliable fallback
- No touch-specific tuning — works via Pointer Events but untested on mobile
- The ⟳/⟲ Unicode rotation symbols in the move buttons render inconsistently across OS/font stacks; the color coding (blue = CW, orange = CCW) and hover tooltip are the primary direction cues
