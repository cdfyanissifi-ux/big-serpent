<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="theme-color" content="#07000e">
  <title>Snake Jungle</title>

  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      background: #0d1f0d;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      font-family: sans-serif;
    }
    #score-bar {
      display: flex;
      gap: 24px;
      margin-bottom: 12px;
      font-size: 14px;
      color: #aaa;
    }
    #score-bar span { font-weight: 600; color: #fff; }
    #game-canvas {
      border-radius: 12px;
      display: block;
      touch-action: none;
    }
    #msg {
      margin-top: 14px;
      min-height: 28px;
      font-size: 15px;
      font-weight: 600;
      color: #e53935;
      text-align: center;
    }
    #restart-btn {
      margin-top: 10px;
      padding: 8px 28px;
      border-radius: 8px;
      border: 1.5px solid #2e7d32;
      background: #2e7d32;
      color: #fff;
      font-size: 14px;
      font-weight: 500;
      cursor: pointer;
      display: none;
    }
    #restart-btn:hover { background: #1b5e20; }
    #controls {
      margin-top: 10px;
      font-size: 12px;
      color: #666;
      text-align: center;
    }
  </style>
</head>
<body>

  <div id="score-bar">Score : <span id="score">0</span> &nbsp;|&nbsp; Record : <span id="best">0</span></div>
  <canvas id="game-canvas" width="400" height="400"></canvas>
  <div id="msg"></div>
  <button id="restart-btn" onclick="startGame()">Rejouer</button>
  <div id="controls">Swipe ou flèches / ZQSD — Touche l'écran pour démarrer</div>

  <script>
    const C = document.getElementById('game-canvas');
    const ctx = C.getContext('2d');
    const COLS = 20, ROWS = 20, CELL = 20;
    const W = COLS * CELL, H = ROWS * CELL;

    let snake, dir, nextDir, apple, score, best = 0, running = false, loop, started = false;

    const DEATH_MSGS = [
      "T'es nul, tu peux mieux faire ! 🐍",
      "Même ma grand-mère joue mieux...",
      "Vraiment ? C'était quoi ce mouvement ?",
      "Tu t'es mangé toi-même... bravo.",
      "Le serpent a honte de toi.",
      "Essaie encore, ça peut pas être pire.",
      "Tu aurais pu tourner, non ?"
    ];

    function rnd(n) { return Math.floor(Math.random() * n); }

    function placeApple() {
      let pos;
      do { pos = { x: rnd(COLS), y: rnd(ROWS) }; }
      while (snake.some(s => s.x === pos.x && s.y === pos.y));
      apple = pos;
    }

    function startGame() {
      snake = [{ x: 10, y: 10 }, { x: 9, y: 10 }, { x: 8, y: 10 }];
      dir = { x: 1, y: 0 };
      nextDir = { x: 1, y: 0 };
      score = 0;
      document.getElementById('score').textContent = 0;
      document.getElementById('msg').textContent = '';
      document.getElementById('restart-btn').style.display = 'none';
      placeApple();
      running = true;
      started = true;
      clearInterval(loop);
      loop = setInterval(tick, 130);
    }

    function tick() {
      dir = nextDir;
      const head = { x: (snake[0].x + dir.x + COLS) % COLS, y: (snake[0].y + dir.y + ROWS) % ROWS };
      if (snake.some(s => s.x === head.x && s.y === head.y)) { gameOver(); return; }
      snake.unshift(head);
      if (head.x === apple.x && head.y === apple.y) {
        score++;
        if (score > best) best = score;
        document.getElementById('score').textContent = score;
        document.getElementById('best').textContent = best;
        placeApple();
      } else { snake.pop(); }
      draw();
    }

    function gameOver() {
      running = false;
      clearInterval(loop);
      document.getElementById('msg').textContent = DEATH_MSGS[rnd(DEATH_MSGS.length)];
      document.getElementById('restart-btn').style.display = 'inline-block';
      drawDead();
    }

    function drawBg() {
      const grad = ctx.createLinearGradient(0, 0, 0, H);
      grad.addColorStop(0, '#1a3a1a');
      grad.addColorStop(1, '#0d2a0d');
      ctx.fillStyle = grad;
      ctx.fillRect(0, 0, W, H);
      ctx.fillStyle = 'rgba(34,85,34,0.18)';
      for (let i = 0; i < 18; i++) {
        const bx = (i * 71 + 30) % W, by = (i * 53 + 15) % H;
        ctx.beginPath();
        ctx.ellipse(bx, by, 22, 9, (i * 0.4), 0, Math.PI * 2);
        ctx.fill();
      }
      for (let i = 0; i < 8; i++) {
        const tx = (i * 97 + 20) % W, ty = (i * 61 + 10) % H;
        ctx.fillStyle = 'rgba(20,60,20,0.35)';
        ctx.beginPath();
        ctx.moveTo(tx, ty + 28);
        for (let j = 0; j < 6; j++) {
          const a = (j / 6) * Math.PI * 2;
          ctx.ellipse(tx + Math.cos(a) * 14, ty + Math.sin(a) * 8, 10, 5, a, 0, Math.PI * 2);
        }
        ctx.fill();
      }
    }

    function drawApple() {
      const ax = apple.x * CELL + CELL / 2, ay = apple.y * CELL + CELL / 2;
      ctx.save();
      ctx.shadowColor = 'rgba(220,50,50,0.5)';
      ctx.shadowBlur = 8;
      ctx.fillStyle = '#e53935';
      ctx.beginPath();
      ctx.arc(ax, ay, 7, 0, Math.PI * 2);
      ctx.fill();
      ctx.shadowBlur = 0;
      ctx.fillStyle = '#ef9a9a';
      ctx.beginPath();
      ctx.arc(ax - 2, ay - 2, 2.5, 0, Math.PI * 2);
      ctx.fill();
      ctx.strokeStyle = '#33691e';
      ctx.lineWidth = 1.5;
      ctx.beginPath();
      ctx.moveTo(ax + 2, ay - 6);
      ctx.quadraticCurveTo(ax + 8, ay - 12, ax + 6, ay - 9);
      ctx.stroke();
      ctx.restore();
    }

    function drawSnake(dead) {
      snake.forEach((seg, i) => {
        const x = seg.x * CELL, y = seg.y * CELL;
        const t = i / snake.length;
        if (dead) {
          ctx.fillStyle = `hsl(0, 60%, ${35 + t * 20}%)`;
        } else {
          ctx.fillStyle = i === 0 ? '#76ff03' : `hsl(${100 + t * 40}, 70%, ${40 + t * 15}%)`;
        }
        ctx.beginPath();
        ctx.roundRect(x + 2, y + 2, CELL - 4, CELL - 4, 5);
        ctx.fill();
        if (i === 0) {
          ctx.fillStyle = dead ? '#b71c1c' : '#212121';
          const ex = dir.x, ey = dir.y;
          const ox = ey !== 0 ? 3 : 0, oy = ex !== 0 ? 3 : 0;
          ctx.beginPath();
          ctx.arc(x + CELL/2 - ex*2 + ox, y + CELL/2 - ey*2 + oy, 2, 0, Math.PI*2);
          ctx.arc(x + CELL/2 - ex*2 - ox, y + CELL/2 - ey*2 - oy, 2, 0, Math.PI*2);
          ctx.fill();
        }
      });
    }

    function draw() { drawBg(); drawApple(); drawSnake(false); }

    function drawDead() {
      drawBg(); drawApple(); drawSnake(true);
      ctx.fillStyle = 'rgba(0,0,0,0.45)';
      ctx.fillRect(0, 0, W, H);
      ctx.fillStyle = '#ef9a9a';
      ctx.font = 'bold 22px sans-serif';
      ctx.textAlign = 'center';
      ctx.fillText('GAME OVER', W/2, H/2);
    }

    // Clavier
    document.addEventListener('keydown', e => {
      const keys = {
        ArrowUp:[0,-1], ArrowDown:[0,1], ArrowLeft:[-1,0], ArrowRight:[1,0],
        KeyZ:[0,-1], KeyS:[0,1], KeyQ:[-1,0], KeyD:[1,0]
      };
      const k = keys[e.code];
      if (k) {
        if (!started) { startGame(); return; }
        if (running && !(k[0] === -dir.x && k[1] === -dir.y)) nextDir = { x: k[0], y: k[1] };
        e.preventDefault();
      }
    });

    // Swipe tactile
    let touchStartX = 0, touchStartY = 0;

    C.addEventListener('touchstart', e => {
      e.preventDefault();
      if (!started) { startGame(); return; }
      touchStartX = e.touches[0].clientX;
      touchStartY = e.touches[0].clientY;
    }, { passive: false });

    C.addEventListener('touchend', e => {
      e.preventDefault();
      if (!running) { if (started) startGame(); return; }
      const dx = e.changedTouches[0].clientX - touchStartX;
      const dy = e.changedTouches[0].clientY - touchStartY;
      if (Math.abs(dx) < 10 && Math.abs(dy) < 10) return;
      let nd;
      if (Math.abs(dx) > Math.abs(dy)) {
        nd = dx > 0 ? { x: 1, y: 0 } : { x: -1, y: 0 };
      } else {
        nd = dy > 0 ? { x: 0, y: 1 } : { x: 0, y: -1 };
      }
      if (!(nd.x === -dir.x && nd.y === -dir.y)) nextDir = nd;
    }, { passive: false });

    // Écran de démarrage
    drawBg();
    ctx.fillStyle = '#76ff03';
    ctx.font = 'bold 15px sans-serif';
    ctx.textAlign = 'center';
    ctx.fillText('Swipe ou touche pour démarrer', W/2, H/2 - 10);
    ctx.fillStyle = 'rgba(150,255,100,0.6)';
    ctx.font = '13px sans-serif';
    ctx.fillText('🐍 Snake Jungle 🌿', W/2, H/2 + 14);
  </script>

</body>
</html>
