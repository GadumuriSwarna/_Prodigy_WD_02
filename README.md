<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Tic‑Tac‑Toe — PvP & Vs AI</title>
  <style>
    :root{
      --bg:#0f1320;
      --panel:#151b2e;
      --line:#2a3558;
      --text:#e8eeff;
      --muted:#a8b3d9;
      --accent1:#3a7bd5; /* blue */
      --accent2:#00d2ff; /* cyan */
      --win:#22c55e;
      --lose:#ef4444;
      --draw:#f59e0b;
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0; display:grid; place-items:center; min-height:100%;
      font:16px/1.4 system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial;
      color:var(--text); background:radial-gradient(900px 500px at 10% -10%, rgba(58,123,213,.15), transparent 60%),
                       radial-gradient(900px 500px at 110% 10%, rgba(0,210,255,.12), transparent 40%),
                       var(--bg);
    }
    .wrap{ width:min(900px, 92%); }

    .header{ display:flex; align-items:center; gap:12px; margin-bottom:16px; }
    .title{ font-weight:800; letter-spacing:.3px; }
    .subtitle{ color:var(--muted); font-size:.95rem }

    /* Monoline solution icon */
    .icon{ width:42px; height:42px; flex:0 0 auto }
    svg{ display:block }
    .panel{
      background:linear-gradient(180deg, rgba(21,27,46,.85), rgba(15,19,32,.85));
      border:1px solid rgba(255,255,255,.06);
      box-shadow:0 20px 50px rgba(0,0,0,.35);
      backdrop-filter: blur(8px) saturate(120%);
      border-radius:18px; padding:18px;
    }

    .controls{ display:flex; flex-wrap:wrap; gap:12px; align-items:center; justify-content:space-between; margin-bottom:16px; }
    .left, .right{ display:flex; gap:12px; align-items:center }

    select, button{
      color:var(--text); background:#0f1526; border:1px solid var(--line);
      padding:10px 12px; border-radius:12px; font-weight:600; cursor:pointer;
      transition: transform .12s ease, box-shadow .2s ease, border-color .2s ease;
    }
    select:focus, button:focus{ outline:none; border-color:#6fa8ff; box-shadow:0 0 0 3px rgba(111,168,255,.2) }
    button.primary{ background: linear-gradient(135deg, var(--accent1), var(--accent2)); border-color:transparent }
    button.primary:hover{ transform: translateY(-1px); box-shadow:0 10px 22px rgba(0,140,255,.35) }

    .status{ padding:10px 12px; border-radius:12px; background:rgba(255,255,255,.04); border:1px solid rgba(255,255,255,.06) }

    /* Board */
    .board{ --size: 420px; width:var(--size); height:var(--size); margin:18px auto; display:grid; grid-template-columns:repeat(3,1fr); gap:10px; }
    .cell{
      display:grid; place-items:center; font-weight:900; font-size:clamp(48px, 8vw, 84px);
      height:100%; border-radius:16px; background:#0e1424; border:1px solid var(--line);
      color:var(--text); cursor:pointer; user-select:none; transition: transform .08s ease, background .2s ease, border-color .2s ease;
    }
    .cell:hover{ transform: translateY(-1px); background:#111a33; border-color:#3a4a7a }
    .cell.X{ color:#a5d8ff; text-shadow:0 6px 18px rgba(0,170,255,.25) }
    .cell.O{ color:#ffd5a5; text-shadow:0 6px 18px rgba(255,170,0,.2) }
    .cell.win{ outline:3px solid var(--win); box-shadow:0 0 0 3px rgba(34,197,94,.2) inset }

    .footer{ display:flex; justify-content:space-between; align-items:center; gap:12px; flex-wrap:wrap; }
    .legend{ color:var(--muted); font-size:.9rem }

    @media (max-width:520px){ .board{ --size: 320px; } }
  </style>
</head>
<body>
  <div class="wrap">
    <div class="header">
      <div class="icon">
        <!-- Monoline tic-tac-toe + bulb hybrid -->
        <svg viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
          <path d="M20 8h24M20 32h24M20 56h24M8 20v24M32 20v24M56 20v24" stroke="#9fb3ff" stroke-width="2" stroke-linecap="round"/>
          <path d="M14 14l8 8M22 14l-8 8M42 42l8 8M50 42l-8 8" stroke="#00d2ff" stroke-width="2" stroke-linecap="round"/>
          <path d="M32 6c7 0 13 6 13 13 0 5-3 9-6 11-2 2-3 5-3 8h-8c0-3-1-6-3-8-3-2-6-6-6-11 0-7 6-13 13-13Z" stroke="#6ee7ff" stroke-width="2" stroke-linecap="round"/>
        </svg>
      </div>
      <div>
        <div class="title">Tic‑Tac‑Toe</div>
        <div class="subtitle">Play local 2‑player or take on an <strong>unbeatable</strong> AI.</div>
      </div>
    </div>

    <div class="panel">
      <div class="controls">
        <div class="left">
          <label for="mode" class="subtitle">Mode</label>
          <select id="mode" aria-label="Game mode">
            <option value="pvp">Two Players</option>
            <option value="ai" selected>Play vs AI</option>
          </select>
          <div class="status" id="status">X to move</div>
        </div>
        <div class="right">
          <button class="primary" id="newGame">New Game</button>
        </div>
      </div>

      <div class="board" id="board" role="grid" aria-label="Tic Tac Toe board"></div>

      <div class="footer">
        <div class="legend">X uses blue, O uses amber. Win = three in a row.</div>
        <div class="legend" id="score">Scores — X: <span id="sx">0</span> · O: <span id="so">0</span> · Draws: <span id="sd">0</span></div>
      </div>
    </div>
  </div>

  <script>
    // --- State ---
    const boardEl = document.getElementById('board');
    const statusEl = document.getElementById('status');
    const modeEl = document.getElementById('mode');
    const newGameBtn = document.getElementById('newGame');
    const scoreX = document.getElementById('sx');
    const scoreO = document.getElementById('so');
    const scoreD = document.getElementById('sd');

    let board = Array(9).fill(null); // 'X' | 'O' | null
    let current = 'X';
    let locked = false;
    let scores = { X:0, O:0, D:0 };

    // Winning combos
    const LINES = [
      [0,1,2],[3,4,5],[6,7,8],
      [0,3,6],[1,4,7],[2,5,8],
      [0,4,8],[2,4,6]
    ];

    // Build board UI
    function renderBoard(){
      boardEl.innerHTML = '';
      for(let i=0;i<9;i++){
        const btn = document.createElement('button');
        btn.className = 'cell';
        btn.setAttribute('role','gridcell');
        btn.setAttribute('aria-label', `cell ${i+1}`);
        btn.dataset.idx = i;
        btn.addEventListener('click', onCellClick);
        boardEl.appendChild(btn);
      }
      updateCells();
    }

    function updateCells(highlight=[]) {
      const cells = boardEl.querySelectorAll('.cell');
      cells.forEach((c, i) => {
        c.textContent = board[i] ? board[i] : '';
        c.classList.toggle('X', board[i] === 'X');
        c.classList.toggle('O', board[i] === 'O');
        c.classList.toggle('win', highlight.includes(i));
      });
    }

    function onCellClick(e){
      if(locked) return;
      const i = +e.currentTarget.dataset.idx;
      if(board[i]) return;
      move(i, current);
      const res = getResult(board);
      if(res.done){ endGame(res); return; }
      // AI turn if enabled
      if(modeEl.value === 'ai'){
        locked = true;
        setTimeout(()=>{
          const aiIndex = bestMove(board, 'O');
          move(aiIndex, 'O');
          const res2 = getResult(board);
          locked = false;
          if(res2.done){ endGame(res2); } else { current = 'X'; updateStatus(); }
        }, 120);
      } else {
        current = current === 'X' ? 'O' : 'X';
        updateStatus();
      }
    }

    function move(i, player){
      board[i] = player;
      updateCells();
    }

    function updateStatus(msg){
      if(msg){ statusEl.textContent = msg; return; }
      statusEl.textContent = `${current} to move`;
    }

    function getResult(b){
      for(const [a,b1,c] of LINES){
        if(b[a] && b[a]===b[b1] && b[a]===b[c]) return {done:true, win:b[a], line:[a,b1,c]};
      }
      if(b.every(Boolean)) return {done:true, win:null};
      return {done:false};
    }

    function endGame(res){
      if(res.win){
        updateCells(res.line);
        updateStatus(`${res.win} wins!`);
        scores[res.win]++; updateScore();
      } else {
        updateStatus(`It's a draw.`);
        scores.D++; updateScore();
      }
      locked = true;
    }

    function updateScore(){
      scoreX.textContent = scores.X;
      scoreO.textContent = scores.O;
      scoreD.textContent = scores.D;
    }

    function resetGame(){
      board = Array(9).fill(null); current = 'X'; locked = false; updateCells(); updateStatus();
    }

    // --- AI (Minimax, optimal) ---
    function bestMove(b, ai){
      const human = ai === 'X' ? 'O' : 'X';
      let best = { score: -Infinity, index: null };
      for(const i of emptyIndices(b)){
        const newB = b.slice(); newB[i] = ai;
        const score = minimax(newB, false, ai, human);
        if(score > best.score){ best = { score, index:i }; }
      }
      return best.index;
    }

    function minimax(b, isMax, ai, human){
      const res = getResult(b);
      if(res.done){
        if(res.win === ai) return 10;
        if(res.win === human) return -10;
        return 0;
      }
      if(isMax){
        let best = -Infinity;
        for(const i of emptyIndices(b)){
          const nb = b.slice(); nb[i] = ai;
          best = Math.max(best, minimax(nb, false, ai, human));
        }
        return best;
      } else {
        let best = Infinity;
        for(const i of emptyIndices(b)){
          const nb = b.slice(); nb[i] = human;
          best = Math.min(best, minimax(nb, true, ai, human));
        }
        return best;
      }
    }
    const emptyIndices = (b) => b.map((v,i)=>v?null:i).filter(v=>v!==null);

    // Events
    newGameBtn.addEventListener('click', resetGame);
    modeEl.addEventListener('change', ()=>{ resetGame(); });

    // Init
    renderBoard();
    updateStatus();
  </script>
</body>
</html>
