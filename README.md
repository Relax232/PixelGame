# PixelGame
İstediğin şekilde düzenleyebildiğin Oyunumuzu dilediğin Gibi oyna / Play our game however you want, which you can edit as you wish.
<!doctype html>
<html lang="tr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Snake - Tek Dosya Oyun</title>
  <style>
    :root{ --bg:#0f1724; --card:#0b1220; --accent:#10b981; --muted:#94a3b8 }
    html,body{height:100%;margin:0;font-family:Inter,Segoe UI,Roboto,Helvetica,Arial,sans-serif;background:linear-gradient(180deg,#071428 0%, #071b2a 100%);color:#e6eef8}
    .wrap{min-height:100vh;display:flex;align-items:center;justify-content:center;padding:24px}
    .card{background:rgba(255,255,255,0.03);padding:18px;border-radius:12px;box-shadow:0 6px 30px rgba(2,6,23,0.6);display:flex;gap:18px;align-items:center}
    canvas{background:linear-gradient(180deg,#052034,#063046);border-radius:8px;display:block}
    .ui{width:220px}
    h1{font-size:18px;margin:0 0 8px}
    p.muted{color:var(--muted);margin:6px 0 12px;font-size:13px}
    .row{display:flex;gap:8px}
    button{cursor:pointer;padding:8px 12px;border-radius:8px;border:0;background:var(--accent);color:#042018;font-weight:600}
    button.ghost{background:transparent;border:1px solid rgba(255,255,255,0.06);color:#d8f3e6}
    .score{font-weight:700;font-size:20px;margin-bottom:8px}
    .small{font-size:13px;color:var(--muted)}
    footer.small{margin-top:10px;color:var(--muted);font-size:12px}
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <canvas id="game" width="480" height="480"></canvas>
      <div class="ui">
        <h1>Snake — Tek Dosya</h1>
        <div class="score">Skor: <span id="score">0</span></div>
        <p class="muted">Yön tuşları veya WASD ile oynayın. Boşluk = duraklat/başlat.</p>
        <div class="row">
          <button id="btnRestart">Yeniden Başlat</button>
          <button id="btnPause" class="ghost">Duraklat</button>
        </div>
        <p class="small">Ayarlar</p>
        <div style="display:flex;gap:8px;margin-top:8px;align-items:center">
          <label class="small">Hız</label>
          <input id="speed" type="range" min="4" max="16" value="8" />
        </div>
        <footer class="small">Tek dosya HTML + JS — GitHub Pages ile çalışır.</footer>
      </div>
    </div>
  </div>

<script>
(() => {
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');
  const scoreEl = document.getElementById('score');
  const btnRestart = document.getElementById('btnRestart');
  const btnPause = document.getElementById('btnPause');
  const speedInput = document.getElementById('speed');

  const tileCount = 24; // grid
  let tileSize = canvas.width / tileCount;

  let snake = [{x:12,y:12}];
  let dir = {x:1,y:0};
  let food = spawnFood();
  let score = 0;
  let running = true;
  let stepInterval = 1000/parseInt(speedInput.value);
  let lastTime = 0;

  function spawnFood(){
    while(true){
      const fx = Math.floor(Math.random()*tileCount);
      const fy = Math.floor(Math.random()*tileCount);
      if(!snake.some(s => s.x===fx && s.y===fy)) return {x:fx,y:fy};
    }
  }

  function draw(){
    // background
    ctx.clearRect(0,0,canvas.width,canvas.height);
    // grid subtle
    for(let i=0;i<tileCount;i++){
      for(let j=0;j<tileCount;j++){
        // optional: subtle checker
      }
    }

    // draw food
    ctx.fillStyle = '#ff4d4f';
    roundRect(ctx, food.x*tileSize+4, food.y*tileSize+4, tileSize-8, tileSize-8, 6, true, false);

    // draw snake
    for(let i=0;i<snake.length;i++){
      const s = snake[i];
      ctx.fillStyle = i===0 ? '#2dd4bf' : '#0ea5a3';
      roundRect(ctx, s.x*tileSize+2, s.y*tileSize+2, tileSize-4, tileSize-4, 6, true, false);
    }
  }

  function roundRect(ctx, x, y, w, h, r, fill, stroke) {
    if (typeof r === 'undefined') r = 5;
    ctx.beginPath();
    ctx.moveTo(x + r, y);
    ctx.arcTo(x + w, y, x + w, y + h, r);
    ctx.arcTo(x + w, y + h, x, y + h, r);
    ctx.arcTo(x, y + h, x, y, r);
    ctx.arcTo(x, y, x + w, y, r);
    ctx.closePath();
    if (fill) ctx.fill();
    if (stroke) ctx.stroke();
  }

  function update(){
    // move snake
    const head = {x: (snake[0].x + dir.x + tileCount) % tileCount, y: (snake[0].y + dir.y + tileCount) % tileCount};

    // collision with self?
    if(snake.some((s,idx) => idx!==0 && s.x===head.x && s.y===head.y)){
      // game over
      running = false;
      btnPause.textContent = 'Oyun Bitti';
      return;
    }

    snake.unshift(head);
    if(head.x===food.x && head.y===food.y){
      score += 1;
      scoreEl.textContent = score;
      food = spawnFood();
    } else {
      snake.pop();
    }
  }

  function gameLoop(ts){
    if(!lastTime) lastTime = ts;
    const dt = ts - lastTime;
    if(running){
      if(dt > stepInterval){
        update();
        draw();
        lastTime = ts;
      }
    }
    requestAnimationFrame(gameLoop);
  }

  // controls
  window.addEventListener('keydown', e => {
    const k = e.key;
    if(k==='ArrowUp' || k==='w' || k==='W') setDir(0,-1);
    if(k==='ArrowDown' || k==='s' || k==='S') setDir(0,1);
    if(k==='ArrowLeft' || k==='a' || k==='A') setDir(-1,0);
    if(k==='ArrowRight' || k==='d' || k==='D') setDir(1,0);
    if(k===' '){ togglePause(); }
  });

  function setDir(x,y){
    // prevent reversing
    if(snake.length>1 && snake[0].x + x === snake[1].x && snake[0].y + y === snake[1].y) return;
    dir = {x,y};
  }

  function togglePause(){
    running = !running;
    btnPause.textContent = running ? 'Duraklat' : 'Devam Et';
  }

  btnPause.addEventListener('click', togglePause);
  btnRestart.addEventListener('click', () => {
    snake = [{x:12,y:12}]; dir = {x:1,y:0}; food = spawnFood(); score = 0; scoreEl.textContent = score; running = true; btnPause.textContent='Duraklat';
  });

  speedInput.addEventListener('input', ()=>{
    const sp = parseInt(speedInput.value);
    stepInterval = 1000/sp;
  });

  // responsive canvas scaling
  function resize(){
    const min = Math.min(window.innerWidth-320, 540);
    const size = Math.max(320, min);
    canvas.style.width = size+'px';
    canvas.style.height = size+'px';
    tileSize = canvas.width / tileCount;
    draw();
  }
  window.addEventListener('resize', resize);
  resize();
  requestAnimationFrame(gameLoop);
})();
</script>
</body>
</html>
