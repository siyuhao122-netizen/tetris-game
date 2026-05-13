# Co-op Tower Defense — Design Spec

Date: 2026-05-14

## Overview

Add a 7th game to Goblin Game Land: a **2-player co-op tower defense**. Players share a grid, gold pool, and HP — placing and upgrading towers to defend against waves of goblin enemies. Supports friend co-op (room codes + invites) and quick match (Supabase matchmaking). Reuses existing lobby/room/broadcast infrastructure.

## Game Mechanics

### Grid & Path
- **14 columns × 8 rows** grid, cell-based tower placement
- Enemies follow a **predefined path** from left edge to right edge
- Towers can only be placed on **non-path tiles**
- 3 path layouts per difficulty:
  - Easy: simple S-curve
  - Normal: winding with cross-backs
  - Hard: long maze-like route

### Difficulties

| Difficulty | Waves | Tower Max Level | Starting HP | Starting Gold |
|------------|-------|-----------------|-------------|---------------|
| Easy | 20 | 5 | 25 | 200g |
| Normal | 30 | 4 | 20 | 150g |
| Hard | 40 | 3 | 15 | 100g |
| Endless | ∞ | **No cap** | 20 | 150g |

### Towers

| Tower | Base Cost | Damage | Range | Attack Speed | Special |
|-------|-----------|--------|-------|-------------|---------|
| 🏹 Arrow | 50g | Low | 3 cells | Fast (0.5s) | — |
| 💣 Cannon | 100g | High | 2 cells | Slow (1.5s) | Splash AOE (1-cell radius) |
| ❄️ Ice | 75g | None | 2.5 cells | Medium (1s) | Slows enemies 50% for 1.5s |
| 💰 Gold Mine | 150g | None | N/A | N/A | Generates +10g per wave |

### Upgrade System
- Level N → N+1 cost: `baseCost × 2^N`
- Each upgrade: damage +30%, range +5%, attack cooldown -10%
- Ice: slow effect +5% per level
- Gold Mine: +5g/wave per level
- **Endless mode**: no level cap; costs continue `baseCost × 2^N` indefinitely
- Sell tower: 50% refund of total gold spent

### Enemies

| Type | HP | Speed | Bounty | Special |
|------|-----|-------|--------|---------|
| 🟢 Goblin Scout | 30 | 2 cells/s | 10g | Basic |
| 🟡 Goblin Warrior | 80 | 1.2 cells/s | 20g | Standard |
| 🔴 Goblin Tank | 200 | 0.6 cells/s | 40g | Takes 2x damage from Cannon |
| 🟣 Goblin Healer | 100 | 1 cells/s | 25g | Heals nearby enemies +5HP/s |
| 👑 Goblin King | 500 | 0.5 cells/s | 200g | Boss — every 5th wave |

### Enemy Scaling (Endless Mode only)
Waves past the base difficulty cap (20/30/40) use scaled stats:
- HP scales: `baseHP × (1.15 ^ (wave - baseWaves))`
- Speed scales: `baseSpeed × (1.05 ^ (wave - baseWaves))`, capped at 3× base
- Boss HP: `baseHP × (1.25 ^ bossNumber)` (bosses always scale regardless of wave)
- Gold bounties scale proportionally with HP
- Prep phase eliminated after wave 10 — waves spawn continuously

### Waves
- Prep phase: 15s between waves (both players can build)
- Wave starts when **both players press Ready** or timer expires
- After wave 10 in endless: **no prep phase** — continuous spawn
- Enemy count per wave: `3 + wave × 2`

### Win/Lose
- **Win**: Survive all waves with HP ≥ 1
- **Lose**: HP reaches 0 (each escaped enemy deals 1 damage)
- Shared HP pool — both players lose together
- Boss escape deals 5 damage

## Multiplayer Design

### Room System
- Reuses existing `rooms` table: `game:'towerdefense'`
- Player 1 creates room → 6-char code → Player 2 joins
- Both modes: **friend invite** + **quick match** (same as Gomoku/Tetris battle)
- Matchmaking polls `rooms` where `status='matching'`

### Co-op Rules
- **Shared gold**: both players spend from same pool
- **Shared HP**: one HP bar for both
- **Tower ownership**: first to place on a cell owns it; both can upgrade any tower
- **Ready system**: both must Ready for wave to start early
- **Disconnect**: surviving player continues solo with full control

### Real-time Sync
- Supabase Realtime broadcast channel per room
- Events: `place_tower`, `upgrade_tower`, `sell_tower`, `player_ready`, `chat_quick`
- Late-join: full board state snapshot sent on channel join
- No polling — event-driven sync

### Quick Chat
4 preset buttons: "Build here!", "Save for upgrade!", "Ready!", "Help!"

## UI Layout

### Screen Flow
```
screenSelect (new card) → screenTowerDefenseDiff → screenTDLobby → screenTowerDefense
```

### Game Select Card
- Emoji: 🏰, Name: "塔防守卫", Desc: "双人合作塔防"
- Accent: `border-top: 3px solid #f44`
- Grid becomes 2 rows × 4 columns (7 games, 1 empty slot)

### In-Game Layout (mobile-optimized)
```
┌──────────────────────────┐
│ ← 退出   ❤20  💰200  🌊3/20│
├──────────────────────────┤
│     GAME BOARD           │
│     14×8 grid            │
├──────────────────────────┤
│ 🏹50 💣100 ❄️75 💰150  │ ← tower palette
├──────────────────────────┤
│ [Ready] [Upgrade Selected]│
│ 💬 Build here! | Save!   │ ← quick chat (2 rows)
└──────────────────────────┘
```

### Tower Placement UX
1. Tap tower icon in palette → it highlights (selected)
2. Tap empty non-path grid cell → place tower (deduct gold)
3. Tap existing tower → popup: "Upgrade (cost)" / "Sell (refund)"
4. Selected tower stays active for rapid multi-placement

## Technical Architecture

### Variable Namespace
All variables prefixed with `tdf` (tower defense):
```javascript
var tdfGrid, tdfPath, tdfEnemies, tdfTowers, tdfHP, tdfGold;
var tdfWave, tdfWaveMax, tdfPrepPhase, tdfReady, tdfSelected;
var tdfScore, tdfStarted, tdfGameOver, tdfPause;
var tdfRoom, tdfPlayer, tdfChannel;
var tdfCanvas, tdfCtx;
```

### Difficulty Config
```javascript
var TDF = {
  easy:   {waves:20, maxLevel:5,  hp:25, gold:200, path:'s_curve',  label:'简单', e:'🟢'},
  normal: {waves:30, maxLevel:4,  hp:20, gold:150, path:'winding',  label:'普通', e:'🟡'},
  hard:   {waves:40, maxLevel:3,  hp:15, gold:100, path:'maze',     label:'困难', e:'🔴'},
  endless:{waves:Infinity, maxLevel:Infinity, hp:20, gold:150, path:'winding', label:'无尽', e:'♾️'}
};
```

### Game Loop (200ms interval)
```
tdfTick():
  1. Move enemies along path waypoints
  2. Check tower attack cooldowns → damage enemies in range
  3. Remove dead enemies → add gold
  4. Check escaped enemies → reduce HP
  5. Check win/lose
  6. Spawn wave if prep timer expired + both ready
  7. tdfDraw()
```

### Path System
- Predefined waypoint arrays per difficulty, stored as `[{x,y},...]`
- Enemies interpolate between waypoints at their speed
- No dynamic pathfinding needed

### Canvas Rendering
- Board: 14×8 grid, cell size ~28px → 392×224 base, HiDPI scaled
- Layers: path tiles → placed towers (colored squares + emoji) → enemies (colored circles)
- Tower palette: DOM buttons (not canvas) for easy touch targets
- Status bar: DOM elements for HP, gold, wave counter

## Shared System Integration

### Leaderboard
- `_showGameLB('towerdefense', 'screenTowerDefenseDiff')`
- Columns: 排名, 玩家, 波数, 分数, 时间
- Sort by `waves DESC, score DESC`
- Add `'towerdefense'` entry to `_showGameLB` title/diffs/labels maps

### Stats
- `recordGame("towerdefense", diff, tdfScore, tdfTimer, won)`
- `_submitLB("towerdefense", diff, tdfScore, tdfTimer, callback, {waves: tdfWave})`

### Achievements
- `towerdefense_wave10` — survive 10 waves
- `towerdefense_wave20` — survive 20 waves
- `towerdefense_noleak` — win without losing any HP
- `towerdefense_rich` — accumulate 1000g at once

### Audio
- Tower shoot: sharp tone per tower type
- Enemy death: low thud
- Wave start: ascending notes
- Game over: standard `_playGameOver()`
- Boss spawn: dramatic two-tone

### Others
- VIP multiplier on score
- Pause overlay (same pattern as other games)
- Confirm dialog for selling towers / exiting

## File Changes (tetris.html)

1. **CSS**: Tower palette buttons, grid cell states, tower level badge (~20 lines)
2. **HTML Body**: 3 new screens + 1 game select card (~50 lines)
3. **HTML Body**: Update game select grid to 2×4
4. **Script**: Tower defense game block (~400 lines)
5. **Script**: Button wiring + game-over overlay support (~30 lines)
6. **Script**: Add `'towerdefense'` to `_showGameLB` maps (~5 lines)

## Out of Scope
- Single-player AI companion mode
- Tower targeting priority (e.g., "attack strongest first")
- Selling towers during combat (only during prep phase)
- Path editing / maze building by players
