# AI Coding Agent Instructions for Fantasy Mirror Quest

## Project Overview
**Fantasy Mirror Quest** (ЗЕРКАЛО СЛЕДОПЫТА) is a single-file web-based puzzle game built with vanilla HTML/CSS/JavaScript. Players guide light beams through mirrors to reach a goal, with escalating difficulty levels, XP progression, and monster/chest mechanics.

## Architecture Essentials

### Single File Structure
- **All code in [index.html](index.html)**: HTML markup (lines 1-505), CSS styling (lines 6-495), JavaScript game logic (lines 520-1301)
- No build system, no external frameworks—runs directly in browser
- State stored in `localStorage` for XP persistence (`mirror_quest_xp_v3` key)

### Core Game Systems
1. **Board Grid System**: 2D array `grid[row][col]` with cell types:
   - `START`: Beam origin with rotatable direction (UP/DOWN/LEFT/RIGHT)
   - `END`: Goal (💎) that triggers win condition when beam reaches it
   - `MIRROR`: Rotatable 90° angle reflection (4 rotations via `angle: 0-3`)
   - `WALL`, `EMPTY`, `CHEST`: Obstacles, placement zones, and loot
   
2. **Beam Physics** (`traceBeam()` ~lines 1200-1290):
   - Recursive ray-casting from start position in initial direction
   - Mirror deflection follows double-rotation logic: mirrors with even/odd angles use different reflection rules
   - Beam width pulses with sine wave (`widthVar = Math.sin(beamPulse) * 1.5 + 4`)
   - Kills monsters and opens chests on collision
   - Win triggered when beam reaches END cell

3. **Difficulty Levels** (const `LEVELS` ~line 572):
   - **Easy (Novice)**: 8×8 board, 2000 energy, 1 monster, 2 placeable mirrors
   - **Normal (Scout)**: 10×10 board, 4000 energy, 2 monsters, 4 placeable mirrors
   - **Hard (Ranger)**: 12×12 board, 8000 energy, 3 monsters, 6 placeable mirrors

4. **Level Generation** (`generateLevel()` ~lines 812-860):
   - Creates solution path via `generatePath()` with random turns
   - Generates additional mirrors between `min-max` range
   - Spawns chests and monsters at random valid positions
   - Path can fail and regenerate if too short or hits boundaries

### Key Game Mechanics

| Mechanism | Implementation | Notes |
|-----------|-----------------|-------|
| **Mirror Rotation** | `clickCell()` rotates `cell.angle = (angle + 1) % 4` | Costs energy per move |
| **Mirror Placement** | `placeMirror()` converts EMPTY cells, decrements counter | Limited by difficulty `placeableMirrors` |
| **Scoring** | `score -= LEVELS[curLevelIdx].moveCost` per action | Win adds XP reward |
| **Monster Patrol** | `buildPatrolRoute()` creates randomized back-and-forth paths | `updateMonsters()` animates movement |
| **Hint System** | `toggleHint()` renders solution path in cyan overlay on canvas | Calculates visual path from `solutionPath` array |
| **XP & Ranks** | Stored in localStorage, 10-tier progression system | `RANK_THRESHOLDS` define tier boundaries |

### UI Patterns
- **Language Toggle**: `setLang('ru'/'en')` updates all TEXT[lang] fields
- **Canvas Overlay**: Canvas layer (`beam-canvas`) renders beam animation on top of DOM grid
- **State Screens**: Main menu (`main-menu`), game UI (`game-ui`), rank screen (`rank-screen`)
- **DOM Generation**: Board rendered via `renderBoard()` creating dynamic HTML elements with click handlers

## Common Developer Tasks

### Adding a New Difficulty Level
1. Add entry to `LEVELS` array with config: `{ size, pathLen, startScore, moveCost, monsterCount, chestCount, xpRewardFactor, placeableMirrors, generatedMirrors }`
2. Add translation strings in `TEXT.ru.easy/norm/hard` and `TEXT.en` equivalents
3. Update menu button HTML to call `startGame()` with new index

### Changing Game Balance
- **Energy cost per move**: Edit `moveCost` in relevant LEVELS entry
- **XP progression pace**: Adjust `xpRewardFactor` or `RANK_THRESHOLDS` values
- **Monster difficulty**: Modify `monsterCount` or patrol route logic in `buildPatrolRoute()`

### Debugging Beam Physics
- Beam tracing starts in `traceBeam()` → reads `startPos.dir` and `grid` cells
- Mirror reflection logic at line ~1270: check `cell.angle % 2` to understand which reflection rule applies
- Canvas rendering has shadow effects (`ctx.shadowBlur`, `ctx.shadowColor`)—disable for clarity if needed

### Modifying Visuals
- **Mirror rotation angle**: `mirror[data-angle="N"]` CSS targets (lines 313-316)
- **Beam color/glow**: Edit `ctx.strokeStyle`, `ctx.shadowColor` in `traceBeam()` (line ~1240)
- **Start icon animation**: `.start-icon.beam-active` triggers `startJitter` keyframe
- **Grid cell size**: `CELL_SIZE = 50` (line 562)—adjust board rendering scale

## Critical Patterns to Preserve

1. **Canvas + DOM Hybrid**: Game board uses DOM divs for cells, canvas overlay for animated beam—do NOT merge into pure canvas
2. **Grid-first Design**: All logic references `grid[y][x]` (row-major), not pixel coordinates
3. **Direction Constants**: Always use `DIRS['UP'/'DOWN'/'LEFT'/'RIGHT']` with `.dx, .dy`—circular array `DIR_NAMES` used for rotation
4. **Move Cost Deduction**: Every player action calls `score = Math.max(0, score - LEVELS[...].moveCost)` to prevent negative energy
5. **LocalStorage Persistence**: Only XP stored; level progress and score are session-only

## File Organization Reference
- **HTML Structure** (lines 1-505): Menu, game UI, rank screen divs
- **CSS Themes** (lines 6-495): Dark fantasy aesthetic with animated orbs, gradient backgrounds
- **Game Constants** (lines 548-573): TEXT, RANK_COLORS, LEVELS, DIRS
- **Game State** (lines 620-634): Grid, monsters, chests, solution path
- **UI Functions** (lines 637-680): Language, menu navigation, HUD updates
- **Board Logic** (lines 812-1180): Level generation, pathfinding, grid interactions
- **Rendering** (lines 1158-1195): DOM board and monster/chest element creation
- **Animation Loop** (lines 1030-1060): Monster updates and beam re-rendering via `traceBeam()`

## Testing Checklist
When modifying core mechanics:
- ✓ Verify beam reaches END → triggers win and XP grant
- ✓ Confirm mirror rotation affects beam direction correctly
- ✓ Test monster patrolling and collision detection with beam
- ✓ Validate placeable mirror limit is enforced
- ✓ Check localStorage XP persists across page reloads
- ✓ Ensure UI updates (HUD, rank display) sync with state changes
