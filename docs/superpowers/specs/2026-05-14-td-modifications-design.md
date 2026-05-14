# Tower Defense — 6 Modifications Design Spec

Date: 2026-05-14

## Overview

Six changes to the tower defense game in `tetris.html`:
1. Add single-player mode
2. Revise turret upgrade UI (Upgrade/Sell buttons with costs)
3. Make enemy path more visually distinct
4. Fix Quick Match bug (matching alone starts game)
5. Add 2x speed-up button
6. Add tower loadout (Prepare screen — confirm all 4 cards)

## 1. Single-Player Mode

### Lobby Screen
Add a 3rd button to `screenTDLobby`:

```
🏰 塔防守卫
[🧑 单人模式]     ← NEW (first button)
[👥 好友合作]
[🌐 快速匹配]
```

### Behavior
- "单人模式" → `tdfStartSolo()`:
  - Sets `tdfPlayer=1`, `tdfRoom=null`, `tdfChannel=null`
  - Navigates to `screenTowerDefense`, calls `tdfInit()` + `tdfStartLoop()`
  - No Supabase room created, no broadcast, no channel subscription
- Hides chat row and ready button in solo mode
- Prep phase: wave auto-starts after 15s timer (no "both players ready" requirement)
- Everything else identical to co-op gameplay

## 2. Turret Upgrade UI

### Current
Shows `showConfirm(message, upgradeFn, '取消')` — cost buried in message text, no sell option unless max level.

### New Design
Replace `showConfirm` with an inline **tower action popup** (DOM overlay):

```
┌──────────────────┐
│ 🏹 箭塔 Lv.2     │
│                  │
│ [⬆ 升级 400g]   │
│ [💰 出售 +200g]  │
│ [✕ 取消]        │
└──────────────────┘
```

- **Upgrade button**: shows cost. If unaffordable or max level → grayed out, shows "已满级" or "金币不足"
- **Sell button**: shows refund amount (50% of total spent)
- **Cancel**: closes popup
- Popup positioned centered over the board, z-index above canvas
- Dismiss on tapping outside the popup

### Implementation
- Create a DOM element `#tdfTowerPopup` in the game screen HTML (or dynamically)
- Show/hide with CSS `.active` class
- Populate dynamically when a tower is tapped

## 3. Path Visuals

### Current
Path tiles: `#2a2a1a` color, same size as non-path tiles.

### New Design
- **Path tile color**: `#3a3020` (warm brown) — distinct from non-path `#0a0a2a` (dark blue)
- **Path border**: `ctx.strokeStyle='#5a4a30'`, draw 1px dash pattern on path edges
- **Direction arrows**: On each path cell, draw a small arrow (▶▼◀▲) indicating enemy flow direction
  - Arrow computed from the vector to the next waypoint
  - Drawn in `#888` semi-transparent
- **Entry marker**: Leftmost path cell gets a green glow (`#4f8` border)
- **Exit marker**: Rightmost/last path cell gets a red glow (`#f44` border)

### Implementation
In `tdfDraw()`, after drawing grid cells, for each path cell:
1. Fill with `#3a3020`
2. Draw directional arrow based on path direction
3. Highlight entry/exit cells

## 4. Quick Match Fix

### Bug
`tdfCreateRoom('match')` creates a room then immediately calls `tdfInit()` + `tdfStartLoop()`, starting the game solo before a partner is found. Player enters game alone even though they chose "Quick Match".

### Fix
When `mode === 'match'`:
- Create room in Supabase
- Show a **waiting screen** instead of starting the game
- `tdfPollMatch()` runs in background
- When partner found → start game together
- Cancel button → delete room, go back to lobby

### Waiting Screen
```
🌐 匹配中...
正在寻找队友...
[← 取消匹配]
```

- Shows for up to 60s (timeout → "未找到对手" message, back to lobby)
- Cancel deletes the room via PATCH `status:'cancelled'` or similar

### Code Change
In `tdfCreateRoom`, when `mode==='match'`:
```javascript
if(mode==='match'){
  ss('screenTDWait'); // NEW waiting screen
  tdfPollMatch();       // polls, and on success navigates to game
}
// Do NOT call tdfInit/tdfStartLoop here
```

## 5. Speed-Up Button

### Design
A `⚡ 2×` toggle button in the game screen, next to the status bar:

```
⚡1×  ❤20  💰200  🌊3/20
```

### Behavior
- Tapping toggles between 1× and 2×
- 2× speed: game loop interval changes from 200ms → 100ms
- Button style changes when active (glow/highlight): `.tdf-speed-btn.on{background:#ff0;color:#000;}`
- Disabled during prep phase (speed only affects combat waves)
- Text shows current speed: "⚡1×" / "⚡⚡2×"

### Implementation
```javascript
var tdfSpeed=1;
function tdfToggleSpeed(){
  if(tdfPrepPhase)return;
  tdfSpeed=tdfSpeed===1?2:1;
  if(tdfLoopId){clearInterval(tdfLoopId);tdfLoopId=setInterval(tdfTick,200/tdfSpeed);}
  // Update button text
}
```

## 6. Loadout System

### Prepare Button
Add to `screenTowerDefenseDiff`, between diff cards and LB button:

```html
<button class="menu-btn" id="btnTDPrepare" style="margin-top:16px;background:linear-gradient(135deg,#2a2a1e,#1a1a0e);border-color:#4af;font-size:16px;padding:10px 40px;">🎒 确认卡组</button>
```

### Loadout Screen (`screenTDLoadout`)
```
🎒 确认卡组
┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│ 🏹   │ │ 💣   │ │ ❄️   │ │ 💰   │
│ 箭塔  │ │ 炮塔  │ │ 冰塔  │ │ 金矿  │
│ 50g  │ │ 100g  │ │ 75g  │ │ 150g  │
│ ✅    │ │ ✅    │ │ ✅    │ │ ✅    │
└──────┘ └──────┘ └──────┘ └──────┘
      [✔ 确认完成]
[← 返回]
```

- All 4 tower cards shown, all pre-selected (✅ checkmark)
- Tap to toggle selection — but **minimum 4 must be selected** (all required per user feedback)
- Card shows: emoji icon, name, cost
- "确认完成" saves to localStorage key `goblin_tdf_loadout` as JSON array `["arrow","cannon","ice","mine"]`
- Future tower types: pick N from pool, save selected set

### In-Game Integration
- Tower palette in `screenTowerDefense` reads from `goblin_tdf_loadout`
- Only selected tower types appear in palette buttons
- If no loadout saved, default to all 4 towers

### localStorage Key
- `goblin_tdf_loadout` — JSON array `["arrow","cannon","ice","mine"]`

## HTML Changes Summary
1. `screenTDLobby` — add "单人模式" button
2. `screenTowerDefense` — add speed button next to status bar, add `#tdfTowerPopup` overlay
3. `screenTowerDefenseDiff` — add "🎒 确认卡组" button
4. New `screenTDWait` — matching wait screen
5. New `screenTDLoadout` — loadout selection screen

## JS Changes Summary
1. `tdfStartSolo()` — single player init (no room/channel)
2. `tdfToggleSpeed()` — 1x/2x toggle
3. Tower popup show/hide logic — replaces showConfirm for tower interaction
4. `tdfCreateRoom` fix — don't start game on match mode
5. Loadout save/load functions
6. Path rendering enhancement in `tdfDraw()`
7. Hide chat/ready in solo mode
