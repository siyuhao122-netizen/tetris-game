# Tower Defense 6 Modifications — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add single-player mode, turret upgrade popup, path visuals, fix quick match, speed-up button, and loadout system to tower defense.

**Architecture:** All changes in `tetris.html`. HTML additions: 2 new screens (wait, loadout), 2 buttons (solo, prepare), 1 popup overlay, 1 speed toggle. JS additions: ~6 new functions, modifications to tdfDraw, tdfCreateRoom, input handlers.

**Tech Stack:** Vanilla HTML/CSS/JS, localStorage, Supabase REST API

---

### Task 1: Single-Player Mode + Quick Match Fix

**Files:**
- Modify: `D:\app\tetris.html` (HTML lobby, JS tdfCreateRoom, input handlers)

- [ ] **Step 1: Add solo mode button to lobby HTML**

Find `screenTDLobby`. Add a new button as the FIRST button after the title:

```html
<button class="menu-btn" id="btnTDSolo" style="margin-bottom:16px;">🧑 单人模式</button>
```

Place it before `btnTDFriendBattle`.

- [ ] **Step 2: Add waiting screen HTML**

After `screenTDLobby` closing `</div>`, add:

```html
<!-- TOWER DEFENSE MATCHING WAIT -->
<div class="screen" id="screenTDWait">
  <div style="width:100%;text-align:left;margin-bottom:8px;"><button class="back-btn" id="btnBackTDWait">← 取消匹配</button></div>
  <div class="select-title">🌐 匹配中...</div>
  <div class="o-detail" style="font-size:14px;color:#aaa;">正在寻找队友</div>
</div>
```

- [ ] **Step 3: Add CSS for waiting screen spinner** (optional — just the screen is fine)

- [ ] **Step 4: Add tdfStartSolo() function**

Find the tower defense JS section. After `tdfCreateRoom`, add:

```javascript
function tdfStartSolo(){
  tdfRoom=null;tdfPlayer=1;tdfChannel=null;
  document.getElementById('tdfRoomCode').textContent='单人模式';
  document.getElementById('tdfChatRow').style.display='none';
  document.getElementById('tdfReadyBtn').style.display='none';
  ss('screenTowerDefense');tdfInit();tdfStartLoop();
}
```

- [ ] **Step 5: Fix tdfCreateRoom for quick match**

Find `function tdfCreateRoom(mode){`. Replace the function body so that match mode does NOT start the game immediately:

```javascript
function tdfCreateRoom(mode){
  var code=_bGenCode();
  var status=mode==='friend'?'waiting':'matching';
  _sbReq('POST','/rest/v1/rooms',{code:code,status:status,game:'towerdefense',player1_id:_userId?parseInt(_userId):null}).then(function(r){return r.json();}).then(function(d){
    tdfRoom=d&&d.length?d[0]:null;tdfPlayer=1;
    document.getElementById('tdfRoomCode').textContent='房间: '+code;
    document.getElementById('tdfChatRow').style.display='';
    document.getElementById('tdfReadyBtn').style.display='';
    if(mode==='match'){
      ss('screenTDWait');
      tdfPollMatch();
    }else{
      tdfSetupChannel();
      ss('screenTowerDefense');tdfInit();tdfStartLoop();
    }
  });
}
```

- [ ] **Step 6: Fix tdfPollMatch to start game on match**

Find `function tdfPollMatch(){`. The match found handler needs to set up channel and start the game (not just init):

```javascript
function tdfPollMatch(){
  var poll=setInterval(function(){
    _sbReq('GET','/rest/v1/rooms?game=eq.towerdefense&status=eq.matching&player1_id=neq.'+_userId+'&select=*&limit=1').then(function(r){return r.json();}).then(function(d){
      if(d&&d.length>0){
        _sbReq('PATCH','/rest/v1/rooms?id=eq.'+d[0].id,{status:'playing',player2_id:parseInt(_userId)}).then(function(){
          tdfRoom=d[0];tdfPlayer=2;clearInterval(poll);
          document.getElementById('tdfRoomCode').textContent='房间: '+d[0].code;
          document.getElementById('tdfChatRow').style.display='';
          document.getElementById('tdfReadyBtn').style.display='';
          tdfSetupChannel();
          ss('screenTowerDefense');tdfInit();tdfStartLoop();
        });
      }
    });
  },2000);
}
```

- [ ] **Step 7: Wire new button onclick handlers**

Add these onclick bindings in the script block near other tower defense button wiring:

```javascript
document.getElementById('btnTDSolo').onclick=function(){_initAudio();tdfStartSolo();};
document.getElementById('btnBackTDWait').onclick=function(){
  if(tdfRoom){_sbReq('PATCH','/rest/v1/rooms?id=eq.'+tdfRoom.id,{status:'cancelled'});}
  ss('screenTDLobby');
};
```

- [ ] **Step 8: Commit**

```bash
git add tetris.html
git commit -m "feat: add single-player mode and fix quick match waiting screen"
```

---

### Task 2: Speed-Up Button

**Files:**
- Modify: `D:\app\tetris.html` (HTML status bar, JS, CSS)

- [ ] **Step 9: Add speed button HTML**

Find the tdf-status div in `screenTowerDefense`. Add the speed button before the heart:

```html
<div class="tdf-status" id="tdfStatusBar">
  <button class="tdf-speed-btn" id="tdfSpeedBtn" style="background:#1a1a2e;border:1px solid #444;border-radius:6px;color:#aaa;font-size:13px;padding:2px 8px;cursor:pointer;">⚡1×</button>
  ❤<span id="tdfHP">20</span> 💰<span id="tdfGold">200</span> 🌊<span id="tdfWave">0</span>/<span id="tdfWaveMax">20</span>
</div>
```

- [ ] **Step 10: Add tdfToggleSpeed function and global**

In the tower defense JS, after `var tdfScore=0,tdfStarted=0,...` line, add:

```javascript
var tdfSpeed=1;
```

Then add the toggle function after `tdfStartSolo()`:

```javascript
function tdfToggleSpeed(){
  if(tdfPrepPhase||tdfGameOver||!tdfStarted)return;
  tdfSpeed=tdfSpeed===1?2:1;
  var btn=document.getElementById('tdfSpeedBtn');
  if(btn){btn.textContent=tdfSpeed===2?'⚡⚡2×':'⚡1×';btn.style.color=tdfSpeed===2?'#ff0':'#aaa';}
  if(tdfLoopId){clearInterval(tdfLoopId);tdfLoopId=setInterval(tdfTick,200/tdfSpeed);}
}
```

- [ ] **Step 11: Wire speed button onclick**

```javascript
document.getElementById('tdfSpeedBtn').onclick=tdfToggleSpeed;
```

- [ ] **Step 12: Reset speed in tdfInit**

In `tdfInit()`, add `tdfSpeed=1;` to the reset block. Also reset button text:

```javascript
tdfSpeed=1;
var btn=document.getElementById('tdfSpeedBtn');if(btn){btn.textContent='⚡1×';btn.style.color='#aaa';}
```

- [ ] **Step 13: Commit**

```bash
git add tetris.html
git commit -m "feat: add 2x speed-up toggle button"
```

---

### Task 3: Turret Upgrade Popup UI

**Files:**
- Modify: `D:\app\tetris.html` (HTML, CSS, JS input handling)

- [ ] **Step 14: Add popup overlay HTML**

Inside `screenTowerDefense`, inside `.board-wrap`, after `tdfPauseOverlay`, add:

```html
<div id="tdfTowerPopup" style="display:none;position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);background:#1a1a2e;border:2px solid #444;border-radius:12px;padding:16px 20px;z-index:15;text-align:center;min-width:140px;">
  <div id="tdfPopupTitle" style="font-size:16px;font-weight:bold;margin-bottom:8px;color:#fff;"></div>
  <button class="o-btn" id="tdfPopupUpgrade" style="display:block;width:100%;margin-bottom:6px;font-size:14px;padding:8px;">⬆ 升级</button>
  <button class="o-btn secondary" id="tdfPopupSell" style="display:block;width:100%;margin-bottom:6px;font-size:14px;padding:8px;">💰 出售</button>
  <button class="o-btn secondary" id="tdfPopupCancel" style="display:block;width:100%;font-size:14px;padding:8px;">✕ 取消</button>
</div>
```

- [ ] **Step 15: Add popup show/hide functions**

In tower defense JS, add:

```javascript
var _tdfPopupTower=null;
function tdfShowTowerPopup(tw){
  _tdfPopupTower=tw;
  var cfg=TD_TOWERS[tw.type];
  document.getElementById('tdfPopupTitle').textContent=cfg.icon+' '+cfg.label+' Lv.'+tw.level;
  var upBtn=document.getElementById('tdfPopupUpgrade');
  var canUp=(tdfMaxLevel===Infinity||tw.level<tdfMaxLevel);
  var cost=tdfUpgradeCost(tw);
  if(canUp&&tdfGold>=cost){
    upBtn.textContent='⬆ 升级 '+cost+'g';upBtn.style.opacity='1';upBtn.onclick=function(){tdfUpgradeTower(tw.x,tw.y,true);tdfHideTowerPopup();tdfDraw();};
  }else if(!canUp){
    upBtn.textContent='已满级';upBtn.style.opacity='0.4';upBtn.onclick=null;
  }else{
    upBtn.textContent='⬆ 升级 '+cost+'g (金币不足)';upBtn.style.opacity='0.4';upBtn.onclick=null;
  }
  document.getElementById('tdfPopupSell').textContent='💰 出售 +'+Math.floor(tdfTotalSpent(tw)*0.5)+'g';
  document.getElementById('tdfPopupSell').onclick=function(){tdfSellTower(tw.x,tw.y,true);tdfHideTowerPopup();tdfDraw();};
  document.getElementById('tdfTowerPopup').style.display='block';
}
function tdfHideTowerPopup(){
  document.getElementById('tdfTowerPopup').style.display='none';_tdfPopupTower=null;
}
```

- [ ] **Step 16: Modify board click handler to use popup**

Find the board click handler (the IIFE with `_tdfBound`). Replace the tower tap logic:

```javascript
if(tw){
  tdfShowTowerPopup(tw);
}else if(tdfSelected){
  if(tdfPlaceTower(cell.x,cell.y,tdfSelected,1,true))tdfDraw();
}
```

- [ ] **Step 17: Wire popup cancel and outside-click dismiss**

```javascript
document.getElementById('tdfPopupCancel').onclick=tdfHideTowerPopup;
document.getElementById('tdfBoard').addEventListener('pointerdown',function(e){
  if(_tdfPopupTower&&e.target===document.getElementById('tdfBoard')){tdfHideTowerPopup();}
});
```

- [ ] **Step 18: Commit**

```bash
git add tetris.html
git commit -m "feat: replace showConfirm with tower action popup showing costs"
```

---

### Task 4: Path Visual Enhancement

**Files:**
- Modify: `D:\app\tetris.html` (tdfDraw function)

- [ ] **Step 19: Enhance tdfDraw path rendering**

Find `function tdfDraw(){`. Replace the grid cell drawing section with enhanced path visuals.

The current code draws path tiles with `#2a2a1a` and non-path with `#0a0a2a`. Replace with:

```javascript
// Draw grid cells with enhanced path visuals
for(var r=0;r<TDF_ROWS;r++){
  for(var c=0;c<TDF_COLS;c++){
    var isPath=tdfPathSet[c+','+r];
    if(isPath){
      tdfCtx.fillStyle='#3a3020';
      tdfCtx.fillRect(c*TDF_CELL+1,r*TDF_CELL+1,TDF_CELL-2,TDF_CELL-2);
      // Dashed border on path edges
      tdfCtx.strokeStyle='#5a4a30';tdfCtx.lineWidth=0.5;
      tdfCtx.setLineDash([2,2]);
      tdfCtx.strokeRect(c*TDF_CELL+1.5,r*TDF_CELL+1.5,TDF_CELL-3,TDF_CELL-3);
      tdfCtx.setLineDash([]);
      // Direction arrow
      var arrow='';
      if(r%2===0){arrow=c<TDF_COLS-1?'▶':'▶';}
      else{arrow=c>0?'◀':'◀';}
      tdfCtx.fillStyle='rgba(136,136,136,0.4)';tdfCtx.font=(TDF_CELL/2)+'px sans-serif';
      tdfCtx.textAlign='center';tdfCtx.textBaseline='middle';
      tdfCtx.fillText(arrow,c*TDF_CELL+TDF_CELL/2,r*TDF_CELL+TDF_CELL/2);
    }else{
      tdfCtx.fillStyle='#0a0a2a';
      tdfCtx.fillRect(c*TDF_CELL+1,r*TDF_CELL+1,TDF_CELL-2,TDF_CELL-2);
    }
  }
}
// Entry marker (first path cell)
var firstCell=tdfPath[0];
tdfCtx.strokeStyle='#4f8';tdfCtx.lineWidth=2;
tdfCtx.strokeRect(firstCell.x*TDF_CELL+1,firstCell.y*TDF_CELL+1,TDF_CELL-2,TDF_CELL-2);
// Exit marker (last path cell before exit)
var lastCell=tdfPath[tdfPath.length-2]; // second-to-last is last on-board cell
tdfCtx.strokeStyle='#f44';tdfCtx.lineWidth=2;
tdfCtx.strokeRect(lastCell.x*TDF_CELL+1,lastCell.y*TDF_CELL+1,TDF_CELL-2,TDF_CELL-2);
```

- [ ] **Step 20: Commit**

```bash
git add tetris.html
git commit -m "feat: enhance path visuals with colors, arrows, and entry/exit markers"
```

---

### Task 5: Loadout System

**Files:**
- Modify: `D:\app\tetris.html` (HTML, CSS, JS)

- [ ] **Step 21: Add loadout screen HTML**

After `screenTDWait` closing `</div>`, add:

```html
<!-- TOWER DEFENSE LOADOUT -->
<div class="screen" id="screenTDLoadout">
  <div style="width:100%;text-align:left;margin-bottom:8px;"><button class="back-btn" id="btnBackTDLoadout">← 返回</button></div>
  <div class="select-title">🎒 确认卡组</div>
  <div style="display:grid;grid-template-columns:1fr 1fr;gap:10px;width:100%;max-width:340px;margin-bottom:16px;" id="tdfLoadoutGrid"></div>
  <button class="menu-btn" id="btnTDLoadoutConfirm" style="background:linear-gradient(135deg,#2a2a1e,#1a1a0e);border-color:#4f8;font-size:16px;padding:10px 40px;">✔ 确认完成</button>
</div>
```

- [ ] **Step 22: Add loadout JS functions**

In tower defense JS block, add loadout logic:

```javascript
function tdfLoadLoadout(){
  try{return JSON.parse(localStorage.getItem('goblin_tdf_loadout')||'[]');}catch(e){return [];}
}
function tdfSaveLoadout(arr){localStorage.setItem('goblin_tdf_loadout',JSON.stringify(arr));}
function tdfShowLoadout(){
  var sel=tdfLoadLoadout();if(!sel.length)sel=['arrow','cannon','ice','mine'];
  var types=['arrow','cannon','ice','mine'];
  var grid=document.getElementById('tdfLoadoutGrid');
  grid.innerHTML=types.map(function(t){
    var cfg=TD_TOWERS[t];var checked=sel.indexOf(t)>=0;
    return '<div class="gcard" style="cursor:pointer;padding:14px 10px;text-align:center;'+(checked?'border-color:#4f8;box-shadow:0 0 10px rgba(64,255,128,0.3);':'')+'" data-type="'+t+'" id="ldt_'+t+'"><div style="font-size:32px;">'+cfg.icon+'</div><div style="font-size:14px;font-weight:bold;">'+cfg.label+'</div><div style="font-size:10px;color:var(--gold);">'+cfg.cost+'g</div><div style="font-size:18px;margin-top:4px;color:'+(checked?'#4f8':'#555')+';">'+(checked?'✅':'⬜')+'</div></div>';
  }).join('');
  // Tap to toggle - but minimum 4 must be selected (all required)
  types.forEach(function(t){
    document.getElementById('ldt_'+t).onclick=function(){
      // All towers required - no toggle allowed
    };
  });
  ss('screenTDLoadout');
}
document.getElementById('btnTDLoadoutConfirm').onclick=function(){
  tdfSaveLoadout(['arrow','cannon','ice','mine']);
  ss('screenTowerDefenseDiff');
};
```

- [ ] **Step 23: Add prepare button to difficulty screen**

In `screenTowerDefenseDiff`, add between the last diff card and the LB button:

```html
<button class="menu-btn" id="btnTDPrepare" style="margin-top:12px;background:linear-gradient(135deg,#2a2a1e,#1a1a0e);border-color:#4af;font-size:16px;padding:10px 40px;">🎒 确认卡组</button>
```

- [ ] **Step 24: Integrate loadout with game palette**

In `tdfInit()`, after the existing palette reset code, add loadout-driven palette filtering:

```javascript
var loadout=tdfLoadLoadout();if(!loadout.length)loadout=['arrow','cannon','ice','mine'];
document.querySelectorAll('#tdfPalette .tdf-tower-btn').forEach(function(b){
  var t=b.getAttribute('data-type');
  b.style.display=loadout.indexOf(t)>=0?'':'none';
});
```

- [ ] **Step 25: Wire loadout button**

```javascript
document.getElementById('btnTDPrepare').onclick=function(){tdfShowLoadout();};
document.getElementById('btnBackTDLoadout').onclick=function(){ss('screenTowerDefenseDiff');};
```

- [ ] **Step 26: Commit**

```bash
git add tetris.html
git commit -m "feat: add tower loadout system with prepare screen"
```

---

### Task 6: Verify & Deploy

- [ ] **Step 27: Verify all changes**

1. Open `tetris.html` in browser
2. Solo mode: tap 🏰 → difficulty → "单人模式" → verify game starts, no chat/ready buttons
3. Quick match: tap "快速匹配" → verify waiting screen shows, not game
4. Speed: in game, tap ⚡ → verify 2× speed, toggle back
5. Tower popup: tap placed tower → verify popup with upgrade cost and sell
6. Path: verify brown path tiles with directional arrows, green entry, red exit
7. Loadout: tap "🎒 确认卡组" on diff screen → verify 4 cards with ✅ → confirm
8. Console: check for JS errors

- [ ] **Step 28: Fix any issues**

- [ ] **Step 29: Push and deploy**

```bash
git push origin main
# Deploy: upload tetris.html as index.html via gh api
```
