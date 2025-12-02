# web
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Caza Rayos — Juego</title>
  <style>
    :root{
      --bg:#0b1020;
      --panel:#0f1724;
      --accent:#00eaff;
      --neon:#7df9ff;
      --muted:#94a3b8;
    }
    *{box-sizing:border-box}
    html,body{height:100%;margin:0;font-family:Inter,system-ui,Segoe UI,Roboto,Arial;background:radial-gradient(1200px 600px at 10% 10%, #071021, var(--bg));color:#e6eef8}
    .wrap{display:flex;gap:20px;align-items:flex-start;justify-content:center;padding:32px}
    .game-card{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.03);padding:18px;border-radius:12px;box-shadow:0 8px 30px rgba(2,6,23,0.6);width:720px}
    header{display:flex;align-items:center;justify-content:space-between;margin-bottom:12px}
    h1{font-size:18px;margin:0}
    .meta{color:var(--muted);font-size:13px}
    #gameCanvas{background:linear-gradient(180deg,#071427 0%, #081524 40%, #05233a 100%);width:100%;height:480px;border-radius:8px;display:block}

    .hud{display:flex;align-items:center;gap:12px;margin-top:12px}
    .battery{width:160px;height:34px;background:#081623;border-radius:8px;padding:6px;border:1px solid rgba(255,255,255,0.03);position:relative}
    .battery .cap{position:absolute;right:-12px;top:8px;width:10px;height:18px;background:#0b1622;border-radius:2px;border:1px solid rgba(255,255,255,0.03)}
    .battery .level{height:100%;width:10%;background:linear-gradient(90deg,var(--accent),var(--neon));border-radius:4px;transition:width 150ms ease}
    .stats{display:flex;gap:14px;align-items:center;color:var(--muted);font-size:13px}

    .controls{display:flex;gap:8px;align-items:center;margin-left:auto}
    button{background:transparent;border:1px solid rgba(255,255,255,0.04);padding:8px 12px;border-radius:8px;color:#dbeafe;cursor:pointer}
    button.secondary{background:rgba(255,255,255,0.02)}

    .mobile-controls{display:none;margin-top:12px;gap:8px;justify-content:center}
    .touch-btn{width:60px;height:48px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);display:flex;align-items:center;justify-content:center;font-weight:600;background:rgba(255,255,255,0.01)}

    .side{width:260px}
    .panel{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:14px;border-radius:12px;border:1px solid rgba(255,255,255,0.03)}
    .panel h3{margin:0 0 8px 0;font-size:14px}
    .legend{display:flex;gap:10px;align-items:center}
    .legend .item{display:flex;gap:8px;align-items:center}
    .dot{width:26px;height:18px;border-radius:4px}

    footer{margin-top:12px;color:var(--muted);font-size:13px;text-align:center}

    
    @media (max-width:980px){
      .wrap{flex-direction:column;align-items:center}
      .game-card{width:92%}
      .side{width:92%}
      .mobile-controls{display:flex}
      .controls{display:none}
    }
  </style>
</head>
<body>
  <div class="wrap">
    <div class="game-card">
      <header>
        <div>
          <h1>Caza Rayos ⚡</h1>
          <div class="meta">Atrapa rayos de electricidad para cargar la pila. Flechas / A D / táctil</div>
        </div>
        <div class="controls">
          <button id="restart">Reiniciar</button>
          <button id="pause" class="secondary">Pausar</button>
        </div>
      </header>

      <canvas id="gameCanvas" width="720" height="480"></canvas>

      <div class="hud">
        <div class="battery" title="Carga de la pila">
          <div class="level" id="batteryLevel" style="width:8%"></div>
          <div class="cap"></div>
        </div>
        <div class="stats">
          <div>Charge: <strong id="chargePct">8%</strong></div>
          <div>Rayos atrapados: <strong id="caught">0</strong></div>
          <div>Vidas: <strong id="lives">3</strong></div>
        </div>
        <div class="controls">
          <button id="soundToggle">🔊</button>
        </div>
      </div>

      <div class="mobile-controls" id="mobileControls">
        <div class="touch-btn" id="leftBtn">◀</div>
        <div class="touch-btn" id="downBtn">⤓</div>
        <div class="touch-btn" id="rightBtn">▶</div>
      </div>

      <footer>
        Objetivo: cargar la pila al 100%. Evita los rayos negros —restan vida— y atrapa los azulados para cargarte.
      </footer>
    </div>

    <aside class="side">
      <div class="panel">
        <h3>Controles</h3>
        <div class="legend">
          <div class="item"><div class="dot" style="background:linear-gradient(90deg,#00eaff,#7df9ff)"></div>Rayos cargables</div>
        </div>
        <div class="legend" style="margin-top:8px">
          <div class="item"><div class="dot" style="background:linear-gradient(90deg,#ffd166,#ff6b6b)"></div>Rayos dañinos (restan vida)</div>
        </div>

        <h3 style="margin-top:12px">Consejos</h3>
        <ul style="color:var(--muted);padding-left:18px;margin:8px 0">
          <li>Puedes moverte con las flechas o A / D.</li>
          <li>En móviles usa los botones táctiles.</li>
          <li>Cada rayo azul añade carga; rayo negro quita vida.</li>
        </ul>
      </div>
    </aside>
  </div>

  <script>
    // --- Configuración del juego ---
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const W = canvas.width, H = canvas.height;

    // Player
    const player = { x: W/2 - 40, y: H - 60, w: 80, h: 18, speed: 6 };

    
    let bolts = [];
    let tick = 0;
    let spawnRate = 60;
    let caught = 0;
    let lives = 3;
    let battery = 8; 
    let running = true;
    let soundOn = true;

    
    const batteryLevel = document.getElementById('batteryLevel');
    const chargePct = document.getElementById('chargePct');
    const caughtNode = document.getElementById('caught');
    const livesNode = document.getElementById('lives');

    
    const keys = {};
    window.addEventListener('keydown', e=>{keys[e.key.toLowerCase()]=true; if(['arrowleft','arrowright','arrowup','arrowdown'].includes(e.key.toLowerCase())) e.preventDefault();});
    window.addEventListener('keyup', e=>{keys[e.key.toLowerCase()]=false});

    
    const leftBtn = document.getElementById('leftBtn');
    const rightBtn = document.getElementById('rightBtn');
    const downBtn = document.getElementById('downBtn');
    function bindTouch(el,key){
      el.addEventListener('touchstart', e=>{ e.preventDefault(); keys[key]=true });
      el.addEventListener('touchend', e=>{ e.preventDefault(); keys[key]=false });
      el.addEventListener('mousedown', e=>{ keys[key]=true });
      el.addEventListener('mouseup', e=>{ keys[key]=false });
    }
    bindTouch(leftBtn,'arrowleft'); bindTouch(rightBtn,'arrowright'); bindTouch(downBtn,'arrowdown');

    
    document.getElementById('restart').addEventListener('click', resetGame);
    document.getElementById('pause').addEventListener('click', ()=>{ running = !running; document.getElementById('pause').textContent = running? 'Pausar' : 'Reanudar'; });
    document.getElementById('soundToggle').addEventListener('click', ()=>{ soundOn = !soundOn; document.getElementById('soundToggle').textContent = soundOn? '🔊' : '🔇' });

    
    const AudioCtx = window.AudioContext || window.webkitAudioContext;
    const audioCtx = AudioCtx? new AudioCtx() : null;
    function playBeep(freq=880, time=0.06, type='sine'){
      if(!audioCtx || !soundOn) return;
      const o = audioCtx.createOscillator();
      const g = audioCtx.createGain();
      o.type = type; o.frequency.value = freq;
      g.gain.value = 0.06;
      o.connect(g); g.connect(audioCtx.destination);
      o.start(); g.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + time);
      setTimeout(()=>o.stop(), time*1000 + 20);
    }

    
    function spawnBolt(){
      const x = Math.random() * (W-24) + 12;
      const speed = 1.6 + Math.random()*2.4 + Math.min(2, caught*0.12);
    
      const type = Math.random() < 0.82 ? 0 : 1;
      bolts.push({x, y: -20, w: 16, h: 28, speed, type, swing: Math.random()*0.8});
    }

    function resetGame(){ bolts=[]; tick=0; spawnRate=60; caught=0; lives=3; battery=8; running=true; document.getElementById('pause').textContent='Pausar'; updateHUD(); }

    function updateHUD(){ batteryLevel.style.width = Math.min(100,battery) + '%'; chargePct.textContent = Math.floor(battery) + '%'; caughtNode.textContent = caught; livesNode.textContent = lives; }

    function rectsOverlap(a,b){
      return !(a.x + a.w < b.x || a.x > b.x + b.w || a.y + a.h < b.y || a.y > b.y + b.h);
    }

    function update(){
      if(!running) return;
      tick++;

      
      if(tick % Math.max(12, spawnRate - Math.floor(caught/3)) === 0) spawnBolt();

      
      if(keys['arrowleft'] || keys['a']) player.x -= player.speed;
      if(keys['arrowright'] || keys['d']) player.x += player.speed;
      if(keys['arrowdown'] || keys['s']) player.y += 2; else player.y = Math.min(H - 60, player.y + 0); // small optional mechanic
      player.x = Math.max(8, Math.min(W - player.w - 8, player.x));

      
      for(let i = bolts.length-1; i>=0; i--){
        const b = bolts[i];
        b.y += b.speed;
        b.x += Math.sin((tick + i*10) * 0.02) * b.swing; 
        
        const boltRect = {x: b.x - b.w/2, y: b.y - b.h/2, w: b.w, h: b.h};
        const playerRect = {x: player.x, y: player.y, w: player.w, h: player.h};
        if(rectsOverlap(boltRect, playerRect)){
          if(b.type === 0){ 
            caught++;
            battery = Math.min(100, battery + (6 + Math.random()*7));
            playBeep(1200 + Math.random()*400, 0.06, 'sine');
          } else {
            lives--; playBeep(220, 0.18, 'sawtooth');
            battery = Math.max(0, battery - 8 - Math.random()*6);
          }
          bolts.splice(i,1);
          updateHUD();
          continue;
        }
        
        if(b.y > H + 40) bolts.splice(i,1);
      }

      
      if(tick % 180 === 0) battery = Math.max(0, battery - 0.6);

      
      if(battery >= 100){ running=false; setTimeout(()=>alert('¡Pila cargada al 100%! Ganaste ⚡'), 60); }
      if(lives <= 0){ running=false; setTimeout(()=>alert('Te quedaste sin vidas. Reinicia para intentarlo otra vez.'), 60); }
    }

    function draw(){
      
      ctx.clearRect(0,0,W,H);
    
      ctx.fillStyle = 'rgba(255,255,255,0.02)';
      for(let i=0;i<20;i++){ ctx.fillRect(0, H-80 - i*18, W, 1); }

      
      for(const b of bolts){
        ctx.save();
        ctx.translate(b.x, b.y);
        
        if(b.type===0){
          ctx.beginPath(); ctx.fillStyle = 'rgba(34,255,255,0.06)'; ctx.ellipse(0,6,22,9,0,0,Math.PI*2); ctx.fill();
        } else {
          ctx.beginPath(); ctx.fillStyle = 'rgba(0,0,0,0.3)'; ctx.ellipse(0,6,20,8,0,0,Math.PI*2); ctx.fill();
        }
        
        ctx.beginPath();
        if(b.type===0){
          
          ctx.strokeStyle = 'rgba(125,249,255,0.95)'; ctx.lineWidth = 3; ctx.lineJoin = 'round';
          ctx.moveTo(-4,-12); ctx.lineTo(4,-2); ctx.lineTo(-2,2); ctx.lineTo(8,12);
          ctx.stroke();
          ctx.strokeStyle = 'rgba(0,200,255,0.25)'; ctx.lineWidth = 8; ctx.stroke();
        } else {
          
          ctx.strokeStyle = 'rgba(255,165,120,0.06)'; ctx.lineWidth = 10; ctx.stroke();
          ctx.strokeStyle = 'rgba(40,40,40,0.95)'; ctx.lineWidth = 3; ctx.moveTo(-6,-14); ctx.lineTo(2,-6); ctx.lineTo(-3,2); ctx.lineTo(6,12); ctx.stroke();
        }
        ctx.restore();
      }

     
      ctx.fillStyle = '#0b1220';
      ctx.fillRect(player.x, player.y, player.w, player.h);
      
      const g = ctx.createLinearGradient(player.x, player.y, player.x+player.w, player.y);
      g.addColorStop(0,'rgba(0,234,255,0.12)'); g.addColorStop(1,'rgba(125,249,255,0.12)');
      ctx.fillStyle = g; ctx.fillRect(player.x, player.y-6, player.w, 6);
      
      ctx.strokeStyle = 'rgba(255,255,255,0.04)'; ctx.strokeRect(player.x, player.y, player.w, player.h);

    
      ctx.fillStyle='rgba(255,255,255,0.02)'; ctx.fillRect(8,8,120,34); 

      
      ctx.fillStyle='#bfefff'; ctx.font = '14px system-ui'; ctx.fillText('Pila: ' + Math.floor(battery) + '%', 14, 32);

    }

    function loop(){ update(); draw(); if(running) requestAnimationFrame(loop); else draw(); }


    updateHUD(); loop();

   
    ['pointerdown','keydown','touchstart','mousedown'].forEach(evt=>window.addEventListener(evt, ()=>{ if(audioCtx && audioCtx.state==='suspended') audioCtx.resume(); }, {once:true}));
  </script>
</body>
</html>
