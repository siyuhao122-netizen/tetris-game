# Co-op Tower Defense — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a 7th game — 2-player co-op tower defense with 4 difficulties, shared board/gold/HP, Supabase realtime multiplayer, friend co-op and quick match.

**Architecture:** All changes in `tetris.html`. `tdf` prefix for all tower defense variables. Reuses existing room/lobby/matchmaking system (same pattern as Tetris battle + Gomoku). New screens: `screenTowerDefenseDiff`, `screenTDLobby`, `screenTowerDefense`. Game select grid expands to 2×4.

**Tech Stack:** Vanilla HTML/CSS/JS, Canvas 2D, Supabase Realtime

---

### Task 1: CSS + Screen HTML + Game Card

**Files:**
- Modify: `D:\app\tetris.html` (style block, body HTML)

- [ ] **Step 1: Add tower defense CSS**

Find the `<style>` block. Before `</style>`, add:

```css
.tdf-palette{display:flex;gap:8px;justify-content:center;flex-wrap:wrap;margin-top:4px;}
.tdf-tower-btn{background:var(--surface);border:2px solid var(--bdr);border-radius:10px;padding:8px 14px;cursor:pointer;font-size:13px;color:var(--txt);min-width:60px;text-align:center;transition:border-color 0.15s;touch-action:manipulation;-webkit-tap-highlight-color:transparent;}
.tdf-tower-btn:active{background:#2a2a4e;}
.tdf-tower-btn.sel{border-color:var(--accent);box-shadow:0 0 12px rgba(80,120,255,0.4);}
.tdf-tower-btn .cost{font-size:10px;color:var(--gold);}
.tdf-status{display:flex;gap:12px;justify-content:center;align-items:center;font-size:14px;font-weight:bold;margin-bottom:2px;flex-wrap:wrap;}
.tdf-status span{color:var(--accent);}
.tdf-chat-row{display:flex;gap:6px;justify-content:center;flex-wrap:wrap;margin-top:4px;}
.tdf-chat-btn{background:#1a1a2e;border:1px solid #444;border-radius:8px;color:#aaa;font-size:11px;padding:4px 10px;cursor:pointer;touch-action:manipulation;}
.tdf-chat-btn:active{background:#2a2a4e;color:#fff;}
```

- [ ] **Step 2: Replace game select grid from 2×3 to 2×4**

Find `#screenSelect` div. The `.game-grid` currently has 6 `.gcard` divs. Change `grid-template-columns` in `.game-grid` CSS to handle 4 columns. Then add the 7th card after breakout:

```html
<div class="gcard towerdefense" id="cardTowerDefense"><div class="gicon">🏰</div><div class="gname">塔防守卫</div><div class="gdesc">双人合作塔防</div></div>
```

Add the CSS accent:
```css
.gcard.towerdefense{border-top:3px solid #f44;}
```

- [ ] **Step 3: Add 3 new screens after `screenBreakoutDiff` closing `</div>`**

Find `screenBreakoutDiff` closing `</div>`. After it, add:

```html
<!-- TOWER DEFENSE DIFFICULTY -->
<div class="screen" id="screenTowerDefenseDiff">
  <div style="width:100%;text-align:left;margin-bottom:8px;"><button class="back-btn" id="btnBackTDDiff">← 返回</button></div>
  <div class="select-title">🏰 塔防守卫 · 难度</div>
  <div class="diff-card" id="tdfDiffEasy"><div class="dname">🟢 简单</div><div class="ddesc">20波 · 塔最高5级 · 25❤</div><div class="dhigh">🏆 最高 <span id="tdfhsEasy">0</span>波</div></div>
  <div class="diff-card" id="tdfDiffNormal"><div class="dname">🟡 普通</div><div class="ddesc">30波 · 塔最高4级 · 20❤</div><div class="dhigh">🏆 最高 <span id="tdfhsNormal">0</span>波</div></div>
  <div class="diff-card" id="tdfDiffHard"><div class="dname">🔴 困难</div><div class="ddesc">40波 · 塔最高3级 · 15❤</div><div class="dhigh">🏆 最高 <span id="tdfhsHard">0</span>波</div></div>
  <div class="diff-card" id="tdfDiffEndless"><div class="dname">♾️ 无尽</div><div class="ddesc">无限波 · 无等级上限 · 20❤</div><div class="dhigh">🏆 最高 <span id="tdfhsEndless">0</span>波</div></div>
  <button class="menu-btn" id="btnTowerDefenseLB" style="margin-top:16px;background:linear-gradient(135deg,#2a2a1e,#1a1a0e);border-color:#f90;font-size:16px;padding:10px 40px;">🏆 排行榜</button>
</div>

<!-- TOWER DEFENSE LOBBY -->
<div class="screen" id="screenTDLobby">
  <div style="width:100%;text-align:left;margin-bottom:8px;"><button class="back-btn" id="btnBackTDLobby">← 返回</button></div>
  <div class="select-title">🏰 塔防守卫</div>
  <button class="menu-btn" id="btnTDFriendBattle" style="margin-bottom:16px;">👥 好友合作</button>
  <button class="menu-btn" id="btnTDOnlineBattle" style="margin-bottom:16px;">🌐 快速匹配</button>
</div>

<!-- TOWER DEFENSE GAME -->
<div class="screen" id="screenTowerDefense">
  <div style="width:100%;text-align:left;margin-bottom:4px;display:flex;justify-content:space-between;align-items:center;">
    <button class="back-btn" id="btnBackTD">← 退出</button>
    <span class="o-detail" id="tdfRoomCode" style="font-size:12px;color:#888;"></span>
  </div>
  <div class="game-wrapper">
    <div class="tdf-status" id="tdfStatusBar">
      ❤<span id="tdfHP">20</span> 💰<span id="tdfGold">200</span> 🌊<span id="tdfWave">0</span>/<span id="tdfWaveMax">20</span>
    </div>
    <div class="board-wrap" style="position:relative;">
      <canvas id="tdfBoard" class="gcanvas" width="392" height="224"></canvas>
      <div id="tdfPauseOverlay" style="position:absolute;top:0;left:0;right:0;bottom:0;background:rgba(0,0,0,0.6);display:none;justify-content:center;align-items:center;z-index:10;border-radius:4px;"><span style="font-size:32px;font-weight:bold;color:#fff;">⏸ 暂停</span></div>
    </div>
    <div class="tdf-palette" id="tdfPalette">
      <button class="tdf-tower-btn" data-type="arrow">🏹<br><span class="cost">50g</span></button>
      <button class="tdf-tower-btn" data-type="cannon">💣<br><span class="cost">100g</span></button>
      <button class="tdf-tower-btn" data-type="ice">❄️<br><span class="cost">75g</span></button>
      <button class="tdf-tower-btn" data-type="mine">💰<br><span class="cost">150g</span></button>
    </div>
    <div class="controls" style="margin-top:4px;">
      <button class="ctrl-btn" id="tdfReadyBtn" style="font-size:16px;padding:8px 24px;">✅ 准备</button>
    </div>
    <div class="tdf-chat-row" id="tdfChatRow">
      <button class="tdf-chat-btn">💬 建这里！</button>
      <button class="tdf-chat-btn">💰 存钱升级</button>
      <button class="tdf-chat-btn">✅ 准备好了</button>
      <button class="tdf-chat-btn">🆘 需要帮助</button>
    </div>
  </div>
</div>
```

- [ ] **Step 4: Commit**

```bash
git add tetris.html
git commit -m "feat: add tower defense CSS, screens, and game card"
```

---

### Task 2: Tower Defense Core — Data, Init, Path System

**Files:**
- Modify: `D:\app\tetris.html` (script block — add after Breakout section)

- [ ] **Step 5: Add tower defense data structures and init**

In the `<script>` block, find where the Breakout section ends (search for `document.getElementById("cardBreakout").onclick`). After the Breakout code block, add:

```javascript
// ===== TOWER DEFENSE =====
var TDF={
  easy:{waves:20,maxLevel:5,hp:25,gold:200,path:'s_curve',label:'简单',e:'🟢'},
  normal:{waves:30,maxLevel:4,hp:20,gold:150,path:'winding',label:'普通',e:'🟡'},
  hard:{waves:40,maxLevel:3,hp:15,gold:100,path:'maze',label:'困难',e:'🔴'},
  endless:{waves:Infinity,maxLevel:Infinity,hp:20,gold:150,path:'winding',label:'无尽',e:'♾️'}
};
var TDF_CELL=28,TDF_COLS=14,TDF_ROWS=8;
var tdfCanvas,tdfCtx,tdfDifficulty='normal',tdfGrid=[],tdfPath=[],tdfPathSet={};
var tdfEnemies=[],tdfTowers=[],tdfHP=20,tdfGold=200,tdfWave=0,tdfWaveMax=20;
var tdfPrepPhase=true,tdfPrepTimer=0,tdfReady=0,tdfSelected=null,tdfMaxLevel=4;
var tdfScore=0,tdfStarted=0,tdfGameOver=0,tdfPause=0,tdfLoopId=null;
var tdfRoom=null,tdfPlayer=0,tdfChannel=null,tdfSpawnQueue=[],tdfSpawnTimer=0;
var tdfEnemiesSpawned=0,tdfStartHP=20,tdfWon=0;

var TD_TOWERS={
  arrow:{cost:50,damage:15,range:3,cooldown:500,icon:'🏹',color:'#4af',label:'箭塔'},
  cannon:{cost:100,damage:40,range:2,cooldown:1500,icon:'💣',color:'#f80',label:'炮塔',splash:1},
  ice:{cost:75,damage:0,range:2.5,cooldown:1000,icon:'❄️',color:'#8ff',label:'冰塔',slow:0.5,slowDur:1500},
  mine:{cost:150,damage:0,range:0,cooldown:0,icon:'💰',color:'#ff0',label:'金矿',goldPerWave:10}
};

var TD_ENEMIES={
  scout:{hp:30,speed:2,color:'#4f8',bounty:10,label:'侦察兵'},
  warrior:{hp:80,speed:1.2,color:'#f90',bounty:20,label:'战士'},
  tank:{hp:200,speed:0.6,color:'#f44',bounty:40,label:'重装兵',weakCannon:true},
  healer:{hp:100,speed:1,color:'#a0f',bounty:25,label:'治疗兵',heal:5,healRange:2},
  king:{hp:500,speed:0.5,color:'#f0f',bounty:200,label:'哥布林王',boss:true}
};

var TDF_PATHS={
  s_curve:[
    {x:0,y:0},{x:13,y:0},{x:13,y:1},{x:0,y:1},{x:0,y:2},{x:13,y:2},{x:13,y:3},{x:0,y:3},
    {x:0,y:4},{x:13,y:4},{x:13,y:5},{x:0,y:5},{x:0,y:6},{x:13,y:6},{x:13,y:7},{x:14,y:7}
  ],
  winding:[
    {x:0,y:0},{x:4,y:0},{x:4,y:2},{x:10,y:2},{x:10,y:0},{x:13,y:0},{x:13,y:3},{x:7,y:3},
    {x:7,y:1},{x:0,y:1},{x:0,y:4},{x:6,y:4},{x:6,y:6},{x:13,y:6},{x:13,y:7},{x:14,y:7}
  ],
  maze:[
    {x:0,y:0},{x:3,y:0},{x:3,y:3},{x:10,y:3},{x:10,y:1},{x:6,y:1},{x:6,y:5},{x:13,y:5},
    {x:13,y:2},{x:0,y:2},{x:0,y:6},{x:8,y:6},{x:8,y:4},{x:3,y:4},{x:3,y:7},{x:13,y:7},{x:14,y:7}
  ]
};

function tdfBuildPathSet(){
  tdfPathSet={};
  for(var i=0;i<tdfPath.length-1;i++){
    var a=tdfPath[i],b=tdfPath[i+1];
    var dx=Math.abs(b.x-a.x),dy=Math.abs(b.y-a.y);
    var steps=Math.max(dx,dy);
    for(var s=0;s<=steps;s++){
      var t=steps===0?0:s/steps;
      var cx=Math.round(a.x+(b.x-a.x)*t);
      var cy=Math.round(a.y+(b.y-a.y)*t);
      tdfPathSet[cx+','+cy]=1;
    }
  }
}

function tdfInit(){
  var cfg=TDF[tdfDifficulty];
  tdfGrid=[];for(var r=0;r<TDF_ROWS;r++){tdfGrid[r]=[];for(var c=0;c<TDF_COLS;c++)tdfGrid[r][c]=null;}
  tdfPath=TDF_PATHS[cfg.path];
  tdfBuildPathSet();
  tdfEnemies=[];tdfTowers=[];tdfSpawnQueue=[];tdfSpawnTimer=0;tdfEnemiesSpawned=0;
  tdfHP=cfg.hp;tdfStartHP=cfg.hp;tdfGold=cfg.gold;tdfWave=0;tdfWaveMax=cfg.waves;
  tdfMaxLevel=cfg.maxLevel;tdfPrepPhase=true;tdfPrepTimer=15;tdfReady=0;tdfSelected=null;
  tdfScore=0;tdfStarted=1;tdfGameOver=0;tdfPause=0;tdfWon=0;
  if(tdfLoopId){clearInterval(tdfLoopId);tdfLoopId=null;}
  tdfCanvas=document.getElementById('tdfBoard');tdfCtx=tdfCanvas.getContext('2d');
  _setupHiDPI(tdfCanvas,tdfCtx,TDF_COLS*TDF_CELL,TDF_ROWS*TDF_CELL);
  document.getElementById('tdfHP').textContent=tdfHP;
  document.getElementById('tdfGold').textContent=tdfGold;
  document.getElementById('tdfWave').textContent='0';
  document.getElementById('tdfWaveMax').textContent(tdfWaveMax===Infinity?'∞':tdfWaveMax);
  document.querySelectorAll('#tdfPalette .tdf-tower-btn').forEach(function(b){b.classList.remove('sel');});
  tdfDraw();
}
```

- [ ] **Step 6: Commit**

```bash
git add tetris.html
git commit -m "feat: add tower defense data structures, path system, and init"
```

---

### Task 3: Tower System — Placement, Upgrade, Sell, Attack

**Files:**
- Modify: `D:\app\tetris.html` (add after tdfInit in tower defense block)

- [ ] **Step 7: Add tower placement, upgrade, sell functions**

```javascript
function tdfUpgradeCost(tower){
  return Math.floor(TD_TOWERS[tower.type].cost*Math.pow(2,tower.level));
}
function tdfTotalSpent(tower){
  var sum=0;for(var i=0;i<tower.level;i++)sum+=Math.floor(TD_TOWERS[tower.type].cost*Math.pow(2,i));
  return sum;
}
function tdfCanPlace(x,y){
  if(x<0||x>=TDF_COLS||y<0||y>=TDF_ROWS)return false;
  if(tdfPathSet[x+','+y])return false;
  if(tdfGrid[y][x])return false;
  return true;
}
function tdfPlaceTower(x,y,type,level,broadcast){
  level=level||1;
  if(!tdfCanPlace(x,y))return false;
  var cost=Math.floor(TD_TOWERS[type].cost*Math.pow(2,level-1));
  if(tdfGold<cost)return false;
  tdfGold-=cost;
  var tw={x:x,y:y,type:type,level:level,cooldown:0};
  tdfGrid[y][x]=tw;tdfTowers.push(tw);
  document.getElementById('tdfGold').textContent=tdfGold;
  if(broadcast!==false&&tdfChannel)tdfChannel.send({type:'broadcast',event:'tdf_place',payload:{x:x,y:y,type:type}});
  return true;
}
function tdfUpgradeTower(x,y,broadcast){
  var tw=tdfGrid[y][x];if(!tw)return false;
  if(tdfMaxLevel!==Infinity&&tw.level>=tdfMaxLevel)return false;
  var cost=tdfUpgradeCost(tw);
  if(tdfGold<cost)return false;
  tdfGold-=cost;tw.level++;
  document.getElementById('tdfGold').textContent=tdfGold;
  if(broadcast!==false&&tdfChannel)tdfChannel.send({type:'broadcast',event:'tdf_upgrade',payload:{x:x,y:y}});
  return true;
}
function tdfSellTower(x,y,broadcast){
  var tw=tdfGrid[y][x];if(!tw)return false;
  var refund=Math.floor(tdfTotalSpent(tw)*0.5);
  tdfGold+=refund;
  tdfGrid[y][x]=null;
  var idx=tdfTowers.indexOf(tw);if(idx>=0)tdfTowers.splice(idx,1);
  document.getElementById('tdfGold').textContent=tdfGold;
  if(broadcast!==false&&tdfChannel)tdfChannel.send({type:'broadcast',event:'tdf_sell',payload:{x:x,y:y}});
  return true;
}
```

- [ ] **Step 8: Add tower attack logic**

```javascript
function tdfTowerAttack(tw){
  var cfg=TD_TOWERS[tw.type];
  if(cfg.goldPerWave)return; // Gold mine doesn't attack
  if(tw.cooldown>0){tw.cooldown-=200;return;}
  var target=null,targetDist=Infinity;
  for(var i=0;i<tdfEnemies.length;i++){
    var e=tdfEnemies[i];
    var dx=e.x-tw.x,dy=e.y-tw.y;
    var dist=Math.sqrt(dx*dx+dy*dy);
    var range=cfg.range+(tw.level-1)*0.05*cfg.range;
    if(dist<=range&&dist<targetDist){target=e;targetDist=dist;}
  }
  if(!target)return;
  var dmg=cfg.damage+(tw.level-1)*Math.floor(cfg.damage*0.3);
  if(cfg.weakCannon&&tw.type==='cannon')dmg*=2;
  target.hp-=dmg;tw.cooldown=cfg.cooldown*(1-(tw.level-1)*0.1);
  if(cfg.splash){
    var splashR=cfg.splash||1;
    for(var j=0;j<tdfEnemies.length;j++){
      var ej=tdfEnemies[j];if(ej===target)continue;
      var dx2=ej.x-target.x,dy2=ej.y-target.y;
      if(Math.sqrt(dx2*dx2+dy2*dy2)<=splashR)ej.hp-=Math.floor(dmg*0.5);
    }
  }
  if(cfg.slow){
    target.slow=cfg.slow+(tw.level-1)*0.05;
    target.slowUntil=Date.now()+cfg.slowDur;
  }
  _playTone(tw.type==='cannon'?150:tw.type==='arrow'?600:300,0.05,0.08);
}
```

- [ ] **Step 9: Commit**

```bash
git add tetris.html
git commit -m "feat: add tower placement, upgrade, sell, and attack logic"
```

---

### Task 4: Enemy System — Spawn, Move, Effects

**Files:**
- Modify: `D:\app\tetris.html` (add after tower attack code)

- [ ] **Step 10: Add enemy spawning logic**

```javascript
function tdfSpawnEnemy(type,overrideHp){
  var cfg=TD_ENEMIES[type];
  var hp=overrideHp||cfg.hp;
  if(tdfDifficulty==='endless'&&tdfWave>20){
    var scaleWaves=tdfWave-20;
    if(!cfg.boss)hp=Math.floor(cfg.hp*Math.pow(1.15,scaleWaves));
  }
  if(cfg.boss)hp=Math.floor(cfg.hp*Math.pow(1.25,Math.floor(tdfWave/5)));
  var e={type:type,hp:hp,maxHp:hp,speed:cfg.speed,color:cfg.color,bounty:cfg.bounty,
    x:tdfPath[0].x,y:tdfPath[0].y,pathIdx:1,slow:0,slowUntil:0,heal:cfg.heal||0,healRange:cfg.healRange||0,
    weakCannon:cfg.weakCannon||false,boss:cfg.boss||false};
  tdfEnemies.push(e);
}
function tdfSpawnWave(){
  tdfEnemiesSpawned=0;
  var count=3+tdfWave*2;
  var types=[];
  if(tdfWave%5===0){types.push('king');count+=1;}
  if(tdfWave<=4)types=['scout'];
  else if(tdfWave<=9)types=['scout','warrior'];
  else if(tdfWave<=14)types=['warrior','warrior','tank'];
  else types=['tank','tank','healer','warrior'];
  for(var i=0;i<count;i++){
    var t=types[Math.floor(Math.random()*types.length)];
    if(t==='king'&&i>0)continue;
    tdfSpawnQueue.push(t);
  }
  tdfPrepPhase=true;tdfPrepTimer=15;tdfReady=0;
  if(tdfDifficulty==='endless'&&tdfWave>10){tdfPrepTimer=0;tdfPrepPhase=false;}
  // Gold mines produce gold each wave
  tdfTowers.forEach(function(tw){
    var cfg=TD_TOWERS[tw.type];
    if(cfg.goldPerWave){tdfGold+=cfg.goldPerWave+(tw.level-1)*5;}
  });
  document.getElementById('tdfGold').textContent=tdfGold;
  document.getElementById('tdfWave').textContent=tdfWave;
}
```

- [ ] **Step 11: Add enemy movement and effects**

```javascript
function tdfMoveEnemies(){
  for(var i=tdfEnemies.length-1;i>=0;i--){
    var e=tdfEnemies[i];
    var spd=e.speed;if(e.slow&&Date.now()<e.slowUntil)spd*=1-e.slow;
    if(e.slow&&Date.now()>=e.slowUntil){e.slow=0;e.slowUntil=0;}
    var wp=tdfPath[e.pathIdx];
    var dx=wp.x-e.x,dy=wp.y-e.y;
    var dist=Math.sqrt(dx*dx+dy*dy);
    if(dist<0.15){
      e.x=wp.x;e.y=wp.y;e.pathIdx++;
      if(e.pathIdx>=tdfPath.length){
        tdfHP-=(e.boss?5:1);tdfEnemies.splice(i,1);
        document.getElementById('tdfHP').textContent=tdfHP;
        if(tdfHP<=0)tdfLose();
        continue;
      }
    }else{
      var move=spd*0.2;
      if(move>dist)move=dist;
      var ratio=move/(dist||0.001);
      e.x+=dx*ratio;e.y+=dy*ratio;
    }
    // Healer effect
    if(e.heal){
      for(var j=0;j<tdfEnemies.length;j++){
        var ej=tdfEnemies[j];if(ej===e)continue;
        var hdx=ej.x-e.x,hdy=ej.y-e.y;
        if(Math.sqrt(hdx*hdx+hdy*hdy)<=e.healRange)ej.hp=Math.min(ej.maxHp,ej.hp+e.heal*0.2);
      }
    }
  }
}
function tdfCheckDead(){
  for(var i=tdfEnemies.length-1;i>=0;i--){
    if(tdfEnemies[i].hp<=0){
      var e=tdfEnemies[i];tdfGold+=e.bounty;tdfScore+=e.bounty;
      tdfEnemies.splice(i,1);
      document.getElementById('tdfGold').textContent=tdfGold;
      _playTone(80,0.06,0.1);
    }
  }
}
```

- [ ] **Step 12: Commit**

```bash
git add tetris.html
git commit -m "feat: add enemy spawn, movement, healing, and death logic"
```

---

### Task 5: Game Loop and Wave Management

**Files:**
- Modify: `D:\app\tetris.html` (add after enemy code)

- [ ] **Step 13: Add main game loop**

```javascript
function tdfTick(){
  if(tdfPause||tdfGameOver||!tdfStarted)return;
  // Spawn queued enemies
  if(tdfSpawnQueue.length>0){
    tdfSpawnTimer-=200;
    if(tdfSpawnTimer<=0){
      tdfSpawnEnemy(tdfSpawnQueue.shift());
      tdfSpawnTimer=400+Math.random()*300;
    }
  }
  // Prep phase countdown
  if(tdfPrepPhase&&!tdfSpawnQueue.length&&tdfEnemies.length===0){
    if(tdfPrepTimer>0&&tdfReady!==3){
      tdfPrepTimer=Math.max(0,tdfPrepTimer-0.2);
    }
    if((tdfPrepTimer<=0||tdfReady===3)&&tdfSpawnQueue.length===0){
      tdfWave++;
      document.getElementById('tdfWave').textContent=tdfWave;
      if(tdfWave>tdfWaveMax&&tdfWaveMax!==Infinity){tdfWin();return;}
      tdfSpawnWave();tdfPrepPhase=false;
    }
  }
  // Process enemies
  tdfMoveEnemies();
  tdfCheckDead();
  // Tower attacks
  tdfTowers.forEach(function(tw){tdfTowerAttack(tw);});
  // Check spawn queue empty + enemies dead
  if(!tdfPrepPhase&&tdfSpawnQueue.length===0&&tdfEnemies.length===0){
    tdfPrepPhase=true;tdfPrepTimer=15;tdfReady=0;
    if(tdfDifficulty==='endless'&&tdfWave>=10){tdfPrepTimer=0;tdfPrepPhase=false;}
  }
  tdfDraw();
}
function tdfStartLoop(){
  if(tdfLoopId)clearInterval(tdfLoopId);
  tdfSpawnWave();tdfPrepPhase=false;
  tdfLoopId=setInterval(tdfTick,200);
}
function tdfWin(){
  tdfGameOver=1;tdfWon=1;_playClear();_vibrate([50,30,50]);
  tdfScore=Math.floor(tdfScore*_vipMultiplier());
  recordGame('towerdefense',tdfDifficulty,tdfScore,0,1);
  var nw=shs('towerdefense',tdfDifficulty,tdfWave);
  document.getElementById('goScore').textContent=tdfScore;
  document.getElementById('goDiff').textContent=TDF[tdfDifficulty].e+' '+TDF[tdfDifficulty].label+' · 塔防';
  document.getElementById('goDetail').textContent='存活 '+tdfWave+' 波 · 💰'+tdfGold+' · ❤'+tdfHP;
  document.getElementById('goRecord').textContent=nw?'🏆 新纪录！':'🏆 最高 '+ghs('towerdefense',tdfDifficulty)+'波';
  document.getElementById('gameOverOverlay').classList.add('active');
  var titleEl=document.querySelector('#gameOverOverlay .o-title');
  if(titleEl)titleEl.textContent='🎉 防守成功！';
  window._gog='towerdefense';
  if(_userId)_submitLB('towerdefense',tdfDifficulty,tdfScore,0,null,{waves:tdfWave});
  if(tdfHP===tdfStartHP)_unlockAch('towerdefense_noleak');
  if(tdfWave>=10)_unlockAch('towerdefense_wave10');
  if(tdfWave>=20)_unlockAch('towerdefense_wave20');
  if(tdfGold>=1000)_unlockAch('towerdefense_rich');
  _initAudio();uhs();
}
function tdfLose(){
  tdfGameOver=1;_playGameOver();
  tdfScore=Math.floor(tdfScore*_vipMultiplier());
  recordGame('towerdefense',tdfDifficulty,tdfScore,0,0);
  var nw=shs('towerdefense',tdfDifficulty,tdfWave);
  document.getElementById('goScore').textContent=tdfScore;
  document.getElementById('goDiff').textContent=TDF[tdfDifficulty].e+' '+TDF[tdfDifficulty].label+' · 塔防';
  document.getElementById('goDetail').textContent='存活 '+tdfWave+' 波 · 💰'+tdfGold;
  document.getElementById('goRecord').textContent=nw?'🏆 新纪录！':'🏆 最高 '+ghs('towerdefense',tdfDifficulty)+'波';
  document.getElementById('gameOverOverlay').classList.add('active');
  window._gog='towerdefense';
  if(_userId)_submitLB('towerdefense',tdfDifficulty,tdfScore,0,null,{waves:tdfWave});
  if(tdfWave>=10)_unlockAch('towerdefense_wave10');
  if(tdfGold>=1000)_unlockAch('towerdefense_rich');
  _initAudio();uhs();
}
```

- [ ] **Step 14: Commit**

```bash
git add tetris.html
git commit -m "feat: add tower defense game loop, wave management, win/lose"
```

---

### Task 6: Canvas Rendering

**Files:**
- Modify: `D:\app\tetris.html` (add after game loop code)

- [ ] **Step 15: Add draw function**

```javascript
function tdfDraw(){
  tdfCtx.clearRect(0,0,TDF_COLS*TDF_CELL,TDF_ROWS*TDF_CELL);
  // Draw path tiles
  for(var r=0;r<TDF_ROWS;r++){
    for(var c=0;c<TDF_COLS;c++){
      var isPath=tdfPathSet[c+','+r];
      tdfCtx.fillStyle=isPath?'#1a1a0a':'#0a0a1a';
      tdfCtx.fillRect(c*TDF_CELL+1,r*TDF_CELL+1,TDF_CELL-2,TDF_CELL-2);
    }
  }
  // Draw towers
  tdfTowers.forEach(function(tw){
    var cfg=TD_TOWERS[tw.type];
    var cx=tw.x*TDF_CELL+TDF_CELL/2,cy=tw.y*TDF_CELL+TDF_CELL/2;
    var r=TDF_CELL/2-3;
    tdfCtx.fillStyle=cfg.color;
    tdfCtx.beginPath();tdfCtx.arc(cx,cy,r,0,Math.PI*2);tdfCtx.fill();
    tdfCtx.fillStyle='#fff';tdfCtx.font=(TDF_CELL/2)+'px sans-serif';tdfCtx.textAlign='center';tdfCtx.textBaseline='middle';
    tdfCtx.fillText(cfg.icon,cx,cy-1);
    if(tw.level>1){
      tdfCtx.fillStyle='#ff0';tdfCtx.font='bold '+(TDF_CELL/4)+'px sans-serif';
      tdfCtx.fillText(tw.level,cx+TDF_CELL/3,cy+TDF_CELL/3);
    }
  });
  // Draw enemies
  tdfEnemies.forEach(function(e){
    var cx=e.x*TDF_CELL,cy=e.y*TDF_CELL;
    var r=TDF_CELL/2-4;
    tdfCtx.fillStyle=e.color;
    tdfCtx.beginPath();tdfCtx.arc(cx,cy,r,0,Math.PI*2);tdfCtx.fill();
    // HP bar
    if(e.hp<e.maxHp){
      tdfCtx.fillStyle='#333';
      tdfCtx.fillRect(cx-r,cy-r-3,r*2,3);
      tdfCtx.fillStyle='#4f8';
      tdfCtx.fillRect(cx-r,cy-r-3,r*2*(e.hp/e.maxHp),3);
    }
    // Slow indicator
    if(e.slow&&Date.now()<e.slowUntil){
      tdfCtx.strokeStyle='#8ff';tdfCtx.lineWidth=1;
      tdfCtx.beginPath();tdfCtx.arc(cx,cy,r+2,0,Math.PI*2);tdfCtx.stroke();
    }
  });
  // Draw prep timer
  if(tdfPrepPhase&&tdfSpawnQueue.length===0&&tdfEnemies.length===0&&tdfPrepTimer>0){
    tdfCtx.fillStyle='rgba(255,255,255,0.8)';tdfCtx.font='bold 20px sans-serif';tdfCtx.textAlign='center';
    tdfCtx.fillText(Math.ceil(tdfPrepTimer)+'s',TDF_COLS*TDF_CELL/2,TDF_ROWS*TDF_CELL/2+8);
    if(tdfReady>0){
      tdfCtx.fillStyle='#4f8';tdfCtx.font='14px sans-serif';
      tdfCtx.fillText('Ready ('+tdfReady+'/2)',TDF_COLS*TDF_CELL/2,TDF_ROWS*TDF_CELL/2-16);
    }
  }
}
```

- [ ] **Step 16: Commit**

```bash
git add tetris.html
git commit -m "feat: add tower defense canvas rendering"
```

---

### Task 7: Input Handling

**Files:**
- Modify: `D:\app\tetris.html` (add after draw function)

- [ ] **Step 17: Add touch/click input for board and palette**

```javascript
function tdfGetCell(x,y){
  if(!tdfCanvas)return null;
  var rect=tdfCanvas.getBoundingClientRect();
  if(!rect||rect.width===0)return null;
  var sx=TDF_COLS*TDF_CELL/rect.width,sy=TDF_ROWS*TDF_CELL/rect.height;
  var cx=Math.floor((x-rect.left)*sx/TDF_CELL),cy=Math.floor((y-rect.top)*sy/TDF_CELL);
  if(cx>=0&&cx<TDF_COLS&&cy>=0&&cy<TDF_ROWS)return{x:cx,y:cy};
  return null;
}
tdfCanvas=document.getElementById('tdfBoard');
tdfCanvas.addEventListener('click',function(e){
  if(!tdfStarted||tdfGameOver||tdfPause)return;
  var cell=tdfGetCell(e.clientX,e.clientY);if(!cell)return;
  var tw=tdfGrid[cell.y][cell.x];
  if(tw){
    // Show upgrade/sell dialog
    var cost=tdfUpgradeCost(tw);
    var msg='升级 '+TD_TOWERS[tw.type].label+' Lv.'+tw.level+' → '+(tw.level+1)+' ('+cost+'g)';
    var canUp=(tdfMaxLevel===Infinity||tw.level<tdfMaxLevel)&&tdfGold>=cost;
    if(canUp){
      showConfirm(msg,function(){tdfUpgradeTower(cell.x,cell.y,true);},'取消');
    }else{
      var msg2='出售 '+TD_TOWERS[tw.type].label+' Lv.'+tw.level+' (+'+Math.floor(tdfTotalSpent(tw)*0.5)+'g)';
      showConfirm(msg2,function(){tdfSellTower(cell.x,cell.y,true);},'取消');
    }
  }else if(tdfSelected){
    tdfPlaceTower(cell.x,cell.y,tdfSelected,1,true);
  }
});
// Tower palette clicks
document.querySelectorAll('#tdfPalette .tdf-tower-btn').forEach(function(btn){
  btn.addEventListener('click',function(){
    var type=this.getAttribute('data-type');
    if(tdfSelected===type){tdfSelected=null;this.classList.remove('sel');return;}
    document.querySelectorAll('#tdfPalette .tdf-tower-btn').forEach(function(b){b.classList.remove('sel');});
    tdfSelected=type;this.classList.add('sel');
  });
});
// Ready button
document.getElementById('tdfReadyBtn').addEventListener('click',function(){
  tdfReady|=tdfPlayer;
  this.textContent='✅ 已准备';
  this.style.background='#2a2a1e';this.style.borderColor='#4f8';
  if(tdfChannel)tdfChannel.send({type:'broadcast',event:'tdf_ready',payload:{player:tdfPlayer}});
});
// Pause overlay toggle
document.getElementById('tdfPauseOverlay').addEventListener('click',function(){
  tdfPause=!tdfPause;
  document.getElementById('tdfPauseOverlay').style.display=tdfPause?'flex':'none';
  if(tdfPause)_playPause();else _playResume();
});
```

- [ ] **Step 18: Commit**

```bash
git add tetris.html
git commit -m "feat: add tower defense input handling and pause support"
```

---

### Task 8: Multiplayer Sync

**Files:**
- Modify: `D:\app\tetris.html` (add after input handling)

- [ ] **Step 19: Add multiplayer room and broadcast setup**

```javascript
function tdfSetupChannel(){
  if(!tdfRoom||!_sbClient)return;
  if(tdfChannel){tdfChannel.unsubscribe();}
  tdfChannel=_sbClient.channel('room:'+tdfRoom.code);
  tdfChannel.on('broadcast',{event:'tdf_place'},function(p){tdfPlaceTower(p.payload.x,p.payload.y,p.payload.type,1,false);});
  tdfChannel.on('broadcast',{event:'tdf_upgrade'},function(p){tdfUpgradeTower(p.payload.x,p.payload.y,false);});
  tdfChannel.on('broadcast',{event:'tdf_sell'},function(p){tdfSellTower(p.payload.x,p.payload.y,false);});
  tdfChannel.on('broadcast',{event:'tdf_ready'},function(p){tdfReady|=p.payload.player;});
  tdfChannel.on('broadcast',{event:'tdf_chat'},function(p){
    var div=document.createElement('div');div.textContent=p.payload.msg;
    div.style.cssText='position:absolute;top:20px;left:50%;transform:translateX(-50%);background:rgba(0,0,0,0.8);color:#fff;padding:4px 12px;border-radius:12px;font-size:12px;z-index:20;pointer-events:none;';
    document.querySelector('#screenTowerDefense .board-wrap').appendChild(div);
    setTimeout(function(){if(div.parentNode)div.parentNode.removeChild(div);},2000);
  });
  tdfChannel.subscribe(function(status){
    if(status==='SUBSCRIBED'){
      // Send full state for late-join sync
      tdfChannel.send({type:'broadcast',event:'tdf_sync',payload:{
        towers:tdfTowers.map(function(t){return{x:t.x,y:t.y,type:t.type,level:t.level};}),
        hp:tdfHP,gold:tdfGold,wave:tdfWave,state:tdfPrepPhase?'prep':'combat'
      }});
    }
  });
  tdfChannel.on('broadcast',{event:'tdf_sync'},function(p){
    var s=p.payload;
    tdfHP=s.hp;tdfGold=s.gold;tdfWave=s.wave;
    tdfTowers=[];tdfGrid=[];for(var r=0;r<TDF_ROWS;r++){tdfGrid[r]=[];for(var c=0;c<TDF_COLS;c++)tdfGrid[r][c]=null;}
    s.towers.forEach(function(t){tdfPlaceTower(t.x,t.y,t.type,t.level,false);});
    document.getElementById('tdfHP').textContent=tdfHP;
    document.getElementById('tdfGold').textContent=tdfGold;
    document.getElementById('tdfWave').textContent=tdfWave;
  });
}
// Quick chat buttons
document.querySelectorAll('#tdfChatRow .tdf-chat-btn').forEach(function(btn,i){
  var msgs=['建这里！','存钱升级！','准备好了！','需要帮助！'];
  btn.addEventListener('click',function(){
    if(tdfChannel)tdfChannel.send({type:'broadcast',event:'tdf_chat',payload:{msg:msgs[i]}});
  });
});
```

- [ ] **Step 20: Add lobby functions (friend co-op + quick match)**

```javascript
function tdfCreateRoom(mode){
  var code=_bGenCode();
  var status=mode==='friend'?'waiting':'matching';
  _sbReq('POST','/rest/v1/rooms',{code:code,status:status,game:'towerdefense',player1_id:_userId?parseInt(_userId):null}).then(function(r){return r.json();}).then(function(d){
    tdfRoom=d&&d.length?d[0]:null;tdfPlayer=1;
    document.getElementById('tdfRoomCode').textContent='房间: '+code;
    tdfSetupChannel();
    ss('screenTowerDefense');tdfInit();tdfStartLoop();
    if(mode==='matching')tdfPollMatch();
  });
}
function tdfJoinRoom(code){
  _sbReq('GET','/rest/v1/rooms?code=eq.'+code+'&select=*&limit=1').then(function(r){return r.json();}).then(function(d){
    if(!d||!d.length){_achieveBadge('房间不存在','❌');return;}
    var room=d[0];
    if(room.status!=='waiting'){_achieveBadge('房间已满','❌');return;}
    _sbReq('PATCH','/rest/v1/rooms?id=eq.'+room.id,{status:'playing',player2_id:_userId?parseInt(_userId):null}).then(function(){
      tdfRoom=room;tdfPlayer=2;
      document.getElementById('tdfRoomCode').textContent='房间: '+code;
      tdfSetupChannel();
      ss('screenTowerDefense');tdfInit();tdfStartLoop();
    });
  });
}
function tdfPollMatch(){
  var poll=setInterval(function(){
    _sbReq('GET','/rest/v1/rooms?game=eq.towerdefense&status=eq.matching&player1_id=neq.'+_userId+'&select=*&limit=1').then(function(r){return r.json();}).then(function(d){
      if(d&&d.length>0){
        _sbReq('PATCH','/rest/v1/rooms?id=eq.'+d[0].id,{status:'playing',player2_id:parseInt(_userId)}).then(function(){
          tdfRoom=d[0];tdfPlayer=2;clearInterval(poll);
          document.getElementById('tdfRoomCode').textContent='房间: '+d[0].code;
          tdfSetupChannel();tdfInit();tdfStartLoop();
        });
      }
    });
  },2000);
}
```

- [ ] **Step 21: Commit**

```bash
git add tetris.html
git commit -m "feat: add tower defense multiplayer sync and lobby"
```

---

### Task 9: Integration — Buttons, Leaderboard, Game Over Overlay

**Files:**
- Modify: `D:\app\tetris.html` (various locations)

- [ ] **Step 22: Wire difficulty cards, lobby buttons, back buttons**

Add these onclick handlers in the script block (near other button wiring):

```javascript
document.getElementById('cardTowerDefense').onclick=function(){_initAudio();ss('screenTowerDefenseDiff');uhs();};
document.getElementById('btnBackTDDiff').onclick=function(){ss('screenSelect');};
document.getElementById('tdfDiffEasy').onclick=function(){tdfDifficulty='easy';ss('screenTDLobby');};
document.getElementById('tdfDiffNormal').onclick=function(){tdfDifficulty='normal';ss('screenTDLobby');};
document.getElementById('tdfDiffHard').onclick=function(){tdfDifficulty='hard';ss('screenTDLobby');};
document.getElementById('tdfDiffEndless').onclick=function(){tdfDifficulty='endless';ss('screenTDLobby');};
document.getElementById('btnBackTDLobby').onclick=function(){ss('screenTowerDefenseDiff');};
document.getElementById('btnBackTD').onclick=function(){
  if(tdfStarted&&!tdfGameOver){showConfirm('确定退出吗？',function(){if(tdfLoopId)clearInterval(tdfLoopId);tdfStarted=0;ss('screenMenu');},'取消');return;}
  if(tdfLoopId)clearInterval(tdfLoopId);tdfStarted=0;ss('screenMenu');
};
document.getElementById('btnTDFriendBattle').onclick=function(){
  if(!_userId){_showAuthModal('login');return;}
  if(_friendsMode!=='battle')_loadFriends('battle');
  else ss('screenFriends');
  // The friend challenge system already handles 'towerdefense' via _battleGame
  _battleGame='towerdefense';
  ss('screenFriends');
};
document.getElementById('btnTDOnlineBattle').onclick=function(){
  if(!_userId){_showAuthModal('login');return;}
  tdfCreateRoom('match');
};
```

- [ ] **Step 23: Add towerdefense to leaderboard maps**

Find `function _showGameLB(game, backScreen){` (~line 3082). Update the maps:

```javascript
var titles={tetris:'🧱 俄罗斯方块 排行榜',snake:'🐍 贪吃蛇 排行榜','2048':'🧩 2048 排行榜',minesweeper:'💣 扫雷 排行榜',breakout:'🧱 打砖块 排行榜',towerdefense:'🏰 塔防守卫 排行榜'};
var diffs={tetris:['easy','normal','hard'],snake:['easy','normal','hard'],'2048':['easy','normal','hard','endless'],minesweeper:['easy','normal','hard'],breakout:['easy','normal','hard'],towerdefense:['easy','normal','hard','endless']};
```

Wire the LB button:
```javascript
document.getElementById('btnTowerDefenseLB').onclick=function(){_showGameLB('towerdefense','screenTowerDefenseDiff');};
```

- [ ] **Step 24: Wire game-over overlay for tower defense**

Find the game-over overlay restart/back handlers. Look for where `_gog==='breakout'` is handled. Add tower defense handling.

In the goRestart handler (search for `_origGoRestart`), add:
```javascript
if(window._gog==='towerdefense'){
  if(tdfLoopId){clearInterval(tdfLoopId);}
  tdfInit();tdfStartLoop();
  document.getElementById('gameOverOverlay').classList.remove('active');
  return;
}
```

In the goBack handler (search for `_origGoBack`), add:
```javascript
if(window._gog==='towerdefense'){
  if(tdfLoopId){clearInterval(tdfLoopId);}
  tdfStarted=0;ss('screenTowerDefenseDiff');
  document.getElementById('gameOverOverlay').classList.remove('active');
  return;
}
```

- [ ] **Step 25: Update friend challenge to support tower defense**

Find `_challengeFriend` function. Add `towerdefense` game type support:
```javascript
if(_battleGame==='towerdefense'){
  var code=_bGenCode();
  _sbReq('POST','/rest/v1/rooms',{code:code,status:'waiting',game:'towerdefense',player1_id:_userId?parseInt(_userId):null}).then(function(r){return r.json();}).then(function(d){
    if(!d||!d.length)return;
    tdfRoom=d[0];tdfPlayer=1;tdfDifficulty='normal';
    _sbReq('POST','/rest/v1/battle_invites',{from_id:parseInt(_userId),to_id:fid,room_code:code,status:'pending'}).then(function(){
      _achieveBadge('邀请已发送','📨');
      ss('screenTowerDefense');tdfInit();tdfStartLoop();
    });
  });
  return;
}
```

- [ ] **Step 26: Commit**

```bash
git add tetris.html
git commit -m "feat: wire tower defense buttons, leaderboard, and game-over handling"
```

---

### Task 10: Verification

- [ ] **Step 27: Verify tower defense game logic**

1. Open `tetris.html` in browser
2. Navigate to game select → verify 🏰 card in 2×4 grid
3. Tap card → verify difficulty screen with 4 options + LB button
4. Tap easy → verify lobby with friend/quick match buttons
5. Verify back navigation works at each step
6. Check browser console for JS errors

- [ ] **Step 28: Fix any issues found**

Address any console errors or UI bugs.

---

### Task 11: Deploy

- [ ] **Step 29: Push and deploy to GitHub Pages**

```bash
git push origin main
python -c "..." # same upload script as before
```
